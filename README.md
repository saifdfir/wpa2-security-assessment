# WPA2 Wireless Security Assessment

## Project Overview

- **Network:** A personally owned WPA2 wireless network, assessed using Kali Linux.
- **Assessment:** Review of the security configuration, deauthentication resistance, WPA2 handshake capture and analysis, and offline password verification using GPU acceleration.
- **Traffic analysis:** Wireshark review of metadata visible without the decryption keys.
- **Authorization:** Active testing was limited to the router and test device.

## Objectives

- Examine the network's WPA2 configuration and Protected Management Frame (PMF) capabilities.
- Test whether forged deauthentication frames can disrupt an authorized client connection.
- Capture and analyze the WPA2 four-way handshake.
- Demonstrate offline dictionary-based password verification using Hashcat.
- Compare pairwise and group encryption in WPA2-CCMP.
- Analyze the metadata visible in encrypted wireless traffic.

## Testing Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| Wireless Adapter | Intel Wi-Fi adapter (monitor mode) |
| GPU | NVIDIA GeForce RTX 4050 Laptop GPU |
| Wireless Security | WPA2-Personal (PSK) |
| Encryption | CCMP |
| Primary Test Band | 2.4 GHz, Channel 9 |
| Test Client | Personal smartphone |

## Tools Used

- **Aircrack-ng Suite:** Wireless monitoring, packet capture and controlled deauthentication testing
- **Wireshark:** Packet inspection and traffic analysis
- **hcxtools:** WPA2 handshake conversion
- **Hashcat:** GPU-accelerated offline password testing

## Testing Methodology

### 1. Wireless Network Reconnaissance

- **Network discovery:** Airodump-ng identified the wireless network and its security configuration.
- **Configuration:** The assessment targeted the 2.4 GHz network on channel 9, configured with WPA2-PSK and CCMP encryption.
- **PMF review:** The access point's beacon frames were examined in Wireshark for advertised Protected Management Frame (PMF) capabilities.

### 2. Deauthentication Testing and Handshake Capture

- **PMF configuration:** The access point did not advertise PMF capability in the tested configuration.
- **Test:** One targeted deauthentication burst was transmitted against the test smartphone while Airodump-ng monitored the network.
- **Outcome:** The phone disconnected and reconnected. The WPA2 four-way handshake was captured during reconnection, and all four EAPOL-Key messages were identified in Wireshark.

### 3. Offline Password Recovery

- **Conversion:** The captured handshake was converted to Hashcat's HC22000 format using `hcxpcapngtool`.
- **Dictionary test:** Hashcat tested candidates offline with GPU acceleration and a prepared RockYou-based wordlist.
- **Execution:** The test ran locally on an NVIDIA RTX 4050 Laptop GPU and required no additional authentication requests to the router.

### 4. WPA2 Encryption and Traffic Analysis

- **Frame analysis:** Protected 802.11 Data frames were compared in Wireshark to review pairwise and group encryption, destination addresses, and CCMP Key Index values.
- **IP filter:** A Wireshark display filter checked for decoded IP packets within protected frames.
- **Traffic patterns:** Wireshark's I/O Graphs were used to review packet rates and activity visible despite wireless encryption.

## Results and Evidence

### 1. WPA2 Security Configuration

The access point was configured to use WPA2-PSK with CCMP encryption.

Wireshark analysis of its beacon frames showed that the access point did not advertise Protected Management Frame (PMF) capability in the tested configuration.

The RSN Capabilities showed:

- **MFPC:** 0 (Not Capable)
- **MFPR:** 0 (Not Required)

The same configuration had previously been observed on the router's 5 GHz network.

Without PMF, connected devices lack its protection against forged deauthentication and disassociation frames.

A detailed explanation of this finding, including its security impact and recommendations, is available in [Security Findings](notes/findings.md).

![PMF capabilities in the access point beacon](evidence/pmf-capabilities.png)

### 2. Deauthentication and Handshake Capture

