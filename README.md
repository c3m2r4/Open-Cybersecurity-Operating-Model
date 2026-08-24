# Open Cybersecurity Operating Model

An open-source cybersecurity operating model connecting business risk, information security, security controls, offensive security, defensive security, purple teaming, architecture, DevSecOps, incident response, and executive decision-making.

## 🎯 Purpose

This repository is a **comprehensive knowledge vault and practical playbook** for cybersecurity professionals and organizations seeking to:

- **Align security** with business risk and governance
- **Understand controls** across all security domains
- **Validate security** through technical assessment and offensive testing
- **Detect threats** through operational monitoring
- **Remediate systematically** with strategic improvements

**Core Philosophy:**
> Understand the risk. Understand the control. Understand how it can fail. Validate it technically. Detect it operationally. Fix it strategically.

## 📚 What's Inside

### Knowledge Domains (Numbered 00–12)

| Section | Focus | Purpose |
|---------|-------|---------|
| **00 - START HERE** | Navigation & Dashboard | Entry point to the vault |
| **01 - BUSINESS & GOVERNANCE** | Risk governance, executive frameworks | Connect security to business objectives |
| **02 - IT RISK MANAGEMENT** | Risk identification, assessment, reporting | Quantify and track security risk |
| **03 - INFORMATION SECURITY** | Policies, classification, data governance | Define security requirements |
| **04 - SECURITY CONTROLS** | Control frameworks (NIST, ISO, CIS) | Map and validate controls |
| **05 - OFFENSIVE SECURITY** | Penetration testing, attack techniques | Understand threat actor methods |
| **06 - DEFENSIVE SECURITY** | Detection, monitoring, SOC operations | Defend and respond |
| **07 - PURPLE TEAM** | Adversary simulation, red/blue integration | Continuous security validation |
| **08 - SECURITY ARCHITECTURE** | System design, zero-trust, segmentation | Build secure infrastructure |
| **09 - APPLICATION & DEVSECOPS** | Secure SDLC, vulnerability management | Secure the development lifecycle |
| **10 - INCIDENT & RESILIENCE** | Incident response, recovery playbooks | Prepare for and recover from incidents |
| **11 - SECURITY PROJECTS** | Lab exercises & reference implementations | Hands-on learning |
| **12 - MANAGEMENT & BOARD** | Metrics, reporting, communication | Executive visibility |
| **99 - TEMPLATES** | Reusable templates for documentation | Accelerate security processes |

### Practical Labs & Runbooks

- **GOAD** (Game of Active Directory) — Intentionally vulnerable Active Directory lab for offensive testing and detection validation
- **Mythic C2** — Command & Control infrastructure for red team operations
- **Operation Cypher-Knife** — Complete offensive security runbook covering the full attack chain from reconnaissance through persistence and cleanup

## 🚀 Getting Started

### Prerequisites

