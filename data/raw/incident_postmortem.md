# Postmortem: Search API latency on 2026-01-22

Document ID: `incident-search-2026-01-22`  
Classification: Internal  
Severity: SEV-2

## Summary

From 14:07 to 14:49 UTC, the customer support search API had a p95 latency of 8.2 seconds. Approximately 31% of search requests exceeded the two-second service objective. No customer data was lost.

## Root cause

A deployment enabled exact exhaustive vector search for all tenants instead of the approximate index. At the same time, a catalog re-index increased the collection from 1.2 million to 4.8 million vectors. The combination saturated CPU on all six search workers.

## Recovery

At 14:36 UTC the on-call reverted to approximate search. Latency returned below the objective at 14:49 UTC after queued requests drained. Two additional workers were temporarily added but were not the primary fix.

## Corrective actions

| Action | Owner | Due date | Status |
| Add a load-test gate for index-mode changes | Search Platform | 2026-02-14 | Complete |
| Alert when exact search is enabled above 250k vectors | Reliability | 2026-02-20 | In progress |
| Add collection-size data to deployment preview | Developer Experience | 2026-03-01 | Planned |

## Lessons

Configuration changes that appear harmless at small scale must be tested against production-like collection sizes. Extra workers can mask the symptom, but they do not correct an unsuitable retrieval algorithm.

