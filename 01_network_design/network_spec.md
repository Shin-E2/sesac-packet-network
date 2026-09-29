# Network Specification

## 1. Network Overview

Packet Tracer에서 VLAN 10과 VLAN 20을 분리하고, L2 Switch 2대와 Multilayer Switch(MLS) 1대를 사용해 Inter-VLAN Routing이 가능하도록 구성했다.

- VLAN 10: `DEV_TEAM`
- VLAN 20: `OPS_TEAM`
- VLAN 10 Gateway: `192.168.10.1`
- VLAN 20 Gateway: `192.168.20.1`
- L2 Switch: `SW1`, `SW2`
- Multilayer Switch: `MLS1`
- Server: DNS / Web Server

## 2. VLAN / Subnet

| VLAN ID | VLAN Name | Network | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| 10 | DEV_TEAM | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.1` |
| 20 | OPS_TEAM | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.1` |

## 3. End Device IP

| Device | VLAN | IP Address | Subnet Mask | Default Gateway | DNS Server |
|---|---:|---|---|---|---|
| PC1 | 10 | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` | `192.168.20.20` |
| PC2 | 10 | `192.168.10.11` | `255.255.255.0` | `192.168.10.1` | `192.168.20.20` |
| PC3 | 20 | `192.168.20.10` | `255.255.255.0` | `192.168.20.1` | `192.168.20.20` |
| PC4 | 20 | `192.168.20.11` | `255.255.255.0` | `192.168.20.1` | `192.168.20.20` |
| Server | 20 | `192.168.20.20` | `255.255.255.0` | `192.168.20.1` | `192.168.20.20` |

## 4. Physical Port Mapping

### SW1

| SW1 Port | Connected Device | Device Port | Mode | VLAN |
|---|---|---|---|---|
| `Fa0/1` | PC1 | `Fa0` | Access | 10 |
| `Fa0/2` | PC2 | `Fa0` | Access | 10 |
| `Gi0/1` | MLS1 | `Gi0/1` | Trunk | 10, 20 |

### SW2

| SW2 Port | Connected Device | Device Port | Mode | VLAN |
|---|---|---|---|---|
| `Fa0/1` | PC3 | `Fa0` | Access | 20 |
| `Fa0/2` | PC4 | `Fa0` | Access | 20 |
| `Fa0/3` | Server | `Fa0` | Access | 20 |
| `Gi0/1` | MLS1 | `Gi0/2` | Trunk | 10, 20 |

### MLS1

| MLS1 Port / Interface | Connected Device / Role | Configuration |
|---|---|---|
| `Gi0/1` | SW1 `Gi0/1` | Trunk, VLAN 10/20 허용 |
| `Gi0/2` | SW2 `Gi0/1` | Trunk, VLAN 10/20 허용 |
| `Vlan10` | VLAN 10 Gateway | `192.168.10.1/24` |
| `Vlan20` | VLAN 20 Gateway | `192.168.20.1/24` |

MLS1에서 `ip routing`을 활성화해 VLAN 10과 VLAN 20 사이의 Inter-VLAN Routing을 수행한다.

## 5. Topology

```text
PC1 (192.168.10.10) ── SW1 Fa0/1
PC2 (192.168.10.11) ── SW1 Fa0/2

SW1 Gi0/1 ── Trunk VLAN 10,20 ── MLS1 Gi0/1
MLS1 Gi0/2 ── Trunk VLAN 10,20 ── SW2 Gi0/1

PC3 (192.168.20.10) ── SW2 Fa0/1
PC4 (192.168.20.11) ── SW2 Fa0/2
Server (192.168.20.20) ── SW2 Fa0/3
```

## 6. Baseline Verification

| Test | Purpose |
|---|---|
| PC1 → PC2 | VLAN 10 내부 통신 확인 |
| PC3 → PC4 | VLAN 20 내부 통신 확인 |
| PC3 → Server | VLAN 20 내부 Server 통신 확인 |
| PC1 → PC3 | Inter-VLAN Routing 확인 |
| PC1 → Server | VLAN 10 → VLAN 20 Server 통신 확인 |

```text
PC1> ping 192.168.10.11
PC3> ping 192.168.20.11
PC3> ping 192.168.20.20
PC1> ping 192.168.20.10
PC1> ping 192.168.20.20
```

## 7. Packet Analyst Baseline Values

```text
VLAN10 Gateway = 192.168.10.1
VLAN20 Gateway = 192.168.20.1
Server IP      = 192.168.20.20
```

장애 상황에서 `192.168.10.254` 같은 Gateway가 관찰되면 정상 설계값이 아니라 오설정된 값으로 판단한다.

## 8. DNS / Web Test Values

| Item | Value |
|---|---|
| DNS Server | `192.168.20.20` |
| Test Domain | `www.packetlab.test` |
| Web Protocol | `HTTP` |
| Web Port | `TCP/80` |

> `www.packetlab.test`와 TCP/80은 팀 협업을 위한 테스트 기준값이다.
