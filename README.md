# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

# Task 1: AIOps assessment

## Scenario and component purposes

The scenario represents an application handling requests through several supporting services. A degradation in one component can affect the entire request path, while producing separate alerts, log entries, and performance changes. The goal is to identify the impact quickly, correlate related symptoms, and guide an effective response.

 the major components have these purposes:

- Application service: Handles incoming requests and provides the user-facing functionality being monitored.
- Supporting components: Supply dependencies such as data, messaging, or infrastructure needed by the application; failures here can cause downstream impact.
- Monitoring and telemetry: Collects metrics, logs, traces, and health signals from the application and its dependencies.
- Alerting: Detects threshold breaches or unhealthy states and notifies operators of potential incidents.
- AIOps analysis: Correlates signals across components, identifies anomalies, suppresses duplicate noise, and helps determine the likely root cause.
- Response workflow: Uses the analysis to prioritize the incident, investigate the affected component, and coordinate remediation and recovery.





# Task 2: Analyse Logs and Metrics

1) Fields that represent metrics
These are the numeric measurements that indicate performance and resource usage:
response_time_ms
cpu_percent
memory_percent
These are the operational metrics because they vary over time and directly reflect service health/performance.

2) Fields that represent log information
These are the textual telemetry fields:
log_level
message
The field service is not a metric or log entry itself; it is a label identifying which component produced the observation.

3) How timestamps are used
The timestamp field is an ISO 8601 time stamp, recorded once per minute:
2026-09-20T10:00:00
2026-09-20T10:01:00
...
2026-09-20T10:09:00
This creates a chronological sequence showing how the service behaves over time. It allows correlation between:
a rising response time,
higher CPU and memory use,
and the corresponding log severity/message.
The pattern shows a normal stable period, followed by a short incident spike, then recovery.

4) Observations that appear normal
Normal behaviour is represented by records with:
response_time_ms around 120–150 ms
cpu_percent around 42–50
memory_percent around 51–57
log_level = INFO
message = “Payment request processed successfully”
This applies to the records from:
10:00
10:01
10:02
10:03
10:04
10:07
10:08
10:09
These are consistent, stable, low-noise observations and look like normal service operation.

5) Observations that appear unusual
Unusual behaviour is clearly visible at:
10:05
10:06
These records show:
response_time_ms jumps to 610 and 640 ms
cpu_percent rises to 75 and 94
memory_percent rises to 70 and 91
log_level = ERROR
messages:
“Payment service timeout”
“Database connection timeout”
This indicates an incident or service degradation, likely tied to timeout conditions and resource saturation. The fact that the values return to normal afterward suggests the problem was transient and resolved quickly.




# Task 3: Identify Anomalies

Use the provided anomaly-detection component to analyse the operational data.

The detection mechanism processes the dataset in chronological order and checks each observation against the configured thresholds:
- response_time_threshold = 500 ms
- cpu_threshold = 80%
- memory_threshold = 80%

The detector flags an observation as anomalous when any of these metric thresholds are exceeded and records the relevant reason(s). It also records a concerning log event when the observation has a log_level of ERROR.

## Verified detection result

The detection process processes all available operational records and identifies the abnormal observations as follows:

### Anomaly 1
- Timestamp: 2026-09-20T10:05:00
- Service: payment-service
- Metric information:
  - response_time_ms = 610
  - cpu_percent = 75
  - memory_percent = 70
- Log information:
  - log_level = ERROR
  - message = "Payment service timeout"
- Why it was flagged:
  - High response time
  - Error log detected

### Anomaly 2
- Timestamp: 2026-09-20T10:06:00
- Service: payment-service
- Metric information:
  - response_time_ms = 640
  - cpu_percent = 94
  - memory_percent = 91
- Log information:
  - log_level = ERROR
  - message = "Database connection timeout"
- Why it was flagged:
  - High response time
  - High CPU utilization
  - High memory utilization
  - Error log detected

