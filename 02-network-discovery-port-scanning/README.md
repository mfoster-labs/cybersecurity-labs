# Lab 02 - Network Discovery and Port Scanning

## Objective

The goal of this lab is to use Nmap to discover active hosts and identify open TCP and UDP ports within the isolated network created in Lab 01.

Wireshark will also be used to capture and analyze the traffic generated during scanning to better understand how host discovery and port scanning work at the packet level.

## Lab Environment

This lab uses the three virtual machines configured in [Lab 01](../01-isolated-lab-environment/).

| System | Role | IP Address |
|---|---|---|
| Kali Linux | Scanning System | 10.10.10.10/24 |
| Metasploitable 2 | Vulnerable Target | 10.10.10.20/24 |
| Ubuntu Observer | Network Monitoring | 10.10.10.30/24 |

All three systems communicate through the VirtualBox Internal Network named `Labnet`, using the `10.10.10.0/24` subnet.

The network remains isolated from the host's normal network and the Internet.

## Methodology

The lab will begin with network validation and host discovery, followed by TCP and UDP port scanning.

Nmap will be used from Kali Linux to scan the lab network and the Metasploitable 2 target. Wireshark will be used on the Ubuntu Observer to capture and examine the network traffic generated during these scans.

Before each scan, I will review the relevant commands and options to understand their purpose and expected results. After running each scan, I will examine the output and captured packets to understand the responses.

Commands, observations, and screenshots will be documented throughout the lab.

## 1. Network Validation

Verify that all three virtual machines have the correct network configurations and can communicate within the isolated network.

*Documentation and screenshots to be added.*

## 2. Host Discovery

Use Nmap to identify active systems on the lab network.

Capture and examine the discovery traffic in Wireshark to identify the protocols and packets used.

*Documentation and screenshots to be added.*

## 3. TCP Port Scanning

Use Nmap to scan the Metasploitable 2 target and identify open TCP ports.

Examine the captured traffic in Wireshark to understand how TCP responses are used to determine whether ports are open, closed, or filtered.

*Documentation and screenshots to be added.*

## 4. UDP Port Scanning

Use Nmap to scan selected UDP ports on the Metasploitable 2 target.

Analyze UDP and ICMP traffic in Wireshark to understand how UDP scanning differs from TCP scanning and why some port states cannot be determined conclusively.

*Documentation and screenshots to be added.*

## 5. Findings

Summarize the discovered hosts, TCP and UDP scan results, and relevant observations from the packet captures.

*Findings to be added.*

## Result

*To be completed after the lab.*

## What I Learned

*To be completed after the lab.*

## References

- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [Nmap Host Discovery](https://nmap.org/book/man-host-discovery.html)
- [Nmap Port Scanning Techniques](https://nmap.org/book/man-port-scanning-techniques.html)
- [Nmap Port Scanning Basics](https://nmap.org/book/man-port-scanning-basics.html)
- [Wireshark Documentation](https://www.wireshark.org/docs/)
