# GossipSub Metrics Specification

> Standardized optional metrics for GossipSub implementations to enable consistent observability and performance monitoring

| Lifecycle Stage | Maturity      | Status | Latest Revision |
|-----------------|---------------|--------|-----------------|
| 1A              | Working Draft | Active | r0, 2025-01-25  |

Authors: [@dennis-tra]

Interest Group: TBD

[@dennis-tra]: https://github.com/dennis-tra

See the [lifecycle document][lifecycle-spec] for context about the maturity level and spec status.

[lifecycle-spec]: https://github.com/libp2p/specs/blob/master/00-framework-01-spec-lifecycle.md

## Motivation

GossipSub implementations across different programming languages currently expose varying sets of metrics for observability and performance monitoring. This inconsistency makes it challenging for:

**Node Operators** to:
- Deploy unified monitoring dashboards across heterogeneous GossipSub deployments
- Compare performance characteristics between different implementations
- Diagnose network health issues using standardized indicators
- Create portable alerting rules and runbooks

**Researchers** to:
- Conduct comparative studies across different GossipSub implementations
- Analyze protocol behavior patterns using consistent metrics across diverse network conditions
- Validate theoretical models against empirical data from multiple implementations
- Share reproducible experimental results using standardized measurement frameworks
- Develop benchmarking methodologies for peer-to-peer messaging protocols

This specification defines a standardized set of **optional Prometheus-style metrics** that GossipSub implementations MAY support to enable consistent observability. These metrics are designed to be:

- **Opt-in**: Implementations can continue using their existing metrics without breaking changes
- **Diagnostic-focused**: Help operators identify when GossipSub is experiencing issues or performing suboptimally
- **Cross-implementation compatible**: Enable meaningful performance comparisons across Go, Rust, JavaScript, and other implementations
- **Research-enabling**: Provide consistent measurement capabilities for academic and industrial research studies

## Scope and Goals

### In Scope
- Definition of standardized metric names, types, and labels
- Semantic specifications for when metrics should be updated
- Export format recommendations for interoperability
- Implementation guidelines for performance-conscious metric collection

### Out of Scope
- Mandating specific metrics collection frameworks or libraries
- Requiring implementations to abandon existing metrics
- Defining wire protocol changes or new GossipSub messages
- Specifying monitoring system deployment or configuration

### Goals
- **Unified Observability**: Enable operators to monitor GossipSub performance consistently across implementations
- **Improved Diagnostics**: Provide metrics that help identify common network health and performance issues
- **Research Facilitation**: Enable reproducible, comparable research studies across different GossipSub implementations and network conditions
- **Implementation Flexibility**: Allow implementations to adopt metrics selectively based on their needs
- **Performance Awareness**: Ensure metric collection doesn't significantly impact GossipSub performance

## Metric Categories

This specification organizes metrics into four primary categories:

### 1. Peer Management Metrics
Metrics related to peer relationships, mesh topology, and peer lifecycle events.

### 2. Message Flow Metrics  
Metrics tracking message processing, validation, delivery, and publishing operations.

### 3. Protocol Control Metrics
Metrics for RPC messages, gossip control operations, and protocol-specific events.

### 4. Performance & Health Metrics
Metrics indicating system performance, resource utilization, and network health indicators.

## Metric Specifications

### Naming Conventions

All standardized metric names MUST follow these conventions:
- Use `gossipsub_` prefix for all metrics
- Use snake_case for metric names and label names
- Use descriptive names that clearly indicate what is being measured
- Include units in metric names where applicable (e.g., `_duration_seconds`, `_bytes_total`)

### Metric Types

This specification defines Prometheus-style metrics using three standard types:

- **Counter**: Monotonically increasing values (e.g., message counts, error counts)
- **Gauge**: Values that can increase or decrease (e.g., peer counts, mesh size)
- **Histogram**: Distribution of values with configurable buckets (e.g., latency, scores)

### Standard Labels

The following labels MAY be applied to metrics where semantically appropriate:

- `topic`: The pubsub topic name (when metric is topic-specific)
- `peer_id`: Peer identifier (when metric is peer-specific, use judiciously for cardinality)
- `reason`: Categorization of why an event occurred (e.g., rejection reason, penalty type)
- `message_type`: Type of RPC or control message
- `validation_result`: Result of message validation (accept, reject, ignore)

