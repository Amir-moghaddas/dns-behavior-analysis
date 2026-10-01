# 🌐 When DNS Packets Get Lost

🔍 A technical analysis of unusual DNS resolution behavior observed in a restricted network environment.

![Status](https://img.shields.io/badge/Status-Complete-green)
![Tools](https://img.shields.io/badge/Tools-Wireshark%20%7C%20nslookup-blue)
![Focus](https://img.shields.io/badge/Focus-IoT%20%7C%20Network%20Security-orange)

---

## 📋 Overview

I was practicing network commands (`nslookup`, `dig`) as part of a networking course. At first everything looked normal. Then I tried a few other domains — `netflix.com` among them — just to see how resolution behaves.

Before running `nslookup`, I opened Wireshark so I could watch the queries and responses.

---

## 🔧 Method

The course usually says to use `8.8.8.8` or `1.1.1.1` for reliability, so I queried `netflix.com` against both:

```bash
nslookup netflix.com 8.8.8.8
nslookup netflix.com 1.1.1.1
```

Both returned:

```
10.10.34.36
```

That's a private IP range (`10.0.0.0/8`). It shouldn't show up as the answer for a public domain.

---

## 🔍 Wireshark Analysis

I went back to Wireshark and checked the packet details. Everything looked normal on the surface:

- 1 answer
- 0 packet loss
- No retransmissions

But the `Authority RRs` field was **0**, which I didn't expect. The IPv6 response was also odd:

```
2001:4188:2:600:10:10:34:36
```

The tail (`10:10:34:36`) matches the IPv4 answer. That doesn't look like a coincidence.

I tested the IP directly — it didn't belong to `netflix.com`.

---

## 🌍 External Verification

To double-check, I used `dnschecker.org`. From outside my network, `netflix.com` resolved to its normal public IPs, completely different from what I got.

---

## 📊 Observations

- Both `8.8.8.8` and `1.1.1.1` returned the same private IP.
- The IPv6 response mirrored the IPv4 result.
- The `Authority RRs` field was zero.
- From outside the network, the correct public IPs were returned.

These observations suggest that **something between the client and the resolver is shaping the DNS response** — possibly an intermediate network device or a transparent DNS proxy.

---

## ❓ Open Questions

- Why did both independent resolvers return the same private IP?
- Is this a transparent DNS proxy or something else?
- Would DNSSEC have caught this? (Not tested)
- Does this happen on other domains too?

---

## 🤖 Why This Matters for IoT and Robotics

If DNS answers can be rewritten silently, any device that trusts DNS — sensors, robots, controllers — can be pointed at the wrong server. In a multi-robot setup, one wrong DNS answer could break coordination between nodes.

---

## 🛠️ Environment

| Component | Details |
| :--- | :--- |
| 🌐 Network | Restricted network with DNS interception |
| 🦈 Wireshark | 4.x |
| 🔍 nslookup | Built-in |
| 📡 dig | Not used |

---

## 📁 Files

| File | Description |
| :--- | :--- |
| 📄 [nslookup.txt](outputs/nslookup.txt) | Raw terminal output from the DNS queries |
| 📦 [capture.pcapng](outputs/capture.pcapng) | Full Wireshark packet capture |

---

## 📚 References

- RFC 1918 — Address Allocation for Private Internets
- RFC 4033 — DNS Security Introduction and Requirements
- Wireshark Documentation
- dnschecker.org

---

## 👤 Author

**Amir Moghaddas**
Computer Engineering Student | IoT & Robotics Enthusiast

- LinkedIn: [linkedin.com/in/amir-moghaddas-027990388](https://www.linkedin.com/in/amir-moghaddas-027990388)
- GitHub: [github.com/Amir-moghaddas](https://github.com/Amir-moghaddas)
