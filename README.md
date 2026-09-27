<picture>
  <source media="(prefers-color-scheme: light)" srcset="assets/theme/hero-light.svg"/>
  <img width="100%" src="assets/theme/hero-dark.svg" alt="NetworkMate. Campus network control room."/>
</picture>

<p align="center">
  A web control room for school and campus networks: identity Wi-Fi, device control, and one-click exam lockdown on MikroTik and FreeRADIUS.<br/>
  Designed and built by <a href="https://github.com/flowser"><b>Eng. Felix Nyachio</b></a>, Savvytex Marines Ltd.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-In_production-34d399?style=flat-square&labelColor=16162a" alt="In production"/>
  <img src="https://img.shields.io/badge/Django_REST-092E20?style=flat-square&logo=django&logoColor=white" alt="Django"/>
  <img src="https://img.shields.io/badge/Vue_3-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white" alt="Vue 3"/>
  <img src="https://img.shields.io/badge/MikroTik-293239?style=flat-square&logo=mikrotik&logoColor=white" alt="MikroTik"/>
  <img src="https://img.shields.io/badge/FreeRADIUS-1f2937?style=flat-square" alt="FreeRADIUS"/>
  <img src="https://img.shields.io/badge/UniFi-0559C9?style=flat-square&logo=ubiquiti&logoColor=white" alt="UniFi"/>
  <img src="https://img.shields.io/badge/Proxmox-E57000?style=flat-square&logo=proxmox&logoColor=white" alt="Proxmox"/>
  <img src="https://img.shields.io/badge/CI-GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white" alt="GitHub Actions"/>
</p>

