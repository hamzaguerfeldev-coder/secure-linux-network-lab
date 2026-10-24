# Secure Linux Network & Infrastructure Diagnostics Lab

A hands-on infrastructure project focused on Linux administration, network engineering, firewall security, Bash automation, and troubleshooting.

The goal is to design, build, secure, test, and document a small enterprise-style network using Linux virtual machines.

## Project Objectives

- Design a segmented network with a LAN, DMZ, and WAN interface.
- Configure static IP addressing, routing, and Network Address Translation (NAT).
- Implement DHCP and DNS services.
- Deploy a Linux web server in the DMZ.
- Configure firewall rules using `nftables` and apply least-privilege access policies.
- Secure administrative access using SSH keys.
- Develop Bash scripts for health checks, system auditing, and configuration backups.
- Validate the infrastructure through repeatable tests and controlled failure scenarios.
- Apply professional Git and GitHub workflows.

## Architecture

The lab consists of three Ubuntu Server virtual machines managed with VirtualBox.

```text
                       Internet
                          |
                    VirtualBox NAT
                          |
                  +----------------+
                  |    Gateway     |
                  | Routing / NAT  |
                  | nftables       |
                  | DNS / DHCP     |
                  +----------------+
                     /          \
                    /            \
          LAN 10.10.1.0/24    DMZ 10.10.2.0/24
                  |                  |
            +-----------+      +-------------+
            |   Client  |      | Web Server  |
            | Linux VM  |      | nginx / SSH |
            +-----------+      +-------------+
```

### Virtual Machines

| Machine | Role |
|---|---|
| `gateway` | Routing, NAT, firewall, DHCP, DNS, and network logging |
| `client` | Connectivity tests, DNS validation, and security testing |
| `web-server` | nginx web server and restricted SSH administration |

### Network Addressing Plan

| Component | Planned address |
|---|---|
| Gateway LAN interface | `10.10.1.1/24` |
| Client | `10.10.1.10/24` |
| Gateway DMZ interface | `10.10.2.1/24` |
| Web server | `10.10.2.10/24` |
| Gateway WAN interface | Assigned through VirtualBox NAT |

These addresses are the initial design and will be validated during implementation.

## Security Policy

The intended firewall policy is based on least privilege:

- Allow explicitly approved traffic from the LAN to the DMZ.
- Restrict SSH access to an authorized administration source.
- Block new connections initiated from the DMZ toward the LAN.
- Restrict outbound DMZ traffic to required services.
- Block unsolicited inbound connections from the WAN.
- Allow return traffic for authorized established connections.
- Log blocked traffic with appropriate rate limiting.

The final rules will be tested and documented against the actual behavior of the lab.

## Bash Automation

The project will include three main scripts.

### `healthcheck.sh`

Checks expected connectivity, DNS resolution, and web service availability. Reports `PASS` or `FAIL` and returns an appropriate exit code.

### `audit.sh`

Collects listening ports, active services, and relevant SSH authentication failures. Generates a timestamped audit report.

### `backup.sh`

Backs up selected infrastructure configuration files, validates the archive, and tests restoration in a temporary location without overwriting active configuration.

Automated execution will be configured only after the scripts have been tested manually.

## Validation and Evidence

The project will include evidence of actual test results, including:

- Expected versus observed connectivity.
- Successful and blocked network connections.
- DNS and DHCP validation.
- Firewall logging and rule verification.
- Before-and-after network scans using `nmap`.
- Packet captures demonstrating NAT behavior using `tcpdump`.
- Backup integrity and restoration tests.
- Documented troubleshooting scenarios and corrective actions.

Only tests that have actually been executed will be reported as completed.

## Planned Repository Structure

```text
secure-linux-network-lab/
├── README.md
├── config/
│   ├── gateway/
│   ├── client/
│   └── web-server/
├── scripts/
├── tests/
├── docs/
├── diagrams/
├── evidence/
├── .gitignore
└── LICENSE
```

The structure will evolve as implementation progresses.

## Implementation Roadmap

1. Prepare the VirtualBox environment and network interfaces.
2. Configure IP addressing and validate basic connectivity.
3. Implement routing and NAT on the gateway.
4. Configure DHCP and DNS.
5. Deploy nginx and secure SSH access.
6. Implement and validate firewall policies.
7. Develop and test the Bash automation scripts.
8. Run connectivity, security, backup, and restoration tests.
9. Document the architecture, test evidence, and troubleshooting results.
10. Finalize the documentation and publish a validated `v1.0.0` release.

## Technology Stack

- Ubuntu Server LTS
- VirtualBox
- Linux networking and Netplan
- Bash
- `nftables`
- `dnsmasq`
- nginx
- OpenSSH
- `systemd` timers and cron
- `tcpdump` and `nmap`
- Git and GitHub

## Project Status

**Status:** Planning and environment preparation.

The infrastructure, scripts, and security controls will be implemented and validated incrementally. The README will be updated to reflect the actual state of the project.

## Learning Approach

**Build → Test → Break → Troubleshoot → Fix → Document → Repeat**

This project is intended as a practical learning and portfolio exercise demonstrating Linux administration, networking fundamentals, automation, security practices, and disciplined engineering workflows.

## License

This project is distributed under the MIT License. See the `LICENSE` file for details.