## Metric Definitions

All metrics follow Prometheus naming conventions and use the `gossipsub_` prefix. The following table defines the complete set of standardized metrics:

| Metric Name | Type | Labels | Description |
|-------------|------|--------|--------------|
| **Peer Management** |
| `gossipsub_peers_total` | Gauge | `topic` (optional) | Current number of known peers, optionally segmented by topic |
| `gossipsub_mesh_peers_total` | Gauge | `topic` (required) | Current number of peers in the mesh for each topic |
| `gossipsub_peer_graft_total` | Counter | `topic` (required) | Total number of GRAFT messages sent, by topic |
| `gossipsub_peer_prune_total` | Counter | `topic` (required), `reason` (optional) | Total number of PRUNE messages sent, by topic and optional reason |
| `gossipsub_peer_score` | Histogram | `topic` (optional) | Distribution of peer scores |
| **Message Flow** |
| `gossipsub_message_received_total` | Counter | `topic` (required), `validation_result` (optional) | Total messages received for processing, optionally by validation result |
| `gossipsub_message_delivered_total` | Counter | `topic` (required) | Total messages successfully delivered to local subscribers |
| `gossipsub_message_rejected_total` | Counter | `topic` (required), `reason` (optional) | Total messages rejected during validation, optionally by reason |
| `gossipsub_message_duplicate_total` | Counter | `topic` (required) | Total duplicate messages detected and discarded |
| `gossipsub_message_published_total` | Counter | `topic` (required) | Total messages published by local node |
| `gossipsub_message_latency_seconds` | Histogram | `topic` (optional) | End-to-end message delivery latency in seconds |
| **Protocol Control** |
| `gossipsub_rpc_received_total` | Counter | `message_type` (required) | Total RPC messages received by type (publish, subscribe, unsubscribe, graft, prune, ihave, iwant, idontwant) |
| `gossipsub_rpc_sent_total` | Counter | `message_type` (required) | Total RPC messages sent by type |
| `gossipsub_ihave_sent_total` | Counter | `topic` (required) | Total IHAVE control messages sent per topic |
| `gossipsub_iwant_sent_total` | Counter | `topic` (required) | Total IWANT control messages sent per topic |
| `gossipsub_idontwant_sent_total` | Counter | `topic` (required) | Total IDONTWANT control messages sent per topic |
| **Performance & Health** |
| `gossipsub_heartbeat_duration_seconds` | Histogram | None | Time spent processing each heartbeat operation |
| `gossipsub_peer_throttled_total` | Counter | `reason` (optional) | Total number of times peers have been throttled |
| `gossipsub_backoff_violations_total` | Counter | None | Total attempts to reconnect before backoff period completion |
| `gossipsub_score_penalty_total` | Counter | `penalty_type` (required), `topic` (optional) | Total peer scoring penalties applied by type |

### Metric Update Semantics

**Counters** are incremented when:
- `*_total` metrics: Each time the corresponding event occurs (message sent/received, peer action, etc.)
- Events are counted at the protocol level, not application level

**Gauges** are updated when:
- `*_peers_total`: Peers are added/removed from peer tracking or topic meshes
- Values reflect current state at time of observation

**Histograms** are updated when:
- `gossipsub_peer_score`: During peer scoring operations (recommended buckets: `[-100, -10, -1, 0, 1, 10, 100, +Inf]`)
- `*_latency_seconds`: When latency measurements are available (recommended buckets: `[0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0, 5.0, +Inf]`)
- `*_duration_seconds`: When timing operations complete (recommended buckets: `[0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1.0, +Inf]`)

## Implementation Guidelines

### Metric Collection Performance

Implementations SHOULD:
- Use efficient metric collection mechanisms that minimize impact on message processing latency
- Implement metric updates asynchronously where possible
- Provide configuration options to disable metric collection entirely
- Consider metric cardinality implications, especially for topic-specific metrics in high-topic-count environments

### Configuration Options

Implementations SHOULD provide configuration to:
- Enable/disable entire metric categories
- Configure histogram bucket boundaries based on expected value distributions  
- Set maximum cardinality limits for high-cardinality labels like `topic`
- Control metric export formats and destinations

### Backward Compatibility

