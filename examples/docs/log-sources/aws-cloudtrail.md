---
source: aws-cloudtrail
selector: {product: aws, service: cloudtrail}
query_scope: index=logs-aws.cloudtrail-* AND event.dataset:aws.cloudtrail
generated: 2026-11-12, from 30 days of data
---
Events in window: 48,210,334

<!--
EXAMPLE. Field names follow the Elastic AWS integration (ECS). The counts, fill rates and
example values are ILLUSTRATIVE. They show what a generated doc looks like and aren't
measurements. Generate your own with the log-source-docs skill.
-->

# AWS CloudTrail

## What it is
AWS control-plane (and optionally data-plane) API activity across all accounts in the
organization trail. Good for detecting identity abuse, persistence through IAM, defense
evasion against logging, and unusual access to sensitive services.

## Top event types
| `event.action` | Events | Share |
|---|---|---|
| AssumeRole | 11,902,551 | 24.7% |
| GetObject | 7,310,882 | 15.2% |
| Decrypt | 5,904,117 | 12.2% |
| DescribeInstances | 2,118,430 | 4.4% |
| GetCallerIdentity | 1,644,092 | 3.4% |
| ConsoleLogin | 18,204 | <0.1% |
| StopLogging | 3 | <0.1% |

## Field reference
| Field | Fill | Example (sanitized) | Notes |
|---|---|---|---|
| `event.action` | 100% | `StopLogging` | The CloudTrail `eventName`. PascalCase, matched exactly |
| `event.provider` | 100% | `cloudtrail.amazonaws.com` | The CloudTrail `eventSource`. Lowercase |
| `event.outcome` | 100% | `failure` | `success` / `failure`, derived from whether an error code is present |
| `aws.cloudtrail.error_code` | 3.1% | `AccessDenied` | Present only on failures |
| `aws.cloudtrail.user_identity.type` | 100% | `AssumedRole` | `IAMUser`, `AssumedRole`, `Root`, `AWSService`, `FederatedUser`, … |
| `aws.cloudtrail.user_identity.arn` | 96.0% | `arn:aws:sts::123456789012:assumed-role/app-deploy/session-01` | Empty for some `AWSService` calls |
| `aws.cloudtrail.user_identity.access_key_id` | 88.4% | `ASIAEXAMPLEEXAMPLE01` | Credential. Redact to the last 4 characters in output |
| `user.name` | 91.2% | `app-deploy` | Role or user name, **not** the session name. Attacker-choosable via role session tags; don't build exclusions on it |
| `source.ip` | 97.5% | `203.0.113.10` | For AWS-internal calls the raw value is a service hostname, so `source.ip` is empty and `source.address` holds the hostname |
| `source.as.number` | 94.8% | `64496` | Enrichment. Good exclusion key for known corporate egress |
| `source.geo.country_iso_code` | 94.8% | `US` | ISO-3166 alpha-2, **not** the country name |
| `user_agent.original` | 99.1% | `aws-cli/2.15.0 Python/3.11.6 …` | Free text, attacker-controlled |
| `cloud.account.id` | 100% | `123456789012` | 12-digit string |
| `cloud.region` | 100% | `us-east-1` | |
| `aws.cloudtrail.request_parameters` | 71.3% | `{name=org-trail}` | Flattened string, not an object. Match with wildcards |

## Known empty
| Field | Why it's empty |
|---|---|
| `aws.cloudtrail.additional_eventdata.mfa_used` | Populated only on `ConsoleLogin`. Empty on every API call |
| `source.geo.city_name` | The GeoIP database in the pipeline has country-level data only |

## Gotchas
- Delivery lag is 5–15 minutes. For scheduled rules, let the lookback cover the lag or
  search on ingest time.
- The organization trail and per-account trails can deliver the same event twice. Deduplicate
  on `event.id` when counting.
- `AssumeRole` dominates the volume. Scope on `event.action` before any aggregation.

## Hand-written notes
- The trail for the security-tooling account is excluded at ingest. Rules can't see its activity.