> **This is a showcase repository.** The source code is private and in production use. A live walkthrough is available on request — [get in touch](#contact).

<p align="center"><img width="100%" src="assets/theme/divider.svg" alt=""/></p>

## The problem

On most campuses the Wi-Fi password is shared, nobody knows which device belongs to whom, and during exams the only way to stop students going online is to switch the internet off for everyone, including the principal's office.

## What NetworkMate does

| Capability | Details |
|---|---|
| **Identity Wi-Fi** | Every student signs in with their own credentials (imported from the college roster) through FreeRADIUS and UniFi. No shared passwords. |
| **Device control** | Detects connected devices, links them to students, flags unauthorised hardware, and applies per-user device limits. |
| **Exam mode** | One switch blocks student internet, keeps approved exam servers reachable, enforces device limits, and leaves staff online. |
| **Separate lanes** | Student and staff networks run on separate paths, so lockdown never silences administration. |
| **Session tracking** | Logs who connected, when, on which device, and for how long. |
| **Router integration** | Talks to MikroTik RouterOS/CHR to monitor health and push policy, so IT clicks a screen instead of editing router configs. |

<p align="center">
  <img src="assets/dual-lan-split.png" width="85%" alt="Student and staff network separation"/>
  <br/><sub>Student and staff lanes through one campus gateway</sub>
</p>

<p align="center"><img width="100%" src="assets/theme/divider.svg" alt=""/></p>

## Architecture

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontSize':'15px','primaryColor':'#16162a','primaryTextColor':'#f0f0ff','primaryBorderColor':'#4f46e5','lineColor':'#0ea5e9','clusterBkg':'#0d0d1a','clusterBorder':'#4f46e5','titleColor':'#0ea5e9','edgeLabelBackground':'#16162a'}}}%%
flowchart LR
  UI["🖥️ Vue 3 control room"]:::staff
  API{{"⚙️ Django REST API"}}:::core
  DB[("🗄️ Database")]:::data
  AE["🏫 AEMMS<br/>people and rooms"]:::ai
  GW["🧭 MikroTik gateway<br/>dual-WAN · exam mode"]:::net
  RAD["🔐 FreeRADIUS"]:::net
  AP["📶 UniFi access points"]:::net
  DEV["🎓 Student and staff devices"]:::trainee
  UI -->|REST| API
  API <--> DB
  AE -.->|Wi-Fi identities| API
  API ==>|RouterOS API| GW
  API --> RAD
  RAD --> AP
  GW --> DEV
  AP --> DEV
  classDef ai fill:#9333ea,stroke:#d8b4fe,stroke-width:2px,color:#ffffff
  classDef core fill:#ea580c,stroke:#fdba74,stroke-width:3px,color:#ffffff
  classDef data fill:#1e1b4b,stroke:#818cf8,stroke-width:2px,color:#e0e7ff
  classDef net fill:#0d9488,stroke:#5eead4,stroke-width:2px,color:#ffffff
  classDef staff fill:#4f46e5,stroke:#a5b4fc,stroke-width:2px,color:#ffffff
  classDef trainee fill:#0284c7,stroke:#7dd3fc,stroke-width:2px,color:#ffffff
  linkStyle default stroke:#0ea5e9,stroke-width:2px
```

<p align="center">
  <img src="assets/campus-server-stack.png" width="80%" alt="Campus server stack on Proxmox"/>
  <br/><sub>Production deployment: dual-WAN uplinks and campus services on one Proxmox host</sub>
</p>

### Exam mode, step by step

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontSize':'15px','actorBkg':'#4f46e5','actorBorder':'#a5b4fc','actorTextColor':'#ffffff','actorLineColor':'#6b6b8a','signalColor':'#0ea5e9','signalTextColor':'#0ea5e9','labelBoxBkgColor':'#f97316','labelBoxBorderColor':'#fdba74','labelTextColor':'#ffffff','loopTextColor':'#f97316','noteBkgColor':'#16162a','noteTextColor':'#f0f0ff','noteBorderColor':'#f97316','activationBkgColor':'#0ea5e9','activationBorderColor':'#7dd3fc','sequenceNumberColor':'#ffffff'}}}%%
sequenceDiagram
  autonumber
  participant IT as 🧑‍💻 IT officer
  participant NM as 🛡️ NetworkMate
  participant GW as 🧭 MikroTik gateway
  participant ST as 🎓 Student devices
  participant SF as 🧑‍💼 Staff devices
  rect rgba(234, 88, 12, 0.16)
    Note over IT,GW: Exam starts
    IT->>NM: Turn exam mode on
    NM->>GW: Enable exam firewall rules (RouterOS API)
    NM->>GW: Allow each student's primary device only
    GW-->>ST: Internet blocked, exam hosts allowed
    GW-->>SF: No change, staff stay online
  end
  rect rgba(5, 150, 105, 0.16)
    Note over IT,GW: Exam ends
    IT->>NM: Turn exam mode off
    NM->>GW: Disable exam firewall rules
    GW-->>ST: Normal access restored
  end
```

## Engineering highlights

- **Layered backend:** Django REST Framework organised as controllers → services → repositories → models, with serializers at the edge.
- **Network as code, behind a UI:** router and RADIUS changes are driven from the API, so exam-mode transitions are repeatable and audited.
- **Runs anywhere:** Docker Compose locally; Proxmox on campus; GitHub Actions CI.
- **Built for handover:** operations runbooks so the client's IT team can run it without the vendor.

## Stack

| Layer | Technologies |
|---|---|
| Backend | Python, Django REST Framework |
| Frontend | Vue 3, Vite, TypeScript |
| Network | MikroTik RouterOS / CHR, FreeRADIUS, UniFi OS, BIND |
| Data | MySQL / PostgreSQL |
| Infrastructure | Docker Compose, Nginx, Proxmox VE, GitHub Actions |

## In production

Deployed at a teacher-training college in Kenya as part of a full campus build: dual-WAN (Starlink + Airtel), identity Wi-Fi at roster scale, exam lockdown, and self-hosted services. Read the [campus network case study](https://github.com/flowser/flowser/blob/main/case-studies/chesta-campus-network.md).

<p align="center"><img width="100%" src="assets/theme/divider.svg" alt=""/></p>

## Contact

Need your campus or office network brought under control?

<p>
  <a href="mailto:eng.felixnyachio@gmail.com?subject=NetworkMate%20inquiry"><img src="https://img.shields.io/badge/Email-eng.felixnyachio%40gmail.com-ef4444?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/></a>
  <a href="https://wa.me/254748650011?text=Hi%20Felix%2C%20I%20saw%20NetworkMate%20on%20GitHub."><img src="https://img.shields.io/badge/WhatsApp-%2B254_748_650_011-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" alt="WhatsApp"/></a>
  <a href="https://github.com/flowser"><img src="https://img.shields.io/badge/Profile-flowser-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub profile"/></a>
</p>

<sub>© 2026 Savvytex Marines Ltd. All rights reserved. Diagrams may not be reused without permission.</sub>
