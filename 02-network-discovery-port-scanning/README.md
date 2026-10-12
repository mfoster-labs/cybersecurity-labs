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

Before beginning the scans, I verified that the three virtual machines retained their network configurations from Lab 01.

On Kali, I checked the IP address and routing table using:

```bash
ip address show dev eth0
ip route
```

On the Ubuntu Observer, I used the same commands with its `enp0s3` network interface:

```bash
ip address show dev enp0s3
ip route
```

On Metasploitable 2, I used:

```bash
ifconfig eth0
route -n
```

All three systems retained their assigned static IP addresses on the `10.10.10.0/24` subnet. Each system had a route to the local network and no default gateway configured.

I then tested connectivity from Kali to Metasploitable 2 and the Ubuntu Observer:

```bash
ping -c 2 10.10.10.20
ping -c 2 10.10.10.30
```

Both systems responded successfully with 0% packet loss.

![Kali connectivity validation](images/01-kali-connectivity.png)

I also tested connectivity from the Ubuntu Observer to Metasploitable 2:

```bash
ping -c 2 10.10.10.20
```

This test also completed successfully with 0% packet loss.

These results confirmed that all three virtual machines could communicate within the isolated lab network and were ready for scanning.

## 2. Host Discovery

After confirming network connectivity, I used Nmap on Kali Linux to identify active hosts on the `10.10.10.0/24` network.

Before starting the scan, I opened Wireshark on the Ubuntu Observer and began capturing traffic on the `enp0s3` interface.

I then ran the following command from Kali:

```bash
nmap -sn 10.10.10.0/24
```

The `-sn` option tells Nmap to perform host discovery without conducting a port scan. The `10.10.10.0/24` argument specifies the network to scan.

Nmap scanned 256 IP addresses and identified three active hosts:

| IP Address | System | MAC Address |
|---|---|---|
| 10.10.10.10 | Kali Linux | Local scanning system |
| 10.10.10.20 | Metasploitable 2 | 08:00:27:d3:c5:c0 |
| 10.10.10.30 | Ubuntu Observer | 08:00:27:81:8b:a9 |

The scan identified all three virtual machines on the isolated lab network.

![Nmap host discovery results](images/02-nmap-host-discovery.png)

### ARP Traffic Analysis

After completing the scan, I examined the captured traffic in Wireshark.

Because the systems were connected to the same local Ethernet network, Nmap used Address Resolution Protocol (ARP) requests to discover active hosts.

The capture showed Kali broadcasting ARP requests across the subnet, asking which devices were associated with specific IP addresses.

For example, Kali sent the following request for Metasploitable 2:

`Who has 10.10.10.20? Tell 10.10.10.10`

Metasploitable responded with its MAC address:

`10.10.10.20 is at 08:00:27:d3:c5:c0`

A similar exchange occurred with the Ubuntu Observer at `10.10.10.30`.

![Wireshark ARP discovery analysis](images/03-arp-discovery-analysis.png)

I inspected the Ethernet and ARP headers to examine how the requests and replies were structured.

The ARP requests used the Ethernet broadcast destination `ff:ff:ff:ff:ff:ff`, allowing devices on the local network to receive them.

The replies were sent directly to Kali's MAC address and contained the responding systems' IP and MAC addresses.

The ARP headers distinguished the two message types using their operation codes:

- Request: Opcode 1
- Reply: Opcode 2

I also noticed differences in the ARP traffic between the two systems. Metasploitable 2 received one ARP request and sent one reply, while Ubuntu received six requests and responded to all six.

Metasploitable 2's ARP reply was 60 bytes, including 18 bytes of Ethernet padding. All six Ubuntu replies were 42 bytes, with no padding present in the captured frames.

The captured traffic confirmed how Nmap used ARP responses to identify active systems and obtain their MAC addresses without performing a port scan.

## 3. TCP Port Scanning

After discovering the active hosts, I used Nmap on Kali Linux to scan TCP ports on Metasploitable 2 (`10.10.10.20`).

The objective was to identify open and closed TCP ports, compare different scanning techniques, and examine the corresponding network traffic in Wireshark.

### TCP SYN Scanning

I began with a targeted TCP SYN scan of five ports:

```bash
sudo nmap -sS -p 21,22,80,443,65000 10.10.10.20
```

The `-sS` option performs a TCP SYN scan, while `-p` specifies which ports to scan.

Nmap reported the following results:

| Port | State | Service |
|---|---|---|
| 21/tcp | open | ftp |
| 22/tcp | open | ssh |
| 80/tcp | open | http |
| 443/tcp | closed | https |
| 65000/tcp | closed | unknown |

![TCP SYN scan results](images/04-tcp-syn-scan.png)

I captured the scan traffic using Wireshark on the Ubuntu Observer to examine how Nmap determined the port states.

