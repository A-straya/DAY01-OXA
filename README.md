
<p align="center">
  <img src="assets/day01.svg" alt="OXA — Day 01 Build" width="900">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/STATUS-INFRA%20ONLINE-00ffb0?style=for-the-badge&labelColor=011014">
  <img src="https://img.shields.io/badge/DOMAIN-oxa.local-00c8ff?style=for-the-badge&labelColor=011014">
  <img src="https://img.shields.io/badge/DAY-01%20%2F%2003-0078d4?style=for-the-badge&labelColor=011014">
  <img src="https://img.shields.io/badge/THREAT-UNPATCHED-ff3b3b?style=for-the-badge&labelColor=011014">
</p>

<h3 align="center">

```
root@oxa:~# cat day01/mission.log
```

</h3>

<p align="center"><i>Stand up the target. Wire the network. Light up the services.<br>
Day 2 breaks it on purpose. Day 3 we go hunting.</i></p>

<br>

```diff
+ [BOOT]  OXA-DC01 .......................... ONLINE
+ [BOOT]  WIN10-CLIENT ...................... ONLINE
+ [BOOT]  KALI-ATK01 ........................ STANDBY (Day 03)
! [NOTE]  Hardening: NOT YET APPLIED (intentional — see day02)
```

---

## `0x00` — Briefing

**Company OXA** just handed us a blank slate: one virgin Windows Server, one Windows 10 client, and an empty GNS3 canvas.
Day 01's job is to turn that into a believable small-business network — real AD domain, real services, real traffic —
so that on **Day 03** our Kali box has something worth attacking.

| | |
|---|---|
|  **Build target** | Company OXA — `oxa.local` |
|  **Nodes** | Windows Server (DC) · Windows 10 (client) · Kali (standby) |
|  **Platform** | GNS3 + VMware Workstation |
|  **Day 01 goal** | AD DS, DNS, DHCP, file share, IIS, FTP — all live, all joined |
|  **Next** | Day 02 strips the hardening on purpose |

---

## `0x01` — Topology

```
                     ┌──────────────────┐
                     │   Kali  Linux     │   (standby — Day 03)
                     │  192.168.10.30    │
                     └─────────┬─────────┘
                               │
                      ┌────────┴────────┐
                      │   Ethernet SW   │
                      └───┬─────────┬───┘
             ┌────────────┘         └────────────┐
   ┌─────────┴─────────┐              ┌──────────┴──────────┐
   │  OXA-DC01          │              │  WIN10-CLIENT        │
   │  Windows Server    │◄────AD──────►│  Windows 10           │
   │  192.168.10.10/24  │   join/auth  │  192.168.10.20/24     │
   │  DC · DNS · DHCP   │              │  (or DHCP leased)     │
   │  IIS · SMB · FTP   │              │                        │
   └────────────────────┘              └────────────────────────┘
```

>  Isolated lab network only. Never bridge this to a real LAN — it's going to be
> *deliberately* vulnerable by Day 02.

<p align="center">
  <a href="https://YOUR-USERNAME.github.io/oxa-pentesting-hardening/topology/oxa_topology.html">
    <img src="https://img.shields.io/badge/▶%20OPEN-LIVE%20INTERACTIVE%20TOPOLOGY-00ffb0?style=for-the-badge&labelColor=011014">
  </a>
</p>

<p align="center"><sub>Click any node — switch, server, client, or Kali — to see its live IP, status, and running services. Green/cyan pulses trace real AD, DNS, DHCP and client traffic across the wire.<br>
Self-contained file, no backend — works locally (just open the .html) or hosted via GitHub Pages. See setup note below.</sub></p>

---

## `0x02` — Build log

> Tick these off as you go. Full step-by-step is in `docs/OXA_Day1_Guide.pdf`.

### GNS3 + hypervisor
- [ ] GNS3 installed, VMware Workstation installed, `vmrun` resolvable from PATH
- [ ] Three VM templates imported: `OXA-DC01`, `WIN10-CLIENT`, `KALI-ATK01`
- [ ] Topology wired: Switch ↔ Server, Switch ↔ Client (Kali attached, powered off)

### `OXA-DC01` — Windows Server
- [ ] Hostname set → `OXA-DC01`
- [ ] Static IP → `192.168.10.10 /24`, DNS → `127.0.0.1`
- [ ] AD DS role installed → forest `oxa.local` created → server promoted
- [ ] OUs created: `Employees`, `IT`, `Servers`
- [ ] Users created: `j.martin` (standard), `s.admin` (Domain Admins)
- [ ] Group created: `GRP-Finance`
- [ ] DHCP role installed → scope `192.168.10.100–200` → activated
- [ ] SMB share live → `\\OXA-DC01\Finance`
- [ ] IIS installed → default site reachable at `http://192.168.10.10`
- [ ] FTP site created (no SSL — intentional)
- [ ] 📸 Snapshot taken: `00-clean-install`

### `WIN10-CLIENT` — Windows 10
- [ ] DNS points at `192.168.10.10` (static or via DHCP)
- [ ] Joined domain `oxa.local`
- [ ] Logged in as `OXA\j.martin` successfully
- [ ] Computer object visible in ADUC on the DC

---

## `0x03` — Verification

```powershell
# On OXA-DC01
Get-ADDomain
Get-Service DNS, NTDS, Netlogon, DHCPServer

# On WIN10-CLIENT
Get-ComputerInfo | Select CsDomain
```

```
[OK]  oxa.local .............................. resolvable
[OK]  \\OXA-DC01\Finance ..................... reachable
[OK]  http://192.168.10.10 ................... 200 OK
[OK]  WIN10-CLIENT joined to OXA ............. confirmed
```

---

## `0x04` — Roadmap

| Day | Codename | What happens |
|:---:|---|---|
| **01** | `BUILD` | ✅ Stand up OXA: AD, DNS, DHCP, IIS, SMB, FTP, domain-joined client |
| 02 | `HARDEN⁻¹` | Strip patches, loosen Defender/firewall, plant weak creds + SMBv1, snapshot the vulnerable state |
| 03 | `BREACH` | Kali recon (Nmap) → vuln scan → Metasploit exploitation → lateral movement → write-up |

---

<details>
<summary><b>⚙ Hosting the live topology yourself (GitHub Pages)</b></summary>

<br>

The diagram is a single self-contained `oxa_topology.html` — no server, no dependencies, nothing tied to any external service.

1. Put the file at `topology/oxa_topology.html` in this repo (matches the link above — adjust the path if you place it elsewhere).
2. Repo → **Settings** → **Pages** → under "Build and deployment", set Source to **Deploy from a branch**, branch `main`, folder `/ (root)` → Save.
3. GitHub gives you a URL like `https://YOUR-USERNAME.github.io/oxa-pentesting-hardening/`. Swap that into the badge link above (append `topology/oxa_topology.html`).
4. Or skip Pages entirely — just double-click the file locally, it opens and works fully offline in any browser.

</details>

<p align="center">
<sub>Internal lab repo · Company OXA is a fictional pentest/hardening training target · Not a real organization</sub>
</p>
