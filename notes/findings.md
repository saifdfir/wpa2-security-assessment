# WPA2 Wireless Security Assessment — Findings

## Finding 01: Protected Management Frames (PMF) Not Advertised as Capable

### 1. Description / Observation

While analyzing the router's beacon frames in Wireshark, I found that Protected Management Frame (PMF) capability was not advertised on the 2.4 GHz network in the tested configuration.

The observed configuration was:

- **Security:** WPA2-PSK
- **Cipher:** CCMP
- **Channel:** 9 (2.4 GHz)
- **MFPC:** 0 (Not Capable)
- **MFPR:** 0 (Not Required)

I had previously observed the same PMF settings on the router's 5 GHz network.

### 2. Evidence

The finding was identified by examining the RSN Capabilities field in an 802.11 beacon frame captured using Wireshark.

The relevant fields showed:

- **Management Frame Protection Required:** No
- **Management Frame Protection Capable:** No

[Redacted screenshot of the beacon's RSN Capabilities](../evidence/pmf-capabilities.png).

### 3. Security Impact

Without PMF, the network does not provide cryptographic protection against forged deauthentication and disassociation frames.

An attacker within wireless range could potentially impersonate the access point and transmit forged management frames to disrupt a connected device's Wi-Fi connection.

The beacon evidence establishes that PMF capability was not advertised in the tested configuration. It does not, by itself, confirm that a deauthentication attempt would successfully disconnect a client.

### 4. Risk Rating

**Preliminary Severity: Medium**

The lack of PMF may expose connected devices to wireless denial-of-service attacks involving forged management frames.

However, this finding primarily concerns connection availability rather than direct exposure of encrypted network traffic or passwords.

The severity is preliminary and may be revised based on controlled testing, client behavior and the importance of uninterrupted connectivity in the assessed environment.

### 5. Recommendation

Enable PMF if the router supports it.

Where client compatibility allows, configure PMF in Required mode to ensure connected devices use management-frame protection.

If supported, WPA3-Personal is another option, as it requires PMF.

After making any configuration changes, verify the advertised RSN capabilities and confirm that connected devices negotiate PMF successfully.
