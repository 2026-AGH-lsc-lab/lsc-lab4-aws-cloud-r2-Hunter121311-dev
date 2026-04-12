# Report

## Assignment 1: Deploy All Environments

All four environments were deployed and exercised in later scenarios:

- Lambda (zip)
- Lambda (container image)
- Fargate
- EC2

From the Scenario A and Scenario B outputs, all tested endpoints returned `200` for all requests.

However, the exact terminal output showing the **same query vector returning identical k-NN result arrays across all four endpoints** is not included in the provided logs.

**Status:** Partially complete  
**Missing:**  
`results/assignment-1-endpoints.txt` showing identical outputs across all four endpoints

---

## Assignment 2: Scenario A — Cold Start Characterization

### Observations

After 20+ minutes idle:

- **Lambda zip**
  - Slowest latency: **1.2782 s**
  - p50 latency: **0.1039 s**

- **Lambda container**
  - Slowest latency: **1.0146 s**
  - p50 latency: **0.0970 s**

### CloudWatch Data

- **Lambda zip**
  - Init Duration: **634.31 ms**
  - Handler Duration: **81.49 ms**

- **Lambda container**
  - Init Duration: **604.47 ms**
  - Handler Duration: **77.56 ms**

Each deployment had **1 cold start**.

---

### Cold Start Decomposition

Formula:

- Cold start RTT = total latency − Init − Handler

| Variant | Total (ms) | Init (ms) | Handler (ms) | Network RTT (ms) |
|--------|-----------:|----------:|-------------:|-----------------:|
| Lambda zip | 1278.20 | 634.31 | 81.49 | 562.40 |
| Lambda container | 1014.60 | 604.47 | 77.56 | 332.57 |

---

### Interpretation

- Container cold start slightly faster than zip:
  - Init difference: ~30 ms
  - End-to-end difference: ~264 ms

- Cold starts increase latency by ~10× vs warm requests.

---

### Warm Latency (from Scenario A)

| Variant | p50 (ms) | p95 (ms) |
|--------|---------:|---------:|
| Lambda zip | 103.9 | 151.5 |
| Lambda container | 97.0 | 153.6 |

**Note:** Warm server-side durations not yet extracted.

---

## Assignment 3: Scenario B — Warm Steady-State Throughput

### Latency Table

| Environment | Concurrency | p50 (ms) | p95 (ms) | p99 (ms) | Server avg (ms) |
|------------|------------:|---------:|---------:|---------:|----------------:|
| Lambda (zip) | 5 | 95.2648 | 115.2653 | 128.0158 | TBD |
| Lambda (zip) | 10 | 96.1489 | 116.3900 | 161.8262 | TBD |
| Lambda (container) | 5 | 92.1562 | 112.8111 | 138.0922 | TBD |
| Lambda (container) | 10 | 86.8830 | 109.0254 | 163.5375 | TBD |
| Fargate | 10 | 804.5 | 1093.3 | 1187.8 | TBD |
| Fargate | 50 | 4084.4 | 4603.1 | 4774.5 | TBD |
| EC2 | 10 | 194.0625 | 246.0448 | 281.4210 | TBD |
| EC2 | 50 | 973.2 | 1028.4 | 1046.0 | TBD |

---

### Tail Latency Check (p99 > 2×p95)

No instability detected in any configuration.

---

### Throughput

| Environment | Concurrency | Requests/sec |
|------------|------------:|-------------:|
| Lambda (zip) | 5 | 55.64 |
| Lambda (zip) | 10 | 104.48 |
| Lambda (container) | 5 | 57.54 |
| Lambda (container) | 10 | 117.22 |
| Fargate | 10 | 12.22 |
| Fargate | 50 | 12.17 |
| EC2 | 10 | 50.86 |
| EC2 | 50 | 51.14 |

---

### Analysis

#### Lambda behavior

- p50 remains stable between concurrency 5 → 10
- Reason: separate execution environments (no queueing)

#### Fargate & EC2 behavior

- Latency increases significantly at higher concurrency
- Cause: request queueing on single instance/task

#### Best performance

- **Lambda container (c=10)**
  - p50: 86.9 ms
  - p95: 109.0 ms
  - p99: 163.5 ms

---

### Client vs Server Latency

Client latency includes:

- Network overhead
- API Gateway / HTTP overhead
- Serialization/deserialization
- Queueing delays

Therefore:

**client p50 > server query_time_ms**

---

## Assignment 4: Scenario C — Burst from Zero

During export of aggregated logs, the account got deactivated - all scenarios passed through
after each scenario there was a break over one hour (except coded sleeps - i didnt touch
anything during execution of scripts) after each scenario I:
- displayed into file required filtered logs
- deactivated lab
- after at least 45 minutes:
  - start new lab
  - set up all credentials
  - update ec2 ip in endpoints.sh
  - source endpoints
  - run scenario
- repeat

at end I tried to export to my local machine logs, but unsuccessfully - account got deactivated
surprisingly it happened after execution of everything during "waiting"


### Required:

- 200-request burst tests
- Lambda cold-start counts from CloudWatch
- Latency distributions for all environments

### Expected Outcome

- Lambda shows bimodal distribution:
  - Warm cluster
  - Cold-start cluster

- Likely fails **p99 < 500 ms SLO** without mitigation

---

## Assignment 5: Cost at Zero Load

### Key Insight

- **Lambda idle cost = $0**
- Fargate and EC2 incur continuous cost

---

### Pricing References

- Lambda: $0.20 / 1M requests  
- Fargate:
  - $0.000011244 per vCPU-second
  - $0.000001235 per GB-second
- EC2 (examples):
  - t3.small: $0.0208/hr
  - t3.medium: $0.0416/hr

---

### Idle Cost Formula

Assuming 18 idle hours/day:

- Lambda: **$0**
- Fargate: `hourly_rate × 18 × 30`
- EC2: `hourly_rate × 18 × 30`

---

## Assignment 6: Cost Model & Recommendation

### Traffic Model

- Peak: 100 RPS (30 min/day)
- Normal: 5 RPS (5.5 hr/day)

Monthly requests:

- **8.37 million**

---

### Lambda Cost

Request cost:

- `8.37 × 0.20 = $1.674`

Total cost formula:
Cost = 1.674 + 69.7501 × duration_seconds

Approximation:

- Zip: ~$8.32/month
- Container: ~$7.73/month

---

### Break-even Formula
R_break_even = Cf / (0.5184 + 21.6000432 × d)

Where:

- `Cf` = monthly Fargate cost
- `d` = handler duration (seconds)

---

## Recommendation

### Best choice: **Lambda (container or zip)**

#### Why:

- Lowest latency:
  - p50 ~ 87–96 ms
- Zero idle cost
- Scales automatically

---

### Limitation

- Cold starts:
  - 1.0–1.3 seconds observed
- Violates p99 < 500 ms during burst

---
