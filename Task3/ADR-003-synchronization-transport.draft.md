# ADR-003: Asynchronous Farm Synchronization Transport (Draft)

## Status

Draft

## Decision

Use a dedicated Kafka topic for asynchronous Farm Synchronization. Retain the hot history in Kafka for approximately 30 days. The Archiver Service stores older synchronization messages as archive objects in the Existing S3 Data Lake at least monthly.

Operate a dedicated Kafka cluster for Farm Synchronization. Do not use the metrics topics for farm synchronization.

## Dedicated Kafka benefits

- Does not affect existing production workloads.
- Provides independent capacity, retention, ACLs, quotas, and schemas.
- Simplifies tenant isolation for future SaaS.
- Allows Farm Synchronization to evolve independently.

## Existing Kafka risks

- Introducing ACLs may break current clients.
- Farm traffic may compete with production workloads.
- Shared retention and capacity policies create coupling.
- Tenant isolation becomes harder to enforce.
- Operational changes may affect unrelated systems.

## Assumptions for MVP sizing

Per farm, with batched telemetry:

- Farm Events: 10–100 messages/day.
- Feed facts: 24–96 messages/day.
- Livestock-count facts: approximately 96 messages/day, assuming a 15-minute interval.
- Configured metrics: according to their configured interval; an hourly metric produces 24 messages/day.
- Raw telemetry: not retained centrally; only derived facts, current state, Farm Events, and selected evidence metadata are synchronized.

Expected central synchronization volume is approximately 150–300 messages per farm per day, excluding video/audio.

## Message-size assumption

Use approximately 2 KB average uncompressed message size, with a 10 KB planning limit for ordinary messages. Evidence messages contain metadata and object references, not video/audio binaries.

## Durability and retention

- Use replicated Kafka topics with a replication factor of at least three where infrastructure permits.
- Use producer acknowledgements equivalent to `all` and idempotent producers.
- Use consumer offsets and replayable topics.
- Include a farm identifier, message identifier, event time, schema version, and idempotency key.
- Keep Local Storage as the Farm Agent’s retry/outbox store while disconnected.
- The Archiver Service writes cold history to the Existing S3 Data Lake at least monthly.

## Rejected alternative

RabbitMQ provides durable delivery queues, but it is not suitable as the primary hot-history store: acknowledged messages leave the queue, and replayable historical retention requires additional storage and application logic.
