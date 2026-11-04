

# Networking

## 1. Purpose

This document explains the main networking responsibilities in the self-hosted environment.

The public version focuses on architecture and service roles. It deliberately omits private IP addresses, internal hostnames, detailed firewall exceptions, exact service ports and private remote-access configuration.

## 2. Network architecture

The environment uses a UniFi gateway for local network services and routing, with Caddy providing application-level reverse proxying.

These are separate responsibilities.

| Component | Responsibility |
|---|---|
| UniFi gateway | Local DNS records and LAN routing |
| Caddy | HTTP/HTTPS reverse proxy and application routing |
| Docker networking | Communication between containerised services |
| Twingate | Remote access to services across the home network |
| UniFi Teleport | Independent remote VPN option |

## 3. DNS and LAN routing

The UniFi gateway handles local DNS records and LAN routing.

This provides local name resolution and network-level communication between devices and services.

DNS resolution does not itself provide application-level HTTP or HTTPS routing.

## 4. Reverse proxying

Caddy handles application-level reverse proxying.

It receives HTTP/HTTPS requests and routes them to configured application upstreams.

This provides a consistent application access layer and separates application routing from gateway DNS responsibilities.

A configured Caddy route is not proof that the upstream is available. Individual upstream connectivity should be tested from the relevant network context.

## 5. Docker networking

The container environment uses multiple Docker networks to organise application and supporting services.

The public documentation intentionally describes service relationships at a logical level rather than publishing exact network names or every container membership.

Some services communicate within application groups, while others rely on host-level connectivity or separate network arrangements.

A diagram of the simplified service relationships is available in [Docker network topology](../diagrams/docker-network-topology.mmd).

## 6. DNS services

Pi-hole provides DNS filtering.

Unbound is also present as a separate DNS service.

In the configuration reviewed, Pi-hole used public DNS resolvers as its upstream services. Unbound was not configured as Pi-hole's upstream in that observed arrangement.

The presence of both services should not be interpreted as proof that DNS traffic passes through both in sequence.

## 7. Remote access

### Twingate

Twingate is configured to provide access to services across the home network.

It is not documented as being restricted to services exposed through Caddy.

The public portfolio does not disclose connector credentials, resource identifiers or detailed access policy configuration.

### UniFi Teleport

UniFi Teleport, accessed through WiFiman, provides a separate remote VPN option.

It is independent of the Twingate setup.

The two remote-access methods are alternatives and do not need to be used together.

## 8. Host port exposure

Some container services were observed publishing ports on all host interfaces.

This is an important configuration detail, but it does not prove that a service is accessible from the public internet.

Actual reachability depends on multiple layers, including:

- Host firewall rules.
- Gateway and routing configuration.
- Network segmentation.
- Any upstream filtering.
- The service's own access controls.

The public portfolio therefore does not publish detailed host-port mappings or claim that a particular service is publicly exposed without direct evidence.

## 9. Validation boundaries

The networking review distinguishes between:

- A configured DNS record.
- A configured proxy route.
- A container's network membership.
- A successful connection test.
- Verified access from a particular network location.

These are different levels of evidence.

For example, the presence of a Caddy upstream in configuration does not establish that the application is healthy or reachable from every client network.

## 10. Security and privacy

The public repository intentionally excludes:

- Private IPv4 addressing.
- Internal DNS zones and hostnames.
- Wi-Fi identifiers and credentials.
- MAC addresses.
- Exact firewall exceptions.
- Detailed remote-access resources.
- Connector tokens and private certificates.
- Exact host-port mappings where they are not necessary to explain the design.

The aim is to demonstrate understanding of network responsibilities and architecture without publishing an unnecessary map of the home network.

