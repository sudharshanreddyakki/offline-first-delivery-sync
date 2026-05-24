# Offline-First Delivery Synchronization Engine

<div align="center">

![React Native](https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![SQLite](https://img.shields.io/badge/SQLite-07405E?style=for-the-badge&logo=sqlite&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![Apache Kafka](https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

**A battle-tested synchronization architecture for delivery operations at scale — where rural network dropouts, duplicate writes, and stale state cannot break field execution for 1,200+ daily drivers.**

</div>

---

## The Problem

Delivery drivers in low-connectivity regions (2G/3G/4G spotty) need their apps to keep working during network outages. Traditional REST-based architectures fail here because duplicate writes after reconnect overwrite valid local data, stale server state bypasses unsynced local mutations, and no idempotency means the same action fires multiple times.

---

## Architecture

MOBILE APP CLIENT
- React Native UI: Driver actions create local mutations
- SQLite Core DB: Local source of truth during outages
- Critical Trigger: Sets is_dirty=1 and generates UUIDv4

NETWORK BOUNDARY: SPOTTY 2G/3G/4G
- 30s Sync Loop with retry and defensive backoff
- Idempotency Key Attached: Prevents duplicate writes after dropped ACKs

BACKEND RECONCILIATION PATH
- API Gateway / LB: Receives sync payloads
- Reconciliation Layer: Redis idempotency check + version locking
- PostgreSQL: Authoritative persistent database
- Proposed Kafka Write Buffer: 100x scale mitigation for burst ingestion

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| P50 Sync Latency | 142ms |
| P95 Sync Latency | 1.8s |
| P99 Sync Latency | 6.4s |
| Average Payload | 2.4 KB |
| Maximum Payload | 4.2 MB |
| Sync Success Rate | 99.94% |
| Conflict Rate | 0.42% |
| Duplicate Mutation Attempts | 3.1% |

---

## Key Engineering Decisions

### Idempotency Keys (UUIDv4)
Every mutation gets a UUID on the mobile client before it hits the network. The reconciliation layer checks Redis before writing to PostgreSQL - dropped ACKs produce safe retries instead of duplicate writes or silent data loss.

### Optimistic Concurrency Control
Route ownership follows a strict one-driver-per-route constraint. OCC + Last-Write-Wins backend queue was chosen over CRDTs and vector clocks, which would add metadata cost and complexity for a conflict model the domain did not require.

### The "Clean Slate" Bug
The original sync flow allowed inbound server state to bypass the mutation queue and overwrite unsynchronized local rows. The redesign forced outbound-first ordering and idempotency-key processing before reconciliation.

---

## Validation

Local simulation used 50 concurrent client threads, synthetic packet loss, network jitter up to 4,000ms, and forced drop/reconnect cycles. A Python verifier checked incoming UUIDv4 mutation integrity across 10,000 simulated offline mutations.

---

## Scale Assessment

At 120,000 daily drivers, PostgreSQL connection pools saturate and write-heavy reconciliation tables face lock contention. The mitigation path introduces Redis for hot idempotency validation and Kafka as a buffered ingestion layer before persistent writes.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Mobile Client | React Native |
| Local Storage | SQLite |
| Sync Transport | REST API + retry backoff |
| Idempotency | Redis |
| Persistent DB | PostgreSQL |
| Scale Buffer (proposed) | Apache Kafka |
| Containerization | Docker |

---

## Author

**Sudharshan Reddy Akki** - Full-Stack Software Engineer
MS Computer Science, University of Illinois Springfield

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/sudharshanreddyakki)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=flat&logo=github&logoColor=white)](https://github.com/sudharshanreddyakki)
