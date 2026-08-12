# ADR-0022: AsyncAPI for Event-Driven Interface Documentation

| Field | Value |
|:------|:------|
| Status | accepted |
| Date | 2026-07-29 |
| Decision-makers | @jpower432 |

## Context and Problem Statement

The ComplyTime API exposes an event-driven interface: consumers subscribe to a NATS JetStream subject and receive CloudEvents-enveloped ingestion events. Consumers need a published, versioned contract to integrate without reading implementation code. OpenAPI documents REST APIs but does not model channels, subjects, or message schemas for event-driven interfaces. What specification format should describe the ComplyTime API's event-driven contract?

[ADR-0021](0021-cloudevents-envelope.md) established CloudEvents as the ingestion event envelope. CloudEvents defines what a single event looks like — the required envelope fields (`id`, `source`, `type`, `specversion`, `data`). It does not describe which NATS subjects exist, what `type` values the ComplyTime API emits, what the `data` payload schema is, or what version of the API a consumer is targeting.

## Decision Drivers

* Published contract — consumers can integrate from the spec alone, without reading ComplyTime implementation code.
* Models channels, subjects, and message schemas — not just HTTP verbs and paths.
* Generates documentation and client bindings.
* License compatible with Apache 2.0.

## Considered Options

* AsyncAPI 3.0
* OpenAPI
* Prose documentation only

## Decision Outcome

Chosen option: "AsyncAPI 3.0", because it is the standard specification format for event-driven APIs, models NATS subjects and CloudEvents messages natively, and generates documentation that gives consumers a stable, machine-readable integration contract.

### Confirmation

An AsyncAPI 3.0 document describing the ComplyTime API ingestion channel is published alongside the API. A consumer uses the document to configure a NATS subscription and parse ingestion events without reading ComplyTime source code.

## Consequences

* Good, because AsyncAPI models NATS channels and CloudEvents message schemas natively — the two specs compose without workarounds.
* Good, because the AsyncAPI Generator produces HTML documentation and client bindings in multiple languages from the spec. Template maturity varies by language; the Go template is community-maintained and should be evaluated before committing to generated client output.
* Good, because a machine-readable spec could enable contract testing between the ComplyTime API and its consumers — realizing this requires a contract testing tool (e.g., Microcks) and CI enforcement; without both, the benefit is latent.
* Good, because AsyncAPI specification is Apache 2.0 licensed.
* Bad, because maintaining the AsyncAPI document in sync with the implementation requires discipline — drift between spec and code is a real risk without CI enforcement.
* Bad, because AsyncAPI 3.0 introduced breaking changes from 2.x; consumers and tooling must target the same major version.

## Pros and Cons of the Options

### AsyncAPI 3.0

The leading open specification for event-driven APIs. Models channels, operations, and message schemas. NATS and CloudEvents bindings are supported. Licensed Apache 2.0.

* Good, because purpose-built for event-driven interfaces — channels and messages are first-class concepts.
* Good, because tooling ecosystem (AsyncAPI Generator, Studio, CLI) supports documentation, validation, and code generation.
* Bad, because spec maintenance must be kept in sync with implementation — tooling helps but does not eliminate the risk.
* Bad, because the AsyncAPI NATS binding has not reached full parity with AsyncAPI 3.0; the binding version must be pinned and verified against 3.0 tooling before publishing the spec. AsyncAPI 2.x has better tooling parity today and should be considered if 3.0 binding gaps block adoption.

### OpenAPI

The standard specification for HTTP/REST APIs. Widely adopted with mature tooling.

* Good, because familiar to most developers and broadly supported.
* Bad, because OpenAPI models request/response over HTTP — it has no concept of pub/sub channels, message brokers, or event subjects.
* Bad, because using OpenAPI for an event-driven interface requires workarounds that produce an inaccurate or misleading contract.

### Prose documentation only

Event subjects, message formats, and subscription patterns described in Markdown.

* Good, because no tooling overhead — the lowest friction path to publishing a consumer guide.
* Good, because prose is readable without any tooling knowledge.
* Bad, because prose cannot be validated or used to generate client bindings.
* Bad, because consumers have no machine-readable way to detect breaking changes — drift is invisible until integration breaks.
* Bad, because as the API evolves, prose docs tend to fall behind implementation faster than a spec that tooling can lint.

## More Information

AsyncAPI fills the gap left by CloudEvents: it describes channel topology and message contracts, and can reference CloudEvents as the envelope format for messages on a given channel.

* [ADR-0019: Event-Driven Ingestion Pattern](0019-event-driven-ingestion.md)
* [ADR-0020: NATS JetStream as Event Bus](0020-nats-jetstream-event-bus.md)
* [ADR-0021: CloudEvents for Ingestion Event Envelope](0021-cloudevents-envelope.md)
* [AsyncAPI specification](https://www.asyncapi.com)
* [AsyncAPI GitHub (Apache 2.0)](https://github.com/asyncapi/spec)
* [AsyncAPI NATS binding](https://github.com/asyncapi/bindings/tree/master/nats)
