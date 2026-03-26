# MITM SSL Vulnerability Lab

A hands-on cybersecurity lab demonstrating a Man-in-the-Middle (MITM) attack on HTTPS traffic in a controlled virtual environment.

Using a Linux VM and a mobile device on the same network, this project intercepts, inspects, and analyzes encrypted traffic by exploiting SSL/TLS certificate trust — and documents what happens when certificate validation is properly implemented vs. when it is not.

---

## Objectives

- Understand how MITM attacks work in practice
- Intercept and analyze HTTP and HTTPS traffic using a proxy
- Identify security weaknesses in certificate validation
- Observe how certificate pinning prevents interception

---

## Technologies Used

| Tool | Purpose |
|------|---------|
| Oracle VM VirtualBox | Virtual machine hosting |
| Lubuntu 24.04 | Lab operating system |
| mitmproxy | Traffic interception and analysis |
| Android / iOS device | Target client |
| Shared WiFi network | Attack surface |

---

## Lab Architecture

```
Phone → mitmproxy (Lubuntu VM) → Internet
```

The proxy sits between the mobile client and the internet, intercepting all outbound requests and inbound responses.

---

## Setup Process

### 1. Virtual Machine Setup

- Installed VirtualBox and created a Lubuntu 24.04 VM
- Allocated appropriate RAM and storage
- Booted into desktop environment

### 2. Network Configuration

- Set network adapter to **Bridged Mode**
- Connected VM to the same WiFi network as the mobile device

> **Issue encountered:** Bridged adapter failed due to a missing driver.
> **Fix applied:** Reinstalled VirtualBox using `--msiparams NETWORKTYPE=NDIS6`, temporarily disabled antivirus, and reinstalled networking drivers.

### 3. Install mitmproxy

```bash
sudo apt update
sudo apt install mitmproxy
mitmproxy
```

### 4. Configure Mobile Proxy

- Connected phone to the same WiFi network
- Set manual proxy:
  - **IP:** VM IP address
  - **Port:** 8080

### 5. Install mitmproxy Certificate on Device

- Navigated to `http://mitm.it` on the phone
- Downloaded and installed the mitmproxy CA certificate

> Without installing the certificate, HTTPS traffic cannot be decrypted — the browser will throw a certificate error and block the connection.

---

## What Was Captured

- HTTP requests and full responses
- Decrypted HTTPS traffic (post certificate installation)
- Request headers including User-Agent and cookies
- API calls from mobile applications
- Structured endpoint data from app network activity

---

## Security Findings

### 1. HTTPS is only secure if certificate trust is maintained
If a user installs a malicious CA certificate, all encrypted traffic can be decrypted transparently — the padlock icon does not protect against this.

### 2. Certificate validation is critical
Applications that do not properly validate the certificate chain are fully vulnerable to interception.

### 3. Certificate pinning works
Apps with certificate pinning actively blocked interception — connections failed and traffic was never exposed. This is the correct defense.

### 4. User behavior is a weak point
Users can unknowingly install malicious certificates (e.g., through a social engineering attack), enabling a full MITM without any technical exploit.

---

## Mitigation and Defense

- Implement certificate pinning in mobile and desktop apps
- Never install certificates from untrusted sources
- Avoid configuring manual proxies on unknown networks
- Enforce strict TLS validation at the application layer
- Keep OS, browsers, and apps updated

---

## Challenges and How I Fixed Them

| Challenge | Root Cause | Fix |
|-----------|-----------|-----|
| Bridged networking failed | Missing VirtualBox NDIS6 driver | Reinstalled VirtualBox with `--msiparams NETWORKTYPE=NDIS6` |
| HTTPS traffic not appearing | Certificate not installed on device | Installed mitmproxy CA cert via `http://mitm.it` |
| Some apps could not be intercepted | Certificate pinning enabled | Expected behavior — documented as a finding |

---

## Project Structure

```
mitm-security-lab/
│
├── README.md
├── screenshots/
│   ├── vm_setup.png
│   ├── bridged_network.png
│   ├── mitmproxy_running.png
│   ├── https_intercept.png
│   └── phone_proxy.png
│
└── logs/
    └── flows.mitm
```

---

## Verification

All testing was conducted in a controlled lab environment on devices owned and authorized for testing. This project is for educational and cybersecurity research purposes only.

---

## Key Takeaway

This lab highlights the gap between theoretical security and real-world implementation. A correctly configured HTTPS connection is secure — but trust in the certificate chain is the foundation that security rests on. Break that trust, and encryption alone is not enough.
