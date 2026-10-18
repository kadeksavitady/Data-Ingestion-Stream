# Technical Analysis

## Architecture Decision

- Partitions: 3; consumer group: `dpw3-case`; consumers: 2.
- Event key: `order_id`, preserving order for each order.
- Offset: `earliest`; manual synchronous commit after processing.
- Delivery: at-least-once; uncommitted events may replay.

Three partitions allow parallel processing across orders. Two consumers share
the group and can be reassigned after a failure. Committing after processing
supports recovery but can produce duplicates.

## Failure Analysis

Action: C1 used `FAIL_AFTER=3`; C2 shared its group; producer sent eight events.
Observation: C1 failed before committing `PAY-008`; C2 replayed it after rebalance.
Diagnosis: the event offset was uncommitted.
Fix: restart with `GROUP_ID=dpw3-case`; producer and consumer source stayed unchanged.
Verification: recovery processed more events; final offsets were 9/6/9, lag 0.
