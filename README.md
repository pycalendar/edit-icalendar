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


```python
from ical_edit import parse
cal = parse("/path/to/file.ics")

cal.events # same as cal.select(name=["VEVENT"]) and a list of events
selection = cal.select(name=["VEVENT"], uid="...")  # same selection interface as recurring-ical-events

```

A selection is a sub-set of the calendar that collects all
components that are relevant to a time query.


```python
class Selection:

    def select(name=None, uid=None):
    def with_uid(uid)
    def with_name(name)

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