For open ports, Kali sent a TCP SYN packet and Metasploitable responded with SYN-ACK. Kali then sent RST to terminate the attempted connection without completing the TCP three-way handshake.

For closed ports, Metasploitable responded to the SYN probe with RST-ACK, rejecting the connection attempt.

I examined the TCP headers for both responses in Wireshark.

The following capture shows the SYN-ACK response from port 22, indicating that the port was open.

![Open TCP port SYN-ACK response](images/05-tcp-syn-open.png)

The next capture shows the RST-ACK response from port 443, indicating that the port was closed.

![Closed TCP port RST-ACK response](images/06-tcp-syn-closed.png)

These responses demonstrated how Nmap distinguishes between TCP ports accepting connections and ports that are reachable but not accepting connections.

### TCP Connect Scanning

I also performed a TCP connect scan against port 22:

```bash
nmap -sT -p 22 10.10.10.20
```

The `-sT` option uses the operating system's TCP connection mechanism to establish a complete connection rather than performing a SYN-only scan.

Nmap reported port 22 as open.

The Wireshark capture showed the following sequence:

1. Kali sent SYN to port 22.
2. Metasploitable responded with SYN-ACK.
3. Kali sent ACK, completing the TCP three-way handshake.
4. Kali sent RST-ACK to terminate the connection.

![TCP connect scan packet analysis](images/07-tcp-connect-analysis.png)

Unlike the SYN scan, the TCP connect scan completed the three-way handshake before terminating the connection. This difference was visible in the captured TCP packets.

### Default vs. Full TCP Port Scanning

After examining individual ports, I performed a broader TCP SYN scan using Nmap's default port selection:

```bash
sudo nmap -sS 10.10.10.20
```

By default, Nmap scans the 1,000 most commonly used TCP ports.

The scan identified 23 open ports and 977 closed ports.

![Default TCP port scan results](images/08-tcp-default-scan.png)

I then scanned the entire TCP port range:

```bash
sudo nmap -sS -p- 10.10.10.20
```

The `-p-` option specifies all 65,535 TCP ports.

The full scan identified 30 open ports and 65,505 closed ports.

![Full TCP port scan results](images/09-tcp-full-scan.png)

Comparing the two scans revealed seven additional open ports that were not included in the default scan:

| Port | Nmap Service Label |
|---|---|
| 3632/tcp | distccd |
| 6697/tcp | ircs-u |
| 8787/tcp | msgsrvr |
| 33189/tcp | unknown |
| 42837/tcp | unknown |
| 45451/tcp | unknown |
| 55971/tcp | unknown |

The full scan discovered 30 open TCP ports compared with 23 in the default scan, demonstrating that scanning only the default 1,000 ports can miss additional open ports.

The service names displayed by Nmap were based on known port associations rather than service/version detection. Identifying the actual services and software running on these ports will be covered in Lab 03.

### TCP ACK and FIN Scan Comparison

I also compared TCP ACK and FIN scanning techniques against port 22, which was open, and port 65000, which was closed.

First, I performed a TCP ACK scan:

```bash
sudo nmap -sA -p 22,65000 10.10.10.20
```

The `-sA` option sends TCP ACK probes to investigate whether traffic is being filtered. Unlike SYN scanning, it does not directly determine whether a port is open or closed.

Both ports were reported as `unfiltered`.

I then performed a TCP FIN scan:

```bash
sudo nmap -sF -p 22,65000 10.10.10.20
```

The `-sF` option sends TCP FIN probes without first establishing a connection.

The FIN scan reported port 22 as `open|filtered` and port 65000 as `closed`.

![TCP ACK and FIN scan results](images/10-tcp-ack-fin-scans.png)

The results were compared with the earlier SYN scan:

| Port | SYN Scan | ACK Scan | FIN Scan |
|---|---|---|---|
| 22/tcp | open | unfiltered | open\|filtered |
| 65000/tcp | closed | unfiltered | closed |

I examined the ACK and FIN scan traffic in Wireshark to understand the differences in the responses.

During the ACK scan, both ports responded with TCP RST packets. Nmap classified both as `unfiltered`, despite one port being open and the other closed.

During the FIN scan, port 65000 responded with RST-ACK, while repeated FIN probes to port 22 received no response.

![TCP ACK and FIN packet analysis](images/11-tcp-ack-fin-analysis.png)

The `open|filtered` result for port 22 did not indicate that a firewall was blocking traffic. It meant Nmap could not determine whether the port was open and ignoring the FIN probes or whether the probes were being filtered.

These experiments demonstrated how different TCP scanning techniques can produce different classifications for the same ports, depending on the probes sent and responses received.

No ports were confirmed to be filtered during this lab. Testing how actual firewall filtering affects scan results was left for a future lab, since configuring firewall rules was outside the scope of this introductory port-scanning exercise.

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
