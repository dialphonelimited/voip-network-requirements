# VoIP Network Requirements

Technical specifications for preparing enterprise networks for Voice over IP deployment. Written for network engineers, IT managers, and MSPs.

*Authored by the DialPhone Limited infrastructure team.*

---

## Bandwidth Requirements

### Per-Call Bandwidth by Codec

| Codec | Payload (Kbps) | With IP/UDP/RTP Overhead | Recommended Provision |
|-------|---------------|------------------------|---------------------|
| G.711 (PCMU/PCMA) | 64 | 87.2 | 100 Kbps |
| G.729 | 8 | 31.2 | 40 Kbps |
| G.722 (wideband) | 64 | 87.2 | 100 Kbps |
| Opus (default) | 24-48 | 48-72 | 80 Kbps |
| Opus (HD voice) | 64 | 87.2 | 100 Kbps |

### Total Bandwidth Planning

```
Required bandwidth = (Concurrent calls x Per-call bandwidth) x 1.2 safety margin
```

| Office Size | Typical Concurrent Calls | G.711 Bandwidth | Opus Bandwidth | Minimum Circuit |
|------------|------------------------|----------------|---------------|----------------|
| 10 users | 3-5 | 500 Kbps | 400 Kbps | 5 Mbps |
| 25 users | 8-12 | 1.2 Mbps | 960 Kbps | 10 Mbps |
| 50 users | 15-25 | 2.5 Mbps | 2 Mbps | 25 Mbps |
| 100 users | 30-50 | 5 Mbps | 4 Mbps | 50 Mbps |
| 250 users | 75-125 | 12.5 Mbps | 10 Mbps | 100 Mbps |

**Rule of thumb:** Provision 2x the calculated bandwidth minimum. Voice quality degrades unpredictably when utilization exceeds 70%.

## Quality of Service (QoS)

### DSCP Markings

| Traffic Type | DSCP Value | Per-Hop Behavior | Priority |
|-------------|-----------|-----------------|----------|
| Voice media (RTP) | 46 (EF) | Expedited Forwarding | Highest |
| Call signaling (SIP) | 24 (CS3) | Class Selector 3 | High |
| Video (if applicable) | 34 (AF41) | Assured Forwarding | Medium-High |
| Data (default) | 0 (BE) | Best Effort | Normal |

### Router Configuration Example (Cisco IOS)

```
policy-map VOICE-QOS
  class VOICE-RTP
    priority percent 30
    set dscp ef
  class VOICE-SIP
    bandwidth percent 5
    set dscp cs3
  class class-default
    fair-queue
```

## Firewall Requirements

### Required Ports

| Protocol | Port | Direction | Purpose |
|----------|------|-----------|--------|
| SIP (TLS) | 5061/TCP | Outbound | Encrypted call signaling |
| SIP (UDP) | 5060/UDP | Outbound | Unencrypted signaling (fallback) |
| RTP | 10000-20000/UDP | Both | Voice media |
| SRTP | 10000-20000/UDP | Both | Encrypted voice media |
| STUN | 3478/UDP | Outbound | NAT traversal |
| HTTPS | 443/TCP | Outbound | Provisioning, admin portal |

### Critical: Disable SIP ALG

SIP Application Layer Gateway (ALG) is enabled by default on most consumer and many business routers. It modifies SIP headers in ways that break VoIP calls.

**Symptoms of SIP ALG issues:**
- One-way audio
- Calls dropping after 30 seconds
- Registration failures
- Intermittent call quality issues

**Fix:** Disable SIP ALG on your router. Every router vendor has a different method â€” consult your router documentation.

## Network Testing Procedures

### Before Deployment

Run these tests during peak business hours (10 AM - 2 PM), not off-hours:

| Test | Tool | Acceptable | Concerning | Unacceptable |
|------|------|-----------|-----------|-------------|
| Jitter | `iperf3 -u` or VoIP provider test | < 20ms | 20-30ms | > 30ms |
| Packet loss | `ping -n 1000` to provider | < 0.5% | 0.5-1% | > 1% |
| Latency (one-way) | Provider test tool | < 80ms | 80-150ms | > 150ms |
| MOS score | Provider test tool | > 4.0 | 3.5-4.0 | < 3.5 |
| Bandwidth (actual) | speedtest to provider region | > 2x calculated need | 1.5-2x | < 1.5x |

### VLAN Configuration

Voice traffic must be on a separate VLAN from data:

```
# Example: Cisco switch
vlan 100
  name DATA
vlan 200
  name VOICE

interface GigabitEthernet0/1
  switchport mode access
  switchport access vlan 100
  switchport voice vlan 200
  spanning-tree portfast
```

## Common Deployment Issues

| Issue | Root Cause | Resolution |
|-------|-----------|------------|
| One-way audio | NAT/firewall blocking RTP | Open RTP port range, enable STUN |
| Audio cuts out | Bandwidth contention | Implement QoS, verify bandwidth |
| Calls drop at 30s | SIP session timer mismatch | Adjust timer on PBX or provider side |
| Echo | Impedance mismatch or acoustic | Use headset, check hybrid balance |
| Registration failure | DNS or credential issue | Verify SRV records, check password |
| Delayed audio start | SRTP key negotiation | Verify both sides support same crypto |

## Provider Network Requirements

When evaluating VoIP providers, verify their infrastructure:

- [ ] Multiple geographically distributed data centers
- [ ] Carrier-grade SBC (Session Border Controller) deployment
- [ ] DDoS protection on SIP infrastructure
- [ ] Published network status page with real-time monitoring
- [ ] Peering agreements with major ISPs
- [ ] Support for STUN/TURN/ICE for NAT traversal

[VestaCall](https://vestacall.com) operates redundant infrastructure across multiple regions with carrier-grade SBCs and published uptime monitoring.

---

*Specifications current as of April 2026 | For deployment assistance, contact the DialPhone infrastructure team*
