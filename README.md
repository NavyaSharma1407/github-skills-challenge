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


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

