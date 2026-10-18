# Experiment Report — DPW Modul 3

## 1. Partition Analysis

Evidence: Topic description and consumer output from the `dpw3-partition`
experiment.

Observation: `dpw3-events` had three partitions. Keyed records were consumed
from partitions 0, 1, and 2. Offsets started independently in each partition.

Conclusion: A topic can distribute records over partitions, and an offset is a
position within one partition, not a global topic position.

## 2. Consumer Group Analysis

Evidence: Consumer-group output and logs for `dpw3-group` and `dpw3-failure`.

Observation: With two members, C1 handled partitions 0 and 1 while C2 handled
partition 2. After C1 failed and the session timeout elapsed, C2 was assigned
all three partitions.

Conclusion: Kafka shares partitions among members of a group and reassigns them
after membership changes.

## 3. Offset Analysis

Evidence: `dpw3-offset` group description after each run.

Observation: The first run processed five events; the group query showed
`CURRENT-OFFSET=5`. The restart continued at offset 5 and advanced the
`CURRENT-OFFSET` to 10; lag changed from 15 to 10.

Conclusion: A consumer group resumes from its committed offsets. The consumer
command used `--manual-commit` because the supplied source disables automatic
commits and otherwise does not commit offsets.

## 4. Failure Analysis

Evidence: Logs for `dpw3-failure`, followed by consumer-group descriptions.

Observation: C1 failed after three records. C2 continued, was eventually
assigned all partitions, and a restarted C1 joined the same group.

Diagnosis: Failure changed group membership; Kafka rebalanced the partition
assignments. A record processed before its offset is committed can be replayed.

Fix: Restart the consumer with the same group ID; no code change was needed for
this experiment.

Verification: The recovered group had active members and continued processing
records.

## 5. At-least-once vs Exactly-once

Input: 10 events. The supplied processor requires input and output topic
arguments, which were omitted from the PDF command examples; `dpw3-events` was
used as input with a separate output topic per mode.

| Parameter | At-least-once | Exactly-once |
|---|---:|---:|
| Total output | 11 | 10 |
| Unique event_id | 10 | 10 |
| Duplicate output | 1 | 0 |
| Failure injected | Yes | Yes |
| Output visible to the inspector | Yes | 10 committed records with `read_committed` |

At-least-once produced an output before the input offset was committed, so the
event was processed again after restart. Exactly-once used a Kafka transaction
for the output and input offset; the aborted first attempt was not visible to
the `read_committed` inspector.

The provided exactly-once script initially failed because
`send_offsets_to_transaction` requires `TopicPartition`, not a Kafka message.
The call was minimally corrected to pass the message's topic, partition, and
next offset; the experiment then produced 10 unique committed outputs.

## 6. Technical Reflection

Partitions provide parallelism while keeping the order of records within each
partition. A consumer group tracks offsets per partition and rebalances
assignments when a member fails. Committing after processing supports recovery,
but a failure before commit can cause replay and duplicate processing.
