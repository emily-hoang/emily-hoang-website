# 🔍 Observability in Performance Testing: Seeing Beyond the Numbers

A common misconception in performance testing is that success is defined by smooth throughput and latency charts — if the graphs look stable, the work is done. But over time, I’ve learned that performance testing goes far deeper than that. By carefully observing metrics like the **golden signals**, examining **error logs, distributed traces**, and even reviewing your **load testing tool’s own logs**, you start to uncover the real story behind your system’s behaviour — what it’s struggling with, where it’s resilient, and what it’s truly trying to tell you.

Observability isn’t just a post-test task or something you add after the fact — it’s the foundation of understanding how your system truly performs under stress. Without proper observability, performance testing becomes guesswork. Metrics, traces, and logs give you visibility into what’s happening beneath the surface — how each service responds to load, where latency builds up, and which dependencies start to fail first. With strong observability in place, you can go beyond identifying *that* something went wrong to uncovering *why* it happened, enabling faster root-cause analysis, better tuning, and ultimately, a more resilient system. 

In this post, I’ll dive into how to apply **observability principles** during performance testing, how to interpret **the Golden Signals**, and how to connect metrics and logs to actionable insights.

---

## 🧠 The Role of Observability in Performance Testing

### 🔍 **Definition**

> Observability is the ability to measure the internal state of a system by examining its external outputs — such as logs, metrics, and traces, etc.
> 

When you run a performance test, your load tool tells you what the user sees — response times, error counts, throughput.

But observability tells you what the *system* experiences while delivering those responses.

Without observability, you’re testing blindfolded. (As this point, if your system hasn’t got observability setup, it’s probably a good idea to start looking into observability tools such as Data Dog, New Relic, etc.)

The real goal is to correlate **external symptoms** (e.g. latency spikes) with **internal causes** (e.g. saturation, cache misses, or thread contention, concurrency issues, etc.).

### Observability gives you three key capabilities:

1. **Detect** — Notice when something abnormal happens (e.g. sudden latency increase).
2. **Diagnose** — Trace the issue back to its source (e.g. a specific service, dependency, or query).
3. **Decide** — Identify the right mitigation (e.g. scaling policy, caching strategy, or query optimization).

---

## ⚙️ The Four Golden Signals

