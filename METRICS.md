# Metrics

This document describes the metrics and logs that PowerSync exposes, and potential ways to monitor them in New Relic.

## Replication Monitoring

### Queue size

- Not exposed directly in metrics.
- Logs show replication backlog progress (Replicating …, Flushed updates). You can parse these via NR Log Parsing rules to derive queue size if exposed.
- Alternative: Use powersync_replication_lag_seconds as a proxy (lag = effectively unprocessed queue).

### Processing latency

- Also not a direct metric.
- Use powersync_replication_lag_seconds for replication latency.
- Optionally parse logs for “Flushed … updates” timestamps to approximate processing times.

### Error rates

- From logs: look for "level":"error" messages, count grouped by time.
- Dashboard widget: Log count query (NRQL: SELECT count(*) FROM Log WHERE service='PowerSync' AND level='error' TIMESERIES).

### Retry counts

- Not in the Prometheus metrics.
- Parse logs for “retry” keywords and count occurrences.

### Active connections

Use powersync_concurrent_connections gauge (direct).

## Performance Monitoring

### Requests per second

- Not directly exposed.
- Approximate from powersync_transactions_replicated_total or powersync_rows_replicated_total rate:
    ```sql
    SELECT rate(sum(powersync_transactions_replicated_total), 1 minute) FROM Metric TIMESERIES
    ```

### Latency percentiles (p50, p90, p99)

- Not available yet — PowerSync doesn’t export request latency histograms.
- Workaround: Use database query telemetry if available, or derive from replication lag logs if detailed timing exists.


### Connection pool stats

No pool metrics exposed. If PowerSync logs pool activity, you can parse logs for connection metrics recorder lines.


### Sync Engines status

- Parse logs: "Successfully started Replication Engine", "Loaded sync rules".
- Create log-based widget: last seen engine init events.

### Scheduler activity
- From logs: "Running migrations", "Executing … snapshot-progress", "Starting Scheduler".
- Count over time to see scheduling patterns.

### Error logs trends

Simple NRQL on Log:
```sql
SELECT count(*) FROM Log WHERE service='powersync' AND level='error' TIMESERIES
```
