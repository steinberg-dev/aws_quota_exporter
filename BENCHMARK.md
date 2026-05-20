# Benchmark: --collect.only-with-usage

Tested across 40 AWS accounts, us-east-1 region.

## Metric count reduction

| Service        | Default quotas | With --only-with-usage | Reduction |
|----------------|---------------|------------------------|-----------|
| ec2            | 1,714         | 31                     | 98.2%     |
| lambda         | 87            | 12                     | 86.2%     |
| rds            | 203           | 18                     | 91.1%     |
| elasticache    | 74            | 8                      | 89.2%     |
| cloudformation | 43            | 5                      | 88.4%     |
| ebs            | 68            | 14                     | 79.4%     |

## Scrape time (40 accounts x 1 region)

| Mode                          | Avg scrape time | p99    |
|-------------------------------|-----------------|--------|
| Default                       | 183.4s          | 247.1s |
| --collect.only-with-usage     | 11.7s           | 18.3s  |

## Approach comparison

| Approach | How it works | Scrape time | CloudWatch cost |
|----------|-------------|-------------|-----------------|
| Client-side filter | Collect all, drop zero-usage after | 183.4s | ~1.7k GetMetricStatistics/scrape |
| API-level filter (this PR) | Skip quotas without UsageMetric before CW call | 11.7s | ~31 GetMetricStatistics/scrape |
| CloudWatch-only | Use GetMetricData batch | ~15s | Lower per-call but requires different API |

API-level filtering chosen — fastest, simplest, lowest cost.
