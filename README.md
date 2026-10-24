# **Secure Linux Network & Infrastructure Diagnostics Lab**  
**A practical infrastructure engineering project demonstrating Linux administration, network segmentation, firewall security, Bash automation, troubleshooting, and Git-based workflows.**  
**Project status:** Planning and environment preparation  
 **Environment:** VirtualBox · Ubuntu Server LTS · Three Linux virtual machines  
 **Focus:** Infrastructure fundamentals, security controls, operational reliability, and verifiable technical evidence  
## **1. Overview**  
This project simulates a small enterprise network built from three Linux virtual machines.  
The infrastructure is designed around a dedicated gateway, an internal client, and a web server isolated in a Demilitarized Zone (DMZ).  
The goal is not simply to configure a working network. The goal is to demonstrate an engineering workflow:  
**Design → Build → Validate → Break safely → Troubleshoot → Fix → Document**  
Every major feature must have a clear purpose, a reproducible test, an expected result, an observed result, and supporting evidence.  
The project will be developed incrementally. Features and test results will only be marked as completed after they have been implemented and verified.  
## **2. Project Objectives**  
### **Linux administration**  
- Install and configure minimal Ubuntu Server virtual machines.  
- Manage users, permissions, services, processes, and system logs.  
- Configure static network interfaces using Netplan.  
- Secure remote administration with SSH keys.  
- Investigate system and service failures using standard Linux tools.  
### **Networking**  
- Design and document an IPv4 addressing plan using CIDR notation.  
- Understand network interfaces, routing tables, and default gateways.  
- Configure IPv4 forwarding and Network Address Translation (NAT).  
- Deploy DHCP and DNS services using dnsmasq.  
- Validate connectivity at the IP, DNS, transport, and HTTP layers.  
- Inspect network traffic with tcpdump.  
### **Infrastructure security**  
- Separate the internal LAN from the web-server DMZ.  
- Implement stateful firewall rules with nftables.  
- Apply a default-deny policy to forwarded traffic.  
- Restrict administrative access and unnecessary outbound connections.  
- Log blocked traffic with rate limiting.  
- Verify security rules through positive and negative tests.  
### **Bash automation and operations**  
- Develop reusable scripts with functions, argument validation, and meaningful exit codes.  
- Automate service and connectivity health checks.  
- Generate system audit reports.  
- Back up configuration files and verify restoration.  
- Schedule selected tasks using systemd timers and cron.  
- Test normal operation, expected failures, and invalid inputs.  
### **Engineering workflow**  
- Maintain a clear Git history with meaningful commits.  
- Use feature branches and pull requests.  
- Document implementation decisions and troubleshooting.  
- Store sanitized test reports and relevant screenshots.  
- Establish measurable acceptance criteria for the final release.  
## **3. Architecture**  
The lab consists of three virtual machines connected through isolated VirtualBox networks.  
                         Internet  
                            |  
                    VirtualBox NAT  
                            |  
                  +-------------------+  
                  |      GATEWAY      |  
                  |                   |  
                  | Routing / NAT     |  
                  | nftables          |  
                  | DHCP / DNS        |  
                  | Network logging   |  
                  +-------------------+  
                     |             |  
              Internal LAN    Internal DMZ  
              10.10.1.0/24    10.10.2.0/24  
                     |             |  
              +-------------+  +-------------+  
              |   CLIENT    |  | WEB SERVER  |  
              |             |  |             |  
              | Test tools  |  | nginx       |  
              | DNS checks  |  | SSH         |  
              +-------------+  +-------------+  
### **Virtual machines**  
  
|  |  |  
| - | - |  
| **Machine** |  **Primary responsibilities** |  
| gateway |  Routing, NAT, firewall enforcement, DHCP, DNS forwarding, and network logs |  
| client |  Connectivity checks, DNS queries, HTTP tests, and authorized security validation |  
| web-server |  nginx hosting, restricted SSH administration, and service-level diagnostics |  
  
