---
title: schedule()
description: Creates a new Schedule to run a function on a defined frequency.
---

Creates a new Schedule to run a function on a defined frequency.

```python
from nitric.resources import schedule
from nitric.faas import EventContext

@schedule('run-a-report', '1 days')
async def process_transactions(ctx: EventContext):
  # do some processing
  pass
```

## Parameters

---

**description** required `string`

The unique name of this Schedule within the app. Subsequent calls to `schedule` with the same name will return the same object.

---

**every** required `string`

The rate description for the schedule. Supported frequencies include `seconds`, `minutes`, `hours` and `days`.

Using `every` as a keyword argument can help with readability of schedules, e.g.

```python
@schedule("backup", every="2 days")
```

#### Examples:

| description      | example schedule                    |
| ---------------- | ----------------------------------- |
| Every day        | @schedule("work", every="day")      |
| Every 14 hours   | @schedule("work", every="14 hours") |
| Every 30 minutes | @schedule("work", "30 minutes")     |

> Singular rates will be automatically converted. e.g. "day" will be interpreted as "1 days".

---

## Notes

- Schedules do not require access permissions to be specified.

- Currently, local execution and testing of schedules is not supported.

- You can directly test the functions that respond to scheduled triggers by sending HTTP requests to those functions with the same payload as defined in your schedule.

> Coming Soon

- Local and manual testing of schedules is on our backlog to be completed soon.

## Examples

### Create a Schedule

```python
from nitric.resources import schedule
from nitric.faas import EventContext

@schedule('process-transactions', '5 minutes')
async def process_transactions(ctx: EventContext):
  # do some processing
  pass

@schedule('send-reminder', '3 hours')
async def process_transactions(ctx: EventContext):
  # do some processing
  pass

@schedule('send-reports', '1 days')
async def process_transactions(ctx: EventContext):
  # do some processing
  pass
```