[Google’s **Golden Signals**](https://sre.google/sre-book/monitoring-distributed-systems/) framework is one of the most effective ways to interpret performance test results through an observability lens.

| Signal | Description | Example Issues | What to Look For |
| ---- | ---- | ---- | ---- |
| **Latency** | How long requests take to be served | Slow database queries, network congestion | Spikes in p95/p99 latency during load |
| **Throughput** | How much traffic your system is handling | Load imbalance, capacity limits | Drops in throughput, uneven node utilization |
| **Errors** | The rate of failed requests | Dependency failures, misconfigurations, unhandled exceptions | Error bursts from one endpoint or service |
| **Saturation** | How “full” your system resources are | CPU, memory, thread pools, DB connections | Sustained high utilization over 80% |

These four signals act as your compass during any performance test.

Let’s break down how to read each one effectively.

---

## 📊 1. Latency — The First Red Flag

Latency is usually the first signal that something’s wrong.

It’s easy to celebrate a test with a 100% success rate — until you realise your 99th percentile latency is five seconds (unless you have a known dependency that takes 4.5s to return a response, and the risks are accepted by stakeholders).

### What to check:

- **p95 / p99 spikes**: Look for sudden increases that align with load phases.
- **Distribution skew**: If median latency is fine but p99 isn’t, you have outlier behavior — often caused by lock contention, cold caches, or uneven load distribution.
- **Trace breakdowns**: Use distributed tracing (e.g., OpenTelemetry, Data Dog, New Relic) to see where time is spent — network, database, or downstream calls.

**Tip:**

Plot latency over time and correlate with resource metrics — a CPU spike preceding latency growth often points to compute bottlenecks; a stable CPU but rising latency hints at I/O or external dependencies.

---

## 🌐 2. Throughput — The Hidden Context

Throughput tells you *how much* work your system is doing.

Throughput (requests per minute) and concurrency (active users) help you interpret the other signals in context.

### What to check:

- **Plateaued throughput**: If throughput stops increasing while requests keep coming, you’ve hit a bottleneck (e.g., connection pool limit, thread starvation, rate limiting).
- **Load imbalance**: Compare throughput across instances — if one pod handles 80% of the load, check your load balancer or service mesh configuration.
- **Scaling responsiveness**: Monitor autoscaler metrics — does it react quickly enough, or too late?

**Tip:**

Visualise throughput and latency together. Throughput rising while latency remains stable = good scaling.

Throughput flattening while latency climbs = saturation.

---

## 🚨 3. Errors — The Truth Behind Failures

Errors are obvious, but their meaning often isn’t.

A 5xx spike doesn’t tell you *why* things broke — that’s where observability and structured logging come in.

### What to check:

- **Error codes breakdown**: Differentiate between 4xx (client) and 5xx (server) errors. E.g. A 403 error means your test user doesn’t have permission to access the resource. It’s time to review your test data setup.
- **Error correlation**: Align error spikes with specific endpoints or dependencies.
- **Error context**: Use structured logs to extract fields like `trace_id`, `endpoint`, `dependency_name`, and `exception_type`.

**Example:**

```json
{
  "trace_id": "abc123",
  "service": "invoice-api",
  "endpoint": "/v2/invoices",
  "dependency": "payment-gateway",
  "status": 500,
  "exception": "TimeoutException",
  "duration_ms": 4800
}

```

When visualized in your logging platform (e.g., Sumo Logic, Splunk, New Relic, Datadog), these logs can be grouped by root cause — for instance, 80% of 5xx errors coming from one downstream call.

**Tip:**

Not all errors are equal. A small error rate with large response latency can be worse than high errors with fast retries — always analyse both together. 

And did I forget to mention that a high error rate during a load test is not 100% failure? When your system experiences a high error rate (e.g >80%), it’s your opportunity to assess the system resiliency. Look for logs related to retry, timeout, and circuit breaker, etc! Those are what your system will show when this type of error happens in a real production environment. Are they working as expected? Should we increase/decrease the total timeout?

---

## 🧮 4. Saturation — The Silent Killer

Saturation metrics measure how close your system is to its limits.

- **CPU Utilization:** Consistently >85% means limited headroom.
- **Memory Usage:** Steady increase → potential leak or unbounded caching.
- **Thread Pool / Connection Pool:** When maxed out, new requests queue and latency skyrockets.
- **GC (Garbage Collection):** Long GC pauses can freeze Java or Node services temporarily.

**Tip:**

Run a quick estimation of how much resources you will need, given the expected throughput, to avoid going back and forth with your test setup and schedule. Correlate saturation spikes with your Golden Signals.

For example:

- CPU is over 80% → latency and error spikes soon after.
- Memory pressure → increased GC time → latency drift.
- Connection pool exhaustion → rising queue wait time.

---

## 🧩 Putting It Together: Finding the Root Cause

Here’s how I usually approach performance issue analysis:

| Observation | Likely Cause | Verification Step | Potential Fix |
| --- | --- | --- | --- |
| High latency + stable CPU | Downstream dependency | Check external API metrics | Cache or retry with backoff |
| High errors + high CPU | App bottleneck | Analyze flame graph or profiler | Fix any errors during the test, optimize code path or add autoscaling |
| Plateaued throughput + high latency | Saturation, downstream dependency | Inspect connection/thread pools | Increase pool size or check duration of requests being sent to dependencies |
| Gradual latency increase over time | Memory leak or GC | Check heap usage trend | Fix caching or tune GC parameters |

The key is correlation — no single metric tells the full story.

The truth lies at the intersection of metrics, logs, and traces.

---

## 🔧 Observability Best Practices for Performance Testing

1. **Tag everything** — Use consistent metadata (`service`, `env`, `version`) in logs and metrics. Have you heard of [Unified Service Tagging](https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes) by Datadog? It allows you to tie all telemetry together across traces, metrics, logs, deployment, etc.
2. **Use tracing** — Tools like OpenTelemetry or New Relic let you follow a single request through every layer of a service. It's invaluable for pinpointing latency sources.
3. **Choose the right tests** — Long-running soak tests expose memory leaks and slow degradation while spike test examines the system with a short busrt of traffic which is useful for understanding your system scalability.
4. **Save your test window** — Annotate dashboards during the test run for easy post-analysis.
5. **Correlate early** — Don’t wait until post-test analysis — watch metrics as the load runs to ensure exceptions can be mitigated early.
6. **Share findings visually** — Screenshots, charts, and annotated dashboards make discussions far more productive and easier to come up with recommendations.

---

## 🧭 Wrapping Up

Performance testing without observability is like driving with your eyes closed.

Metrics tell you *that* something’s wrong. Observability tells you *why.*

By focusing on the **Golden Signals**, correlating them with logs and traces, and understanding how saturation propagates through your system, you gain a clear, actionable view of performance bottlenecks.

Because the ultimate goal of performance testing isn’t just passing a load test — it’s building systems that are **understandable, resilient, and ready for the real world**.