### **VirtualBox network design**  
- **WAN:** VirtualBox NAT network interface on the gateway.  
- **LAN:** Internal Network shared by the gateway and client.  
- **DMZ:** Separate Internal Network shared by the gateway and web server.  
The gateway will have three network interfaces. The client and web server will each have only the interface required for their respective zone.  
The gateway must be the only path between the LAN, DMZ, and WAN. The web server must not have a second network interface that bypasses the firewall.  
## **4. IP Addressing Plan**  
The following is the initial design. Actual interface names and connectivity will be verified during implementation.  
  
|  |  |  
| - | - |  
| **Component** |  **Planned configuration** |  
| Gateway WAN |  Address assigned through VirtualBox NAT |  
| Gateway LAN |  10.10.1.1/24 |  
| Client LAN |  DHCP reservation: 10.10.1.10/24 |  
| Gateway DMZ |  10.10.2.1/24 |  
| Web server DMZ |  10.10.2.10/24 |  
| LAN default gateway |  10.10.1.1 |  
| DMZ default gateway |  10.10.2.1 |  
| Internal web hostname |  web.lab.test |  
| Internal DNS resolver |  10.10.1.1 for the LAN client |  
  
The 10.10.1.0/24 and 10.10.2.0/24 networks must remain separate routed networks.  
The gateway will provide a DHCP reservation for the client and DNS resolution for the internal web hostname. The web server will use a static IP address.  
The .test suffix is used for testing and avoids relying on a public DNS domain.  
## **5. Network and Security Policy**  
The firewall policy will follow the principle of least privilege.  
  
|  |  |  |  
| - | - | - |  
| **Source** |  **Destination** |  **Intended policy** |  
| LAN client |  Gateway DHCP and DNS |  Allow required services |  
| LAN client |  Web server |  Allow HTTP and HTTPS as configured |  
| Authorized LAN administrator |  Web server SSH |  Allow TCP/22 |  
| Other LAN hosts |  Web server SSH |  Deny unless explicitly authorized |  
| DMZ |  LAN |  Block new connections |  
| DMZ |  WAN |  Permit only required outbound DNS, HTTP, and HTTPS traffic |  
| WAN |  LAN and DMZ |  Block unsolicited inbound connections |  
| Established authorized connections |  Return traffic |  Permit tracked responses |  
| Other forwarded traffic |  Any |  Deny and log with rate limiting |  
  
The exact rules will account for connection state, DNS forwarding, DHCP traffic, and required return packets.  
Firewall policy enforcement will be implemented using nftables. The gateway will not simultaneously use UFW as a second firewall manager.  
The web server will use SSH key authentication. Password-based SSH authentication will only be disabled after key-based access has been tested successfully.  
Firewall rules must be validated from the relevant source machines. A rule existing in a configuration file is not sufficient evidence that the policy works.  
## **6. Implementation Work Packages**  
### **WP1 — Virtualization and network foundations**  
- Create the three virtual machines.  
- Configure the WAN, LAN, and DMZ virtual networks.  
- Identify and document each VM's network interfaces.  
- Configure static addresses and the client DHCP reservation.  
- Validate routes and basic reachability.  
**Acceptance criteria:**  
- Each VM starts successfully.  
- Interface assignments match the network design.  
- The LAN and DMZ use different subnets.  
- The initial routing tables are documented.  
- Connectivity results are recorded.  
### **WP2 — Gateway routing and NAT**  
- Enable IPv4 forwarding.  
- Configure forwarding and source NAT with nftables.  
- Set the correct default routes.  
- Validate outbound connectivity from the client.  
- Inspect packets on the gateway's internal and external interfaces.  
**Acceptance criteria:**  
- The client reaches an approved external test destination through the gateway.  
- The packet capture demonstrates the relevant address translation.  
- The gateway's routing and NAT configuration is documented.  
- A failed connectivity test can be diagnosed using routes, packet captures, and firewall counters.  
### **WP3 — DHCP and DNS**  
- Configure dnsmasq for the internal LAN.  
- Configure the client reservation.  
- Configure internal name resolution for web.lab.test.  
- Verify DNS forwarding and resolver behavior.  
- Document the difference between name resolution and IP connectivity.  
**Acceptance criteria:**  
- The client receives the expected address and network settings.  
- The internal hostname resolves to the web server's address.  
- DNS behavior is verified independently from HTTP connectivity.  
- Failure scenarios and corrective actions are recorded.  
### **WP4 — Web server and SSH security**  
- Install nginx on the web-server VM.  
- Serve a simple test page identifying the lab service.  
- Configure SSH key-based administration.  
- Restrict SSH to the authorized administrative source.  
- Verify service status and listening sockets.  
**Acceptance criteria:**  
- The web page is accessible from an authorized client.  
- SSH key authentication works.  
- Unauthorized SSH access is rejected.  
- The actual listening ports and service status are documented.  
### **WP5 — Stateful firewall and network isolation**  
- Define the forwarding policy.  
- Apply default-deny behavior for unapproved forwarded traffic.  
- Permit required established and related return traffic.  
- Restrict LAN-to-DMZ and DMZ-to-LAN traffic.  
- Restrict DMZ outbound connections.  
- Enable rate-limited logging of dropped packets.  
**Acceptance criteria:**  
- Each required allowed flow succeeds.  
- Each required denied flow fails for the expected reason.  
- Firewall counters and logs support the observed results.  
- The rules survive a controlled service or VM restart.  
- The configuration can be recovered if a firewall change causes loss of access.  
### **WP6 — Bash health checks**  
Implement scripts/healthcheck.sh.  
The script will:  
- Check the gateway's reachability.  
- Check DNS resolution for the internal web hostname.  
- Check the configured HTTP endpoint.  
- Report individual PASS or FAIL results.  
- Return a non-zero exit status when a required check fails.  
- Handle timeouts and unavailable commands gracefully.  
The script will first be executed manually. A systemd timer will be introduced only after the behavior and exit codes have been verified.  
### **WP7 — System audit**  
Implement scripts/audit.sh.  
The script will:  
- Report system information and uptime.  
- Report memory, swap, and disk usage.  
- List listening ports using ss.  
- Summarize relevant active services.  
- Collect available SSH authentication failure events.  
- Produce a timestamped report.  
- Handle missing log entries and insufficient permissions clearly.  
Audit reports must not expose passwords, private keys, or unnecessary personal information.  
### **WP8 — Configuration backup and restoration**  
Implement scripts/backup.sh.  
The script will:  
- Back up selected configuration files.  
- Include a timestamp in each archive.  
- Check whether archive creation succeeded.  
- Verify archive integrity.  
- Apply a defined retention policy.  
- Restore files into a temporary location for validation.  
- Document the recovery procedure.  
Backups will initially be tested manually. Cron scheduling will be added after successful backup and restoration tests.  
**Acceptance criteria:** the restoration test demonstrates that the expected configuration files can be recovered without overwriting the active configuration.  
### **WP9 — Troubleshooting and evidence**  
- Execute the complete connectivity test matrix.  
- Capture selected traffic with tcpdump.  
- Scan only the lab machines with nmap.  
- Verify firewall logs and counters.  
- Test backup restoration.  
- Document at least three meaningful troubleshooting incidents, including symptoms, diagnosis, root cause, fix, and verification.  
### **WP10 — Release preparation**  
- Review the README and technical documentation.  
- Check the scripts with bash -n and ShellCheck.  
- Verify that the test results correspond to the final configuration.  
- Review the repository for secrets and irrelevant files.  
- Merge the final feature branch through a pull request.  
- Create an annotated v1.0.0 tag only after the documented acceptance criteria are met.  
## **7. Bash Scripts and Repository Layout**  
Planned repository structure:  
secure-linux-network-lab/  
├── README.md  
├── LICENSE  
├── .gitignore  
├── config/  
│   ├── gateway/  
│   ├── client/  
│   └── web-server/  
├── scripts/  
│   ├── healthcheck.sh  
│   ├── audit.sh  
│   └── backup.sh  
├── tests/  
│   ├── test-healthcheck.sh  
│   └── test-matrix.md  
├── docs/  
│   ├── architecture.md  
│   ├── addressing-plan.md  
│   ├── firewall-policy.md  
│   ├── troubleshooting.md  
│   ├── backup-and-restore.md  
│   └── test-results.md  
├── diagrams/  
└── evidence/  
    ├── screenshots/  
    ├── packet-captures/  
    ├── sample-reports/  
    └── test-logs/  
The final structure may evolve as the implementation progresses. Empty directories and unimplemented scripts will not be presented as completed deliverables.  
Configuration files committed to Git must be safe to share. Private keys, passwords, environment secrets, full VM disk images, and unnecessary log archives must never be published.  
## **8. Validation and Test Matrix**  
Every test will record:  
- Test ID.  
- Date and environment.  
- Source and destination.  
- Command or procedure.  
- Expected result.  
- Actual result.  
- Pass/fail status.  
- Evidence path.  
- Notes or corrective action.  
The following matrix is a plan, not a claim that these tests have already passed.  
  
|  |  |  |  |  
| - | - | - | - |  
| **ID** |  **Test** |  **Expected result** |  **Status** |  
| NET-01 |  Inspect interface addresses |  Match the addressing plan |  Not tested |  
| NET-02 |  Verify default routes |  Match the network design |  Not tested |  
| NET-03 |  Client DHCP lease |  Expected address and gateway |  Not tested |  
| DNS-01 |  Resolve web.lab.test |  Returns the web-server IP |  Not tested |  
| NAT-01 |  Client outbound connectivity |  Works through gateway NAT |  Not tested |  
| WEB-01 |  Client HTTP request |  Succeeds |  Not tested |  
| SSH-01 |  Authorized SSH key login |  Succeeds |  Not tested |  
| SSH-02 |  Unauthorized SSH attempt |  Rejected and logged where configured |  Not tested |  
| FW-01 |  DMZ initiates connection to LAN |  Blocked |  Not tested |  
| FW-02 |  Unapproved DMZ outbound port |  Blocked |  Not tested |  
| FW-03 |  Unsolicited inbound WAN traffic |  Blocked |  Not tested |  
| LOG-01 |  Generate a blocked connection |  Relevant firewall log appears |  Not tested |  
| HC-01 |  Health check during normal operation |  Required checks pass |  Not tested |  
| HC-02 |  Health check with a controlled failure |  Failure is reported with non-zero exit status |  Not tested |  
| BAK-01 |  Create configuration backup |  Archive is created and validated |  Not tested |  
| BAK-02 |  Restore backup to temporary directory |  Expected files are recovered |  Not tested |  
  
Only tests that have actually been executed will be assigned a final pass/fail result.  
## **9. Evidence and Reproducibility**  
Evidence will be organized so that a reviewer can independently understand and assess the results.  
### **Network evidence**  
- Addressing and routing table outputs.  
- DHCP lease information.  
- DNS query results.  
- Successful and blocked connection tests.  
- Sanitized tcpdump output showing NAT.  
- Relevant nftables rules, counters, and logs.  
### **Linux and security evidence**  
- Active service and listening-port reports.  
- SSH key authentication results.  
- Relevant authentication logs.  
- Firewall behavior before and after applying the intended policy.  
- ShellCheck and Bash syntax-check results.  
### **Operations evidence**  
- Sample health-check reports.  
- Audit report examples.  
- Backup integrity verification.  
- Restoration test results.  
- Troubleshooting reports showing root cause and verified resolution.  
Screenshots will support the documentation but will not replace command output, test procedures, or explanations. All evidence must come from the actual lab and be reviewed before publication.  
## **10. Troubleshooting Method**  
For each significant failure, the project will document:  
1. **Symptom:** what failed and how the failure was observed.  
2. **Expected behavior:** what should have happened.  
3. **Investigation:** commands, logs, routes, counters, or captures examined.  
4. **Root cause:** the technical reason for the failure.  
5. **Fix:** the configuration or code change made.  
6. **Verification:** the test that demonstrated the correction.  
7. **Prevention:** how the issue could be detected earlier or avoided in future.  
Examples of possible investigation scenarios include incorrect routes, DNS resolution failures, blocked return traffic, a stopped web service, and a backup archive that cannot be restored. These scenarios will only be documented as incidents if they are actually reproduced or encountered.  

## **11. Git and Collaboration Workflow**

The project follows a feature-branch workflow to keep `main` stable and make changes reviewable.

### Branching strategy

- `main` is the primary integration branch and should remain in a working, documented state.
- Create a dedicated branch for each task.
- Use descriptive branch names with consistent prefixes:
  - `feature/` for new functionality, such as `feature/network-topology`.
  - `fix/` for bug fixes, such as `fix/firewall-rule`.
  - `docs/` for documentation changes, such as `docs/improve-git-workflow`.