A targeted deauthentication test was conducted against the test smartphone while Airodump-ng monitored the access point.

**Results:**

- Deauthentication bursts transmitted: 1
- Client disconnection: Successful
- Client reconnection: Successful
- WPA2 four-way handshake: Captured successfully

Wireshark showed all four EAPOL-Key messages in the capture.

The capture also contained multiple retransmissions of Message 1 before the client responded. Retransmission behavior was not analyzed further in this experiment.

The test showed that the targeted deauthentication attempt disconnected the test client and triggered a new WPA2 handshake.

![Four WPA2 handshake messages in Wireshark](evidence/handshake.png)

### 3. Offline WPA2 Password Recovery

The captured handshake was converted to Hashcat's HC22000 format with `hcxpcapngtool`.

Hashcat performed an offline dictionary attack with GPU acceleration.

**Results:**

| Metric | Result |
|---|---|
| Tool | Hashcat v7.1.2 |
| GPU | NVIDIA RTX 4050 Laptop GPU |
| Attack Mode | Straight Dictionary Attack |
| Hash Mode | 22000 |
| Wordlist | RockYou-based test dictionary |
| Candidate progress, including rejected candidates | 14,344,386 / 14,344,386 |
| Rejected Candidates | 4,734,913 |
| Reported testing speed at completion | 234.8 kH/s |
| Attack Runtime | Approximately 41 seconds |
| Password Recovery | Successful |

The test verified password candidates against the captured WPA2 handshake without further communication with the access point.

The prepared dictionary included the test credential, so this result demonstrates offline verification rather than recovery of an unknown password.

![Hashcat offline password verification results](evidence/hashcatresults.png)

### 4. WPA2 Encryption and Traffic Analysis

Protected 802.11 Data frames were examined in Wireshark to compare pairwise and group encryption.

**Observed results:**

| Property | Unicast Traffic | Broadcast Traffic |
|---|---|---|
| Example Packet | #82 | #34 |
| Destination | Individual client | Broadcast address |
| CCMP Key Index | 0 | 1 |
| Key Type | Pairwise Temporal Key | Group Temporal Key |
| Protected Frame | Yes | Yes |

A display filter for protected 802.11 Data frames and IP traffic returned **0 matching packets**. No IP packets were decoded from the protected frames in this capture.

The traffic was also reviewed using Wireshark's I/O Graphs.

Packet activity was relatively low early in the capture and increased after the client's reconnection. The peak observed rate was approximately **300 packets per second**.

Although WPA2-CCMP protects the contents of wireless data frames, MAC addresses, frame sizes, packet timing and traffic volume remain observable.

#### Encrypted Traffic Analysis — Wireshark I/O Graph

![Encrypted wireless traffic analysis](evidence/encrypted-traffic-graph.png)

The graph shows the packet rate of protected 802.11 Data frames involving the test client, measured at one-second intervals.

## Conclusion

The tests showed that the access point did not advertise PMF and that a targeted deauthentication burst disconnected the test phone before it reconnected. The captured handshake was then used to check the prepared test password offline. Without the decryption keys, Wireshark could not show the protected payloads, but MAC addresses, frame sizes, timing and traffic volume remained visible.

## Limitations

- The assessment was conducted on a single home router and personal test device.
- Deauthentication testing was limited to a controlled experiment, so the results may differ with other devices and network configurations.
- The network was intentionally configured with a test password included in the wordlist to demonstrate offline password recovery in the lab.
- The traffic analysis focused on observable wireless metadata rather than decrypting application traffic.
- The assessment did not cover WPA3-SAE, enterprise authentication or other wireless security configurations.

## Security Recommendations

The following changes would improve the network's security:

- Enable PMF where supported, preferably in Required mode when compatible with connected devices.
- Use a strong, unique Wi-Fi passphrase to reduce the effectiveness of offline password-guessing attacks.
- Consider WPA3-Personal where supported.
- Keep router firmware updated and periodically review wireless security settings.
