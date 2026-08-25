# Security Lists and Network Security Groups

## Overview

This document describes the Security List (SL) and Network Security Group (NSG) configuration illustrated for the [One-OE + Hub A deployment](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/blueprints/one-oe/runtime/one-stack/one_oe_hub_a.md). It is intended as an implementation and review aid for the referenced landing-zone template; the deployed rules must be validated against the template revision and the requirements of each workload.

Security Lists and NSGs provide layered, allow-list controls for VNIC traffic. Security Lists apply to every VNIC in an associated subnet, whereas NSGs apply only to the VNICs or supported resources assigned to the NSG. If both are used, the effective allowed traffic is the union of the rules in the subnet's Security Lists and all NSGs attached to the VNIC. Therefore, a flow is blocked only when no applicable rule allows it.

The rules support the deployment models shown while following least-privilege principles. Broad rules shown in the diagram (for example, `0.0.0.0/0` or `ALL` egress) must be reviewed and justified for the resource's role, such as a load balancer or an inspection path, before deployment to a production environment.

For general OCI behavior, see [Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm) and [Stateful Compared to Stateless Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm#stateful).

## Stateless rules and return traffic

Within the Hub VCN, the NSGs associated with OCI Network Firewall and load balancers use stateless rules where shown in the template. OCI recommends that security controls associated with a Network Firewall do not contain stateful rules for better firewall performance. For load balancers and other high-throughput resources, stateless rules can also be appropriate when their bidirectional flows are explicitly designed and tested.

Stateless rules do not track connections. Every required return path must therefore be allowed explicitly. For example, when a load balancer sends an HTTP health check to a backend, the request normally uses an ephemeral source port and destination port `80`. The response sent by the backend has source port `80` and the load balancer's ephemeral destination port. A stateless ingress rule on the load balancer side can therefore require:

| Direction | Protocol | Source | Source port | Destination port | Purpose |
| --- | --- | --- | --- | --- | --- |
| Ingress | TCP | Backend subnet or NSG | `80` | `1024-65535` | Backend response to a load balancer HTTP health check |

Use the actual ephemeral-port range required by the operating system and service in use. The example above illustrates the response path only; it must be paired with the corresponding request rule and applied to the correct endpoint.

If a packet matches both stateful and stateless rules in the same direction, OCI gives the stateless rule precedence and does not track the connection. A corresponding rule in the reverse direction is then required for response traffic.

> [!IMPORTANT]
> SL and NSG rules do not establish end-to-end connectivity on their own. Confirm the forward and return routes, the Network Firewall policy, and any other applicable controls before treating a flow as permitted. For inspected paths, maintain symmetric routing through the firewall.

## Diagram

The following diagram illustrates the SL and NSG rules for the referenced One-OE + Hub A deployment.

<img src="./sl_nsg.png" alt="Security List and Network Security Group rules for the One-OE plus Hub A deployment" width="980">

## Rule review checklist

Use this checklist when reviewing or adapting the rules:

1. Confirm the source, destination, protocol, and port for each intended flow.
2. Identify every Security List and NSG that applies to each endpoint; an allow rule in either control can permit the flow.
3. For every stateless rule, confirm an explicit reverse-direction rule for the return flow.
4. Restrict broad source, destination, and `ALL`-protocol rules where the resource role permits it; record the rationale where they must remain.
5. Verify that subnet and DRG routing supports both the forward and return paths.
6. Verify that the OCI Network Firewall policy allows the intended inspected traffic.
7. Test health checks, application traffic, management access, and failure scenarios after deployment.

## Scope and considerations

- The diagram does not include dedicated NSG configurations for the management, monitoring, and DNS subnets.
- The generic Security List `sl-fra-lz-hub-mgmt`, which applies to those subnets, is not depicted in the diagram and must be reviewed separately.
- East-west communication between environments is intended to be restricted at the spoke level and controlled through the Network Firewall in the Hub. Validate that no additional applicable SL or NSG rule permits a bypass path.
- This document describes network access controls only. It does not replace validation of route tables, DRG route tables, Network Firewall policy, operating-system firewalls, or application-level authorization.

## References

- [OCI Security Rules](https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/securityrules.htm)
- [OCI Network Firewall: Create a Firewall](https://docs.oracle.com/en-us/iaas/Content/network-firewall/firewall-create.htm)
- [One-OE + Hub A deployment](https://github.com/oci-landing-zones/oci-landing-zone-operating-entities/blob/master/blueprints/one-oe/runtime/one-stack/one_oe_hub_a.md)

# License

Copyright (c) 2026 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](/LICENSE.txt) for more details.