### Development workflow

**1. Update the local main branch and create a task branch.**

```bash
git switch main
git pull --ff-only origin main
git switch -c docs/update-readme
```

Replace the example branch name with a descriptive name appropriate to the task.

**2. Review and stage changes.**

```bash
git status
git diff
git add README.md
git diff --staged
```

Review the staged changes before committing. Stage only the files relevant to the task.

**3. Create a descriptive commit and publish the branch.**

```bash
git commit -m "docs: improve Git workflow documentation"
git push -u origin docs/update-readme
```

Use a commit message that accurately describes the change. The branch name in the push command must match the current task branch.

**4. Open and review a pull request.**

- Open a pull request on GitHub with `main` as the base branch and the task branch as the compare branch.
- Explain the purpose of the change and summarize the implementation.
- Include relevant validation results and evidence when available.
- Review the diff and address any identified issues before merging.
- Merge only after the required reviews and checks have passed.

**5. Synchronize the local repository after the merge.**

```bash
git switch main
git pull --ff-only origin main
git branch -d docs/update-readme
```

If GitHub has not automatically deleted the remote branch, it can be deleted after confirming that the pull request was merged:

```bash
git push origin --delete docs/update-readme
git fetch origin --prune
```

Replace `docs/update-readme` with the actual branch name. Do not delete a branch containing work that has not been merged or otherwise preserved.

### Release management

- Keep commits focused and the Git history understandable.
- Use pull requests to document changes and their review.
- Create annotated version tags only for validated releases.
- Tag the release `v1.0.0` only after the implemented features, security tests, and documentation have been reviewed against the project's acceptance criteria.
## **12. Security and Scope**  
This is an isolated educational lab. All scans and security tests must target systems under the project's control.  
The project will not expose internal services to the public Internet. VirtualBox internal networks will be used for the LAN and DMZ, and inbound access will remain restricted.  
The lab will not store or publish real credentials, private SSH keys, or sensitive machine data.  
Docker, Terraform, Ansible, Kubernetes, and cloud deployment are intentionally outside the initial scope. They may be introduced in later projects after the Linux, networking, automation, and troubleshooting foundations have been demonstrated.  
## **13. Technology Stack**  
- Ubuntu Server LTS  
- Oracle VirtualBox  
- Netplan and Linux networking tools  
- Bash  
- nftables  
- dnsmasq  
- nginx  
- OpenSSH  
- systemd timers and cron  
- ip, ss, ping, curl, dig  
- tcpdump and nmap  
- bash -n and ShellCheck  
- Git and GitHub  
Tools will be introduced as needed rather than installed all at once.  
## **14. Project Roadmap**  
1. Prepare the VM environment and isolated networks.  
2. Configure addressing, routes, and basic connectivity.  
3. Implement gateway routing and NAT.  
4. Configure DHCP and DNS.  
5. Deploy nginx and secure SSH access.  
6. Implement the stateful firewall policy.  
7. Develop and test the Bash scripts.  
8. Validate security, logging, backup, and restoration.  
9. Publish sanitized evidence and troubleshooting reports.  
10. Review the project and prepare a validated release.  
## **15. Definition of Done**  
The project will be considered complete when:  
- The documented architecture matches the working lab.  
- IP addressing, routing, DNS, DHCP, and NAT have been verified.  
- The intended firewall policy passes its required positive and negative tests.  
- SSH administration uses the intended access restrictions.  
- All three Bash scripts have documented behavior and error handling.  
- Backup restoration has been tested in a safe location.  
- The test matrix contains actual results and evidence references.  
- Troubleshooting cases explain verified causes and fixes.  
- The repository contains no committed secrets.  
- The README and supporting documentation reflect the final implementation.  
- The final Git history and release tag have been reviewed.  
## **16. Learning Philosophy**  
**Build → Test → Break safely → Diagnose → Fix → Verify → Document**  
The purpose of this project is to demonstrate practical understanding, not just familiarity with command syntax.  
Each feature should be explainable, each important result reproducible, and each claimed capability supported by evidence.  
## **License**  
This project is intended to be distributed under the MIT License. See the LICENSE file for the applicable terms.  
   
