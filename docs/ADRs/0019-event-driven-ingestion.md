# ADR-0019: Event-Driven Ingestion Pattern

| Field | Value |
|:------|:------|
| Status | accepted |
| Date | 2026-07-29 |
| Decision-makers | @jpower432 |

## Context and Problem Statement

The ComplyTime API stores compliance evidence objects submitted by automated pipelines. Downstream systems need to act on newly stored evidence — indexing it, triggering assessments, or aggregating it for reporting. How should the ComplyTime API notify consumers of new evidence without requiring them to poll for changes?

The API is designed to serve multiple independent consumer systems, each with its own processing cadence and failure domain. Synchronous coupling to any consumer would introduce latency and failure-mode dependencies the API must not own.

## Decision Drivers

* The ComplyTime API should not need to change when a new consumer is added or removed.
* Multiple consumers may independently process the same ingestion event for different purposes.
* Missed events mean consumers operate on stale or incomplete evidence.
* The ComplyTime API should not dictate or depend on consumer behavior.
* Consumers may operate in environments with intermittent connectivity; the pattern must not require persistent producer-consumer reachability.

## Considered Options

* Pub/sub event bus
* Consumer polling
* Direct webhook push to registered consumers

## Decision Outcome

Chosen option: "Pub/sub event bus", because it delivers events to multiple independent consumers without the ComplyTime API tracking or coupling to any of them.

### Confirmation

The ComplyTime API publishes an ingestion event for every stored evidence object. At least one downstream consumer successfully subscribes and processes events without a ComplyTime API code change. Failed deliveries route to a dead-letter queue for investigation.

## Consequences

* Good, because consumers subscribe independently — the ComplyTime API has no knowledge of who is listening.
* Good, because multiple consumers receive the same event without coordination between them.
* Good, because retry and dead-letter logic is handled by the bus, not the ComplyTime API.
* Bad, because operating an event bus adds infrastructure complexity.
* Bad, because at-least-once delivery requires consumers to handle duplicate events idempotently.
* Neutral, because the event bus technology and delivery mechanism must support backpressure control and consumers with intermittent connectivity.

## Pros and Cons of the Options

### Pub/sub event bus

A producer publishes events to a topic. Any number of subscribers receive events independently.

* Good, because producer and consumer are fully decoupled.
* Good, because fan-out to N consumers requires no ComplyTime API changes.
* Good, because durable delivery and dead-lettering are bus-level concerns.
* Bad, because introduces an event bus as an infrastructure dependency.

### Consumer polling

Consumers periodically query the ComplyTime API or evidence store for new objects.

* Good, because no additional infrastructure required.
* Bad, because latency is bounded by poll interval, not ingestion time.
* Bad, because each consumer independently manages polling state and backoff.
* Bad, because load scales with the number of consumers and poll frequency.

### Direct webhook push to registered consumers

The ComplyTime API maintains a registry of consumer endpoints and pushes HTTP callbacks on ingestion.

* Good, because no broker infrastructure required.
* Bad, because the ComplyTime API must own consumer lifecycle: registration, health checks, retry.
* Bad, because adding a consumer requires a ComplyTime API configuration change.
* Bad, because a slow or failing consumer endpoint can block or degrade ingestion.
* Bad, because push delivery requires the consumer endpoint to be reachable; air-gapped or intermittently connected consumers cannot receive events.

## More Information

* Batch ingestion is out of scope for this ADR and will be captured separately.
* ADR-0020: Event Bus Technology Selection
* ADR-0021: CloudEvents for Ingestion Event Envelope
* ADR-0022: AsyncAPI for Event-Driven Interface Documentation
