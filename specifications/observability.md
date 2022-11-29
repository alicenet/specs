# Observability

## Introduction

#### Summary

Observability is the foundation for being able to operate a production service reliably. Given the nature
of work that the AliceNet network does, reliability is a paramount concern. Therefore significant work
should be done to ensure that operators of nodes are able to have visibility into what their nodes are doing
and how to diagnose and correct any issues with them.

Observability generally falls into 3 categories:

- Centralized, structured logging
- Continuous metric export
- Distributed tracing support

#### Context

There are a number of supported libraries and platforms in this space compatible with our codebase:

- [OpenCensus](https://opencensus.io) metrics and tracing reporting
- Future state: [OpenTelemetry](https://opentelemetry.io) for metrics, tracing and logging
- [Google Cloud Structured Logging](https://cloud.google.com/logging/docs/structured-logging)
- [Google Cloud Custom Metrics](https://cloud.google.com/monitoring/custom-metrics)

#### Goals

- Support metrics reporting for nodes
- Support centralized, structured logging for nodes
- Support Google Cloud Platform as first class ecosystem

#### Non Goals

- Support for tracing for nodes
- Support local collection and review (e.g. Prometheus)
- Support for AWS, Azure, other cloud platforms as a follow on

#### Assumptions

- Nodes currently are running only on Google Cloud Platform
- Nodes are written only in Go

## Specification

### Overview

#### Centralized, Structured Logging

Currently the project has a somewhat custom logging framework set up. While this is a
sufficient mechanism for logging and debugging locally, it unfortunately does not work
well for running a production service. One of the design decisions made early on was to
support logigng levels per sub-system, controlled via flags to make debugging easier.
This approach was similar to [slf4j](https://www.slf4j.org). In our cloud based
environment there is no need to do this, as logs can be searched and filtered with ease.
As part of this work we should remove log levels and instead rely on the centralized
logging facility's capabilities. This has the benefit of cleaning up a bunch of the
code around node initialization.

Logs are currently handled only at the OS level. Interacting with them is via SSH'ing
into the VMs hosting the nodes and utilizing [journalctl](https://www.freedesktop.org/software/systemd/man/journalctl.html)
to review them. This mechanism requires too many permissions for most users to safely
utilize, and we should instead use a centralized location. For now that will be in
Google Cloud. All node VMs should install a [cloud logging agent](https://cloud.google.com/logging/docs/agent/logging/installation)
to steam logs to the platform. This way there is one place to look, and as a bonus allows
for log based metrics and alerting to be used.

Google Cloud Operations Suite's Logging supports rich, structured data. The project
currently uses that somewhat, but since logs aren't being piped to cloud logging, the
benefits are not there. There are a few extra well known fields that should be included,
such as file, package and line number that will allow a lot more correlation. In the
future if distributed tracing is used that can also be included as a field allowing
all logs for a given trace to be seen in one view.

The structured logging entry is currently passed around methods as a parameter. To
simplify the method signatures we should move to having log entries populated in a
`context.Context` which gets passed to all methods.

#### Metrics Export

Currently no metrics are exported from an AliceNet node. With metrics being exported to
[GCP Operations Suite](https://cloud.google.com/products/operations) there will be the
opportunity to monitor the overall state of the system under our control as well as alert
on liveness indicators.

The following metrics should be exported as part of our code base:

- Peering
  - Number of connected peers, by type
  - Number of exchanged messages
- Dynamics
  - The value of each variable in dynamics
- Blocks
  - Current epoch
  - Number of blocks
  - Number of transactions
- Bridge
  - Number of events seen by type

In addition to those metrics, the following metrics should be set up to have alerting:

- Disk usage
- CPU usage

#### Security / Risks

- It is possible to accidentally leak secrets via logs
  - Partially mitigated by access to logging being restricted

## Further Considerations

#### Alternative Solutions

Current alternative is to leave observability as is, with semi-structured logging only.