Implementations adopting this specification:
- MUST NOT remove or modify existing metrics without appropriate deprecation periods
- MAY implement these metrics alongside existing metrics systems
- SHOULD clearly document which standardized metrics are supported

## Prometheus Export Format

Implementations MUST export metrics in Prometheus format following these conventions:

- Use standard Prometheus metric types (counter, gauge, histogram)
- Include `_total` suffix for counters following Prometheus conventions
- Use `_seconds` suffix for time-based metrics
- Provide help text describing each metric's purpose
- Follow Prometheus naming best practices for metric and label names

### Example Prometheus Output

```
# HELP gossipsub_mesh_peers_total Current number of peers in the mesh for each topic
# TYPE gossipsub_mesh_peers_total gauge
gossipsub_mesh_peers_total{topic="ipfs-dht"} 8
gossipsub_mesh_peers_total{topic="libp2p-announce"} 12

# HELP gossipsub_message_received_total Total messages received for processing
# TYPE gossipsub_message_received_total counter
gossipsub_message_received_total{topic="ipfs-dht",validation_result="accept"} 1543
gossipsub_message_received_total{topic="ipfs-dht",validation_result="reject"} 23

# HELP gossipsub_heartbeat_duration_seconds Time spent processing each heartbeat operation
# TYPE gossipsub_heartbeat_duration_seconds histogram
gossipsub_heartbeat_duration_seconds_bucket{le="0.001"} 45
gossipsub_heartbeat_duration_seconds_bucket{le="0.005"} 123
gossipsub_heartbeat_duration_seconds_bucket{le="+Inf"} 150
gossipsub_heartbeat_duration_seconds_sum 0.456
gossipsub_heartbeat_duration_seconds_count 150
```

## Security Considerations

When implementing these metrics, consider:

- **Information Disclosure**: Topic names in metrics may reveal sensitive information about network usage patterns
- **Cardinality Attacks**: Malicious peers could potentially cause high cardinality by creating many topics or using diverse peer IDs
- **Resource Consumption**: Metric collection itself consumes memory and CPU resources that should be bounded

Implementations SHOULD provide mechanisms to:
- Hash or obfuscate sensitive label values
- Limit the number of unique label combinations
- Monitor and alert on metric collection resource usage

## Example Usage

### Grafana Dashboard Query Examples

```promql
# Current mesh size across all topics  
sum by (topic) (gossipsub_mesh_peers_total)

# Message delivery success rate
rate(gossipsub_message_delivered_total[5m]) / rate(gossipsub_message_received_total[5m])

# 95th percentile heartbeat duration
histogram_quantile(0.95, rate(gossipsub_heartbeat_duration_seconds_bucket[5m]))

# Peer churn rate (grafts + prunes)
rate(gossipsub_peer_graft_total[5m]) + rate(gossipsub_peer_prune_total[5m])
```

### Common Alerting Rules

```yaml
# High message rejection rate
- alert: GossipSubHighRejectionRate
  expr: rate(gossipsub_message_rejected_total[5m]) / rate(gossipsub_message_received_total[5m]) > 0.1
  
# Mesh size below optimal
- alert: GossipSubLowMeshSize  
  expr: gossipsub_mesh_peers_total < 6

# Slow heartbeat processing
- alert: GossipSubSlowHeartbeat
  expr: histogram_quantile(0.95, rate(gossipsub_heartbeat_duration_seconds_bucket[5m])) > 1.0
```

## Checklist for Candidate Recommendation

To promote this specification from Working Draft to Candidate Recommendation:

- [ ] At least one complete reference implementation
- [ ] Community feedback from multiple implementation teams
- [ ] Validation that metrics provide actionable diagnostic information
- [ ] Performance impact analysis showing acceptable overhead
- [ ] Documentation of metric correlation patterns for common issues
- [ ] Interoperability testing between implementations using these metrics
- [ ] Finalized Interest Group with representatives from major implementations

## References

- [GossipSub v1.1 Specification](../pubsub/gossipsub/gossipsub-v1.1.md)
- [go-libp2p-pubsub Issue #534](https://github.com/libp2p/go-libp2p-pubsub/issues/534) - Initial metrics standardization discussion
- [Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)
- [OpenTelemetry Metrics API](https://opentelemetry.io/docs/reference/specification/metrics/api/)