- **Obsidian** (free) — Download from [obsidian.md](https://obsidian.md) to open this vault with full linking and graph features
- **Git** — To clone this repository
- **For labs:** Docker, Vagrant, or hypervisor (VirtualBox, Hyper-V, VMware)

### Quick Start

1. **Clone the repository:**
   ```bash
   git clone https://github.com/c3m2r4/Open-Cybersecurity-Operating-Model.git
   cd Open-Cybersecurity-Operating-Model
   ```

2. **Open in Obsidian:**
   - Launch Obsidian
   - Click "Open folder as vault"
   - Select the cloned directory
   - Start with `Welcome.md` → Dashboard

3. **Explore the vault:**
   - Use the sidebar to navigate numbered sections
   - Use the graph view to see topic connections
   - Search for security concepts across all domains

### Running the Labs

#### GOAD (Game of Active Directory)
- **Entry point:** `🏰 GOAD — GAME OF ACTIVE DIRECTORY — FULL ASSAULT PLAYBOOK.md`
- **Time to deploy:** 30–60 minutes
- **Requirements:** 
  - 4+ cores, 16GB+ RAM
  - Virtualization (VirtualBox recommended)
  - Vagrant + Ansible

#### Mythic C2
- **Entry point:** `🎯 MYTHIC C2 — COMPLETE OPERATION CYPHER-KNIFE RUNBOOK.md`
- **Time to deploy:** 20–30 minutes
- **Requirements:**
  - Docker + Docker Compose
  - Linux or WSL2

#### Operation Cypher-Knife
- **Entry point:** `OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md`
- **Phases:** 0–14 (prep through cleanup)
- **Full chain:** Foothold → privilege escalation → lateral movement → persistence → exfiltration
- **Requirements:** Access to GOAD lab or similar target

## 📖 How to Use This Knowledge Base

### For Security Architects & Leaders
1. Start with **01 - BUSINESS & GOVERNANCE**
2. Review **02 - IT RISK MANAGEMENT** to quantify risk
3. Use **04 - SECURITY CONTROLS** to map frameworks
4. Reference **12 - MANAGEMENT & BOARD** for reporting

### For Offensive Security Professionals
1. Review **05 - OFFENSIVE SECURITY** for techniques
2. Study **Operation Cypher-Knife** for complete attack chains
3. Use **GOAD** and **Mythic C2** labs to practice
4. Cross-reference with **04 - SECURITY CONTROLS** to understand what you're testing

### For Defensive Security & SOC Teams
1. Study **06 - DEFENSIVE SECURITY** for detection strategies
2. Use **04 - SECURITY CONTROLS** to understand what to monitor
3. Reference **10 - INCIDENT & RESILIENCE** for response playbooks
4. Review **07 - PURPLE TEAM** for validation exercises

### For DevSecOps & Application Security
1. Start with **09 - APPLICATION & DEVSECOPS**
2. Reference **04 - SECURITY CONTROLS** for relevant controls
3. Use **templates** in **99 - TEMPLATES** for documentation

### For Learning & Training
1. **Beginners:** Follow 00 → 01 → 02 → 03 → 04 sequentially
2. **Hands-on practice:** Deploy **GOAD** lab, run through techniques in **05**
3. **Advanced:** Study **Operation Cypher-Knife** full runbook and customize for your environment

## 🏗️ Repository Structure

```
.obsidian/                          Obsidian vault configuration

00 - START HERE/                    Entry point & navigation
01 - BUSINESS & GOVERNANCE/         Risk & governance
02 - IT RISK MANAGEMENT/            Risk assessment & metrics
03 - INFORMATION SECURITY/          Security policies & standards
04 - SECURITY CONTROLS/             Control mappings & frameworks
05 - OFFENSIVE SECURITY/            Attack techniques & methodologies
06 - DEFENSIVE SECURITY/            Detection & response
07 - PURPLE TEAM/                   Red/blue integration & validation
08 - SECURITY ARCHITECTURE/         Infrastructure & design
09 - APPLICATION & DEVSECOPS/       Secure development & deployment
10 - INCIDENT & RESILIENCE/         Incident response & recovery
11 - SECURITY PROJECTS/             Lab exercises & implementations
12 - MANAGEMENT & BOARD/            Metrics & executive reporting
99 - TEMPLATES/                     Reusable templates

GOAD/                               Game of Active Directory lab files
Mythic/                             Mythic C2 setup & configuration
Operation Cypher-Knife/             Offensive security resources

Welcome.md                          Vault entry point
OPERATION CYPHER-KNIFE — 
  COMPLETE RUNBOOK.md               Full offensive security playbook
🎯 MYTHIC C2 — COMPLETE 
  OPERATION CYPHER-KNIFE RUNBOOK.md Mythic C2 deployment guide
🏰 GOAD — GAME OF ACTIVE 
  DIRECTORY — FULL ASSAULT 
  PLAYBOOK.md                       GOAD lab deployment guide

LICENSE                             MIT License
README.md                           This file
```

## 🔗 Interconnected Knowledge Model

This vault uses **Obsidian linking** to connect concepts across domains:

- **Business risk** informs **security controls**
- **Security controls** are validated through **offensive security** testing
- **Offensive techniques** inform **defensive detection** rules
- **Incident response** playbooks feed back into **architecture** improvements
- **Metrics** in management reporting track progress across all domains

Use Obsidian's **graph view** (Ctrl/Cmd + Shift + G) to visualize these connections.

## 📋 Key Files & Entry Points

| File | Purpose |
|------|---------|
| `Welcome.md` | Start here — vault overview & philosophy |
| `OPERATION CYPHER-KNIFE — COMPLETE RUNBOOK.md` | Full offensive security kill chain (80K+ lines) |
| `🏰 GOAD — GAME OF ACTIVE DIRECTORY — FULL ASSAULT PLAYBOOK.md` | GOAD lab setup & exploitation guide |
| `🎯 MYTHIC C2 — COMPLETE OPERATION CYPHER-KNIFE RUNBOOK.md` | Mythic C2 deployment & operations |
| `192.168.56.0 24 is the standard FULL GOAD network.md` | GOAD network reference |

## 🛠️ Contributing

This is an open-source project. Contributions are welcome:

1. **Fork** the repository
2. **Create a branch** for your changes
3. **Submit a pull request** with:
   - Clear description of changes
   - References to relevant sections
   - Links to updated/new topics

### Contribution Ideas
- Additional control frameworks or mappings
- New offensive/defensive techniques
- Lab enhancements or variations
- Translation to other languages
- Visual diagrams or flow charts
- Additional runbooks for specific scenarios

## 📄 License

MIT License — See [LICENSE](LICENSE) file for details.

You are free to use, modify, and distribute this material for personal, commercial, and educational purposes.

## 🤝 Support & Community

- **Issues:** Report gaps, errors, or missing content via GitHub Issues
- **Discussions:** Share experiences, ask questions, and discuss security topics
- **Wiki:** Extended documentation and community contributions

## 🎓 Learning Paths

### Path 1: Security Fundamentals (Beginner)
```
Welcome.md 
  → 00 - START HERE 
  → 01 - BUSINESS & GOVERNANCE 
  → 02 - IT RISK MANAGEMENT 
  → 03 - INFORMATION SECURITY 
  → 04 - SECURITY CONTROLS
```
**Time:** 2–4 weeks

### Path 2: Offensive Security (Intermediate)
```
04 - SECURITY CONTROLS 
  → 05 - OFFENSIVE SECURITY 
  → GOAD Lab (deploy & practice) 
  → Operation Cypher-Knife (study phases 0–5)
```
**Time:** 3–6 weeks

### Path 3: Defensive Security & Detection (Intermediate)
```
04 - SECURITY CONTROLS 
  → 06 - DEFENSIVE SECURITY 
  → 07 - PURPLE TEAM 
  → 10 - INCIDENT & RESILIENCE 
  → (Run GOAD lab, build detection rules)
```
**Time:** 3–6 weeks

### Path 4: Architecture & Strategy (Advanced)
```
01 - BUSINESS & GOVERNANCE 
  → 02 - IT RISK MANAGEMENT 
  → 08 - SECURITY ARCHITECTURE 
  → 09 - APPLICATION & DEVSECOPS 
  → 12 - MANAGEMENT & BOARD
```
**Time:** 4–8 weeks

### Path 5: Purple Team & Continuous Validation (Advanced)
```
05 - OFFENSIVE SECURITY 
  → 06 - DEFENSIVE SECURITY 
  → 07 - PURPLE TEAM 
  → Mythic C2 (advanced operations) 
  → Operation Cypher-Knife (phases 6–14)
```
**Time:** 6+ weeks

## ❓ FAQ

**Q: Is this for penetration testers only?**
A: No. This is designed for the entire security organization—from executives to SOC analysts to system administrators. Each section is relevant to different roles.

**Q: Can I use the labs in production?**
A: No. GOAD, Mythic, and Cypher-Knife are intentionally vulnerable and designed for isolated lab environments only. Never deploy in production.

**Q: How often is this updated?**
A: This is an active project. Check the repository for the latest updates. Contributions and feedback drive improvements.

**Q: Can I use this for training my team?**
A: Yes. The MIT license allows for educational use. Sections 11 (SECURITY PROJECTS) and 99 (TEMPLATES) are specifically designed for team training.

**Q: Do I need to read everything?**
A: No. Use the learning paths above to select sections relevant to your role and goals.

## 🔒 Security & Responsible Disclosure

This repository contains offensive security techniques and playbooks intended for authorized security testing and training only. Users are responsible for:

- Obtaining proper authorization before any security testing
- Following applicable laws and regulations in your jurisdiction
- Using this material only on systems you own or have explicit permission to test
- Reporting security vulnerabilities responsibly to affected organizations

## 📞 Get in Touch

- **GitHub:** [c3m2r4/Open-Cybersecurity-Operating-Model](https://github.com/c3m2r4/Open-Cybersecurity-Operating-Model)
- **Issues & Feedback:** Use GitHub Issues for bugs, suggestions, or questions

---

**Start exploring:** Open `Welcome.md` in Obsidian or visit the [00 - START HERE](00%20-%20START%20HERE) section to begin.