## Normal observations vs anomalous observations

Normal observations are the records from 10:00, 10:01, 10:02, 10:03, 10:04, 10:07, 10:08 and 10:09, where:
- response_time_ms stays around 120–150 ms
- cpu_percent stays around 42–50%
- memory_percent stays around 51–57%
- log_level is INFO
- message is "Payment request processed successfully"

These are not flagged because they do not exceed the configured thresholds and do not include concerning error events.

## Review summary
- Anomalies detected: 2
- Relevant metric/log information: response_time_ms, cpu_percent, memory_percent, log_level, message
- Expected anomaly missed: none in the provided dataset
- Normal event incorrectly flagged: none

The detector produces a readable anomaly output that includes the timestamp, service, type, and list of reasons, which is sufficient to explain why each record was flagged.

## Limitation / possible improvement
A limitation of this approach is that it relies on fixed thresholds alone. It does not consider time-window correlation or cross-signal context, so a future improvement could be to require multiple related signals (for example high latency + elevated CPU/memory + ERROR log) before raising a high-confidence incident alert.







# Task 4: Verify the AIOps Event Flow

The repository contains a lightweight event-stream simulation for anomaly propagation. This workflow validates that a detected anomaly is transformed into an event, passed to the producer, published to the topic, consumed, and then made available to downstream AIOps processing.

## Components and roles
- Producer: emits the anomaly event to the topic.
- Topic: stores and distributes the event to subscribers.
- Consumer: reads events from the topic.
- Event/message: the structured payload containing anomaly details.

## Verified event flow
The workflow confirms the following:
1. An anomaly identified by the detection process results in an event.
2. The event is passed to the producer.
3. The producer publishes the event to the appropriate topic.
4. The consumer receives the event from the topic.
5. The consumer processes the received event.
6. The processed event reaches the downstream AIOps component.

## Execution result
Running the provided workflow produces the following result:
- Records processed: 10
- Anomalies detected: 2
- Events consumed: 2

The detected anomaly events are:
- 2026-09-20T10:05:00, payment-service, ANOMALY, Reasons: High response time, Error log detected
- 2026-09-20T10:06:00, payment-service, ANOMALY, Reasons: High response time, High CPU utilization, High memory utilization, Error log detected

This confirms that the anomaly event successfully traveled through the complete event-processing pipeline and was available to the downstream component.






# Task 5: Investigate and Correct the Workflow

The assessment environment contained a small number of issues that prevented the AIOps workflow from operating as expected. These problems were investigated and corrected within the existing architecture without replacing the provided components.

## Problem 1: Incorrect log condition in the detector
- Affected component: anomaly detector
- Cause: the detector checked for log_level == "WARNING" instead of the real concerning condition, which is ERROR.
- Correction: updated the detector to treat ERROR log entries as anomalous and include "Error log detected" in the event reasons.
- Verification: the pipeline now includes the relevant timeout log entries in the anomaly reasons for the 10:05 and 10:06 observations.

## Problem 2: Producer and consumer were connected to different topics
- Affected component: AIOps pipeline
- Cause: the pipeline created separate topic instances for publishing and consuming, so the event never reached the consumer.
- Correction: the workflow now uses a shared anomaly-events topic for both publish and consume operations.
- Verification: the final workflow consumes the same events that were published, producing 2 consumed events matching the 2 detected anomalies.

## Problem 3: Module import compatibility issue
- Affected component: pipeline import path
- Cause: the workflow used direct imports that failed when the project was executed in a package/test-aware environment.
- Correction: added compatible fallback imports so the existing architecture works both as a script and when imported via the package path.
- Verification: the test suite and execution workflow both run successfully.

## Execution verification
The corrected workflow now successfully validates the end-to-end event flow:
- 4 tests passed
- 10 records processed
- 2 anomalies detected
- 2 events consumed

This confirms that the issues were fixed within the original architecture and the AIOps event pipeline operates correctly again.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


