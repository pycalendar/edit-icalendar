# edit-icalendar

Edit icalendar files, recurring events, todos, alarms and more.

----

icalendar files contain events, todos, journal entries, alarms and other components.
These can be scheduled to re-appear at different times.

## Complexity

Depending on how these are scheduled, editing the calendar changes. These are some examples:

- Deleting an event
  - A single event - the event component can be removed
  - A re-occurring event - it can be excluded
  - An edited event - the edits need to be found and removed
- Editing an event
  - A single event can be replaced or a new version added
  - A re-occurring event - a new scheduled version can be added
- Snoozing an alarm
  - Should be compatible to existing standards and implementations
  - Should be clearly scoped: Several events can have the same alarm.

## API

We assume that the component we are dealing with is a `VCALENDAR`.
The initial data comes in the form of an `icalendar.Calendar` (or data to parse for that).

Process:

1. **Parse**
2. **Select** what is important to you (one event, all events, all todos, only alarms)
3. **Query** the time frame that you need.
4. **Edit** the calendar
5. **Save** the calendar

### 1. Parsing

Parse from a file:

```python
from ical_edit import parse
cal = parse("/path/to/file.ics")
```

Use an existing `icalendar.Calendar`:

```
import ical_edit
ical_edit.Calendar(cal : icalendar.Calendar)
```


### 2. Selection

```
cal.events # same as cal.select(name=["VEVENT"]) and a list of events
selection = cal.select(name=["VEVENT"], uid="...")  # same selection interface as recurring-ical-events
```

This would also be nice:

```
cal.events + cal.todos
```

A selection is a sub-set of the calendar that collects all
components that are relevant to a time query.


```python

class SelectorInterface:
    """Interface to execute a selection.

    This can be replaced e.g. by caldav to load web resources.
    """
    def __init__(self, calendar):

    def get_components_with(name, uid) -> list[Components]

class Selection:

    def __init__(SelectorInterface)

    def select(name=None, uid=None):
    def with_uid(uid)
    def with_name(name)

    def get() -> list[icalendar.Component]
```

### 3. Query

While a selection limits the result, a query expands it based on time.

The query for components occurring within a time span is the same for all types of components.
It is based on a `Selection` bound to a calendar.

```python
cal.events.after(dt)
cal.events.before(dt)
cal.events.between(dt, dt)
cal.events.between(dt, duration)
cal.events.between(duration, dt)
cal.events.at(dt)

cal.journals ...
cal.todos ...
```

#### Iteration of results

```python
result : AfterResult = cal.journals.after(...)
result : BeforeResult = cal.journals.after(...)
result : RangeResult = cal.journals.after(...)
```

Results can be combined if they are of the same type.
Iteration then happens in some kind of order.
Before and After are potentially endless.

```python
for occurrence in result:  # default order
    ...
```

The results have the selector and query saved in them, so one can check
why they occur.

```python
    occurrence.selector
    occurrence.query -> AfterQuery, BeforeQuery, SpanQuery
```

#### Alarms 

Alarms are a bit different.
They are inside a component that might not be scheduled within the
time queried.

```python
if occurrence.selector.is_alarm(): ...
    occurrence.alarm # not None
```

The sorting order of the result depends on the alarm.

### 4. Editing

Editing has several modes.
It takes place on the parsed calendar.
What works for events also works for journal entries and todos.

```python
event = cal.events.after(date(2026, 1, 3)).first

# read-only accessors

event.start # 2026-01-03

edit = event.edit = "Edited"
```
Saving it has different possibilities.

This is the series of events (with the same `UID`) of which one event gets edited.

| Time | Old Summary | `save_this()` |
| ---- | ---- | ------- |
| 2026-01-01 | Original | Original |
| 2026-01-02 | Original | Original |
| 2026-01-03 | Original | Edited   |
| 2026-01-04 | Original | Original |
| 2026-01-05 | Original | Original |

- `save_this()` edits only this event.

#### Alarms

Alarms are a bit different again. They can be 

```python
occurrence.alarm.delete(...)
occurrence.alarm.snooze(...)
occurrence.alarm.unsnooze(...)
occurrence.alarm.acknowledge(...)
occurrence.alarm.unacknowledge(...)
```

## Implementation

Dependencies of `edit-icalendar`:

- `icalendar` for parsing
- `recurring-ical-events` for scheduling computations
- `x-wr-timezone` for compatibility

Projected usage:

- `caldav` allows to edit calendars

Prior work:

- `recurring-ical-events` documents [how to edit an event](https://recurring-ical-events.readthedocs.io/en/latest/user-guide/examples.html#edit-one-event-of-an-existing-series).

Ecosystem:

- As of 2026-10-12 I have no knowledge of a Python library that allows editing calendars based on their occurrences in an easy way.
