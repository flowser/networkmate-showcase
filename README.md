<h1 align="center">NetworkMate — Campus Network Control Room</h1>

<p align="center">
  A web control room for school and campus networks: identity Wi-Fi, device control, and one-click exam lockdown on MikroTik and FreeRADIUS.<br/>
  Designed and built by <a href="https://github.com/flowser"><b>Eng. Felix Nyachio</b></a>, Savvytex Marines Ltd.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-In_production-22c55e?style=flat-square" alt="In production"/>
  <img src="https://img.shields.io/badge/Django_REST-092E20?style=flat-square&logo=django&logoColor=white" alt="Django"/>
  <img src="https://img.shields.io/badge/Vue_3-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white" alt="Vue 3"/>
  <img src="https://img.shields.io/badge/MikroTik-293239?style=flat-square&logo=mikrotik&logoColor=white" alt="MikroTik"/>
  <img src="https://img.shields.io/badge/FreeRADIUS-1f2937?style=flat-square" alt="FreeRADIUS"/>
  <img src="https://img.shields.io/badge/UniFi-0559C9?style=flat-square&logo=ubiquiti&logoColor=white" alt="UniFi"/>
  <img src="https://img.shields.io/badge/Proxmox-E57000?style=flat-square&logo=proxmox&logoColor=white" alt="Proxmox"/>
  <img src="https://img.shields.io/badge/CI-GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white" alt="GitHub Actions"/>
</p>

> **This is a showcase repository.** The source code is private and in production use. A live walkthrough is available on request — [get in touch](#contact).

---

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

---

## Architecture

```mermaid
flowchart LR
  UI["Vue 3 control room"] -->|REST| API["Django REST API"]
  API --> DB[("Database")]
  API -->|RouterOS API| GW["MikroTik gateway<br/>dual-WAN · exam mode"]
  API --> RAD["FreeRADIUS"]
  RAD --> AP["UniFi access points"]
  GW --> DEV["Student and staff devices"]
  AP --> DEV
```

<p align="center">
  <img src="assets/campus-server-stack.png" width="80%" alt="Campus server stack on Proxmox"/>
  <br/><sub>Production deployment: dual-WAN uplinks and campus services on one Proxmox host</sub>
</p>

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

---

## Contact

Need your campus or office network brought under control?

<p>
  <a href="mailto:eng.felixnyachio@gmail.com?subject=NetworkMate%20inquiry"><img src="https://img.shields.io/badge/Email-eng.felixnyachio%40gmail.com-ef4444?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/></a>
  <a href="https://wa.me/254748650011?text=Hi%20Felix%2C%20I%20saw%20NetworkMate%20on%20GitHub."><img src="https://img.shields.io/badge/WhatsApp-%2B254_748_650_011-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" alt="WhatsApp"/></a>
  <a href="https://github.com/flowser"><img src="https://img.shields.io/badge/Profile-flowser-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub profile"/></a>
</p>

<sub>© 2026 Savvytex Marines Ltd. All rights reserved. Diagrams may not be reused without permission.</sub>
