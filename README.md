```markdown
# IRIS

**Internal Reconnaissance & Infrastructure Security**

> See the inside. Understand the exposure. Track the change.

IRIS is a local-first **Internal Attack Surface Management (IASM)** platform for authorized security assessments of LAN, Windows, and Active Directory environments.

It is the internal counterpart to an external attack surface platform such as [SCOPEX](#relationship-with-scopex).

IRIS turns internal network and directory observations into a structured inventory of hosts, services, Windows infrastructure, Active Directory objects, certificates, evidence, and historical changes.

It is designed for **visibility and evidence**, not automatic exploitation.

---

## Authorized Use Only

IRIS is intended **only** for:

- Networks you own
- Lab environments
- CTFs
- Authorized penetration tests
- Authorized internal security assessments

**Never scan a network without explicit written authorization.**

IRIS enforces strict scope controls, including explicit CIDR allowlists, before any active network discovery.

The core platform intentionally avoids:

- Uncontrolled scanning
- Stealth / evasion
- Credential theft
- Persistence
- Automatic exploitation

---

## Why IRIS?

External attack surface management focuses on what an organization exposes to the internet.

Internal environments contain a different class of security information:

```
Hosts
  ↓
Services
  ↓
Windows infrastructure
  ↓
Active Directory
  ↓
Identities
  ↓
Groups
  ↓
Trusts
  ↓
Certificates
  ↓
Relationships
```

A useful internal security platform must understand more than:

```
10.10.10.15 → 445/tcp OPEN
```

It should build a picture such as:

```
CORP.LOCAL
     │
     ├── DC01
     │    ├── DNS
     │    ├── LDAP
     │    ├── LDAPS
     │    ├── Kerberos
     │    └── SMB
     │
     ├── DC02
     │
     ├── FILE01
     │    └── SMB
     │
     ├── WEB01
     │    ├── HTTP
     │    └── HTTPS
     │
     └── USER-PC-042
          └── RDP
```

IRIS turns those observations into an understandable internal attack-surface map.

---

## Core Principles

### 1. Evidence First
Every important observation has supporting evidence.

```
Asset:        DC01
Service:      Kerberos
Port:         88/tcp
Evidence:     Observed TCP service + protocol response
Confidence:   HIGH
```

### 2. Confidence Over Guessing
IRIS uses four confidence levels:

```
HIGH
MEDIUM
LOW
UNKNOWN
```

Direct observations carry more confidence than inferred information.

### 3. Relationships Matter
Internal security is relationship-driven. IRIS models:

```
Domain
  ↓
Domain Controller
  ↓
Active Directory
  ↓
User
  ↓
Group
  ↓
Computer
  ↓
Service
```

---

## Architecture

IRIS is a native Windows application.

```
                         IRIS
                          │
             ┌────────────┴────────────┐
             │                         │
            CLI                    Dashboard
             │                         │
             └────────────┬────────────┘
                          │
                    Scan Manager
                          │
             ┌────────────┼─────────────┐
             │            │             │
          Network        SMB           AD
          Discovery      LDAP        Kerberos
             │            │             │
             └────────────┼─────────────┘
                          │
                   Asset Normalization
                          │
                    Evidence Engine
                          │
                  Confidence Engine
                          │
                   History / Diff
                          │
                       SQLite
```

---

## Technology Stack

**Core**
- Python 3.11+
- SQLite
- SQLAlchemy
- Pydantic
- asyncio

**CLI**
- Typer
- Rich

**API**
- FastAPI
- Uvicorn

**Network / Protocol Collection**
- Nmap
- DNS libraries
- SMB protocol libraries
- LDAP libraries
- Kerberos-aware collection tooling
- Windows-native networking capabilities

IRIS orchestrates and normalizes existing protocol implementations rather than reinventing them.

---

## Native Windows Design

IRIS runs directly on:

```
Windows 10 / Windows 11
Python 3.11+
PowerShell
VS Code
```

**No Docker.**

The core application does **not** require:

- Docker Desktop
- Kubernetes
- PostgreSQL
- Redis
- Neo4j
- Linux VM
- WSL
- Cloud infrastructure

- Database: `data/iris.db`
- Dashboard: `http://127.0.0.1:8000`

---

## Installation

```powershell
git clone <repository>
cd iris

python -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt

iris doctor
```

Expected output:

```
IRIS SYSTEM CHECK

Python          ✓ 3.11.9
SQLite          ✓
Nmap            ✓
Configuration   ✓
Database        ✓

IRIS is ready.
```

---

## Usage

### Scope Model

IRIS requires explicit scope controls.

```powershell
iris recon 10.10.10.0/24
```

Before scanning, IRIS:

1. Parses the target.
2. Validates the CIDR.
3. Confirms the authorized scope.
4. Creates a scan ID.
5. Applies concurrency and rate limits.
6. Performs discovery.
7. Ensures active operations remain within scope.

### Dry Run

```powershell
iris recon 10.10.10.0/24 --dry-run
```

```
Target scope:
10.10.10.0/24

Hosts that would be scanned:
256 addresses

Network activity:
DISABLED

Dry-run complete.
```

### CLI Commands

```powershell
iris --help
iris doctor
iris recon 10.10.10.0/24
iris hosts
iris services
iris ad
iris findings
iris scans
iris diff 41 42
```

Example output:

```
╭──────────────────────────────────────╮
│                 IRIS                 │
│ Internal Reconnaissance & Security   │
╰──────────────────────────────────────╯

Scope: 10.10.10.0/24
Scan: #42
Status: COMPLETED

Hosts                 87
Open Ports            214
Windows Hosts         63
Domain Controllers     2
SMB Hosts             51
LDAP Hosts              4
Kerberos Hosts          2
RDP Hosts             18

Potential Findings     9
Changes Since Last     6
```

---

## Core Capabilities

### 1. Host Discovery
IP address, MAC address, vendor, hostname, reverse DNS, OS indicators, discovery method, timestamp, confidence.

### 2. Port and Service Discovery
```
HOST: DC01

PORT      STATE     SERVICE

53/tcp    open      DNS
88/tcp    open      Kerberos
389/tcp   open      LDAP
445/tcp   open      SMB
636/tcp   open      LDAPS
3389/tcp  open      RDP
```

Every observation is stored as structured data.

### 3. SMB Intelligence
SMB availability, protocol/version indicators, server identity, domain/workgroup, shares, authentication requirements, anonymous-access indicators where safely testable.

Distinguishes between:
- Not tested
- Not observed
- Confirmed
- Unknown

### 4. Active Directory Intelligence
Where authorized credentials and permissions are available: domains, domain controllers, users, groups, computers, OUs, Group Policy metadata, trusts, service-related identity information.

No automatic exploitation.

### 5. LDAP / LDAPS
Collects authorized information from 389/tcp and 636/tcp.

### 6. Kerberos
Identifies Kerberos infrastructure (88/tcp, 464/tcp) and correlates services with domains and domain controllers.

The objective is visibility, not credential attacks.

### 7. RDP / SSH / WinRM
Identifies remote administration exposure and tracks changes between scans.

```
DC01
├── RDP     3389
└── WinRM   5985

LINUX01
└── SSH     22
```

### 8. Internal DNS
Collects A, AAAA, CNAME, MX, NS, SOA, SRV, TXT records. Focus on infrastructure records such as `_ldap._tcp`, `_kerberos._tcp`, `_gc._tcp`.

### 9. DHCP
Where authorized, identifies DHCP infrastructure and relevant metadata.

### 10. Certificate Services
Identifies internal certificate infrastructure: CAs, AD CS, certificate services, LDAP/RDP/web certificates, expiration, issuer relationships.

Focus initially on **discovery and inventory**, not certificate abuse.

---

## Asset Model

Core asset types:

```
Host
IP Address
MAC Address
Domain
User
Group
Computer
Service
Port
Share
Certificate
Certificate Authority
DNS Record
Network
Finding
Evidence
Relationship
```

Example:

```
DC01
│
├── IP
├── DNS
├── SMB
├── LDAP
├── LDAPS
├── Kerberos
├── RDP
└── Domain Controller
       │
       └── CORP.LOCAL
```

---

## Evidence Model

Each observation contains:

```
asset_id
observation_type
value
source
method
timestamp
confidence
raw_evidence
```

Example:

```json
{
  "asset": "DC01",
  "observation": "domain_controller",
  "value": true,
  "source": "LDAP",
  "confidence": "HIGH"
}
```

---

## Historical Monitoring

```powershell
iris diff 41 42
```

```
INTERNAL NETWORK CHANGES

NEW HOST
10.10.10.77

NEW SERVICE
FILE01:445/tcp

NEW RDP
10.10.10.42:3389

REMOVED HOST
10.10.10.91

CHANGED
DC01 certificate
```

Tracks new assets, removed assets, new ports, removed ports, changed services, changed technologies, changed certificates, DNS changes, AD structure changes.

---

## Dashboard

The local dashboard eventually provides:

**Network Overview**
```
Hosts                  418
Windows Hosts          302
Linux Hosts             76
Domain Controllers       2
Open Services          891
AD Users               842
AD Groups              126
Certificates            43
Potential Findings      17
```

**Infrastructure Map**, **AD View**, **Exposure View**.

---

## Database

SQLite at `data/iris.db`.

Potential tables:

```
scans
networks
hosts
ip_addresses
mac_addresses
ports
services
domains
dns_records
smb_shares
ad_domains
ad_users
ad_groups
ad_computers
ad_ous
ad_trusts
certificates
certificate_authorities
findings
evidence
relationships
scan_changes
```

Managed with SQLAlchemy. The abstraction stays clean enough that PostgreSQL can be added later.

---

## Project Structure

```
iris/
│
├── iris/
│   ├── __init__.py
│   ├── __main__.py
│   │
│   ├── cli/
│   │   ├── main.py
│   │   ├── recon.py
│   │   ├── doctor.py
│   │   ├── hosts.py
│   │   ├── services.py
│   │   ├── ad.py
│   │   └── diff.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── scope.py
│   │   ├── logging.py
│   │   └── exceptions.py
│   │
│   ├── database/
│   │   ├── database.py
│   │   ├── models.py
│   │   └── repositories.py
│   │
│   ├── collectors/
│   │   ├── discovery.py
│   │   ├── nmap.py
│   │   ├── dns.py
│   │   ├── smb.py
│   │   ├── ldap.py
│   │   ├── kerberos.py
│   │   ├── ad.py
│   │   ├── certificates.py
│   │   └── remote_access.py
│   │
│   ├── intelligence/
│   │   ├── confidence.py
│   │   ├── correlation.py
│   │   └── history.py
│   │
│   ├── services/
│   │   ├── scanner.py
│   │   ├── asset_manager.py
│   │   └── scan_manager.py
│   │
│   └── api/
│       ├── main.py
│       ├── routes/
│       └── schemas/
│
├── data/
│   └── iris.db
├── logs/
├── tests/
├── docs/
├── frontend/
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
└── LICENSE
```

---

## Development Roadmap

| Phase | Focus | Target |
|---|---|---|
| **1** | Native Windows CLI, scope enforcement, SQLite, scan management, logging, `iris doctor`, basic host + port discovery | 1–2 weeks |
| **2** | Service detection, OS fingerprinting, DNS, reverse DNS, MAC/vendor, infrastructure relationships |  |
| **3** | SMB discovery, share enumeration (authorized), Windows host info, remote-access service detection |  |
| **4** | Domain discovery, domain controllers, LDAP, Kerberos, users, groups, computers, OUs, trusts |  |
| **5** | AD CS discovery, CAs, certificate relationships, internal DNS, DHCP, TLS/certificate monitoring |  |
| **6** | Confidence engine, evidence model, scan comparison, change detection, asset history |  |
| **7** | Dashboard, network overview, host inventory, service inventory, AD explorer, certificate view, findings, historical changes, relationship visualization |  |

---

## What IRIS Should NOT Do Initially

IRIS is **not** an automated exploitation framework.

Avoid building automatic:

- Credential attacks
- Password spraying
- Kerberoasting execution
- NTLM relay
- AD exploitation
- Privilege escalation
- Persistence
- Lateral movement
- Credential dumping

The platform answers:

> What exists inside this environment, how is it connected, what is exposed, and what changed?

---

## Future Security Analysis

Once inventory is mature, IRIS can add an analysis layer that identifies relationships requiring security review.

Example:

```
User A
  ↓
Member of Group B
  ↓
Group B has access to Server C
  ↓
Server C exposes SMB
```

The platform clearly distinguishes **observed fact** from **security interpretation**.

---

## Testing

Tests cover:

- CIDR validation
- Scope enforcement
- Host parsing
- Nmap XML parsing
- DNS parsing
- SMB parsing
- LDAP parsing
- AD object normalization
- Kerberos service detection
- Certificate parsing
- Confidence scoring
- Relationship creation
- Historical diff
- Database operations
- API endpoints

Mocks are used wherever possible. The test suite does **not** depend on a live corporate network.

```powershell
pytest
```

---

## Relationship with SCOPEX

IRIS and SCOPEX share a common security-intelligence philosophy.

```
             SECURITY SURFACE PLATFORM
                       │
          ┌────────────┴────────────┐
          │                         │
        SCOPEX                     IRIS
       External                  Internal
          │                         │
       Internet                    LAN
          │                         │
        DNS/HTTP                 SMB/LDAP
        TLS/ASN                  AD/Kerberos
        Services                 DNS/DHCP
          │                         │
          └────────────┬────────────┘
                       │
                Common Intelligence
                       │
                Evidence + History
```

Both systems remain independently useful.

---

## Final Vision

A mature IRIS installation lets a security professional immediately understand:

- What hosts exist?
- Which ones are Windows?
- Where are the domain controllers?
- What services are exposed?
- Where is SMB exposed?
- Where are LDAP and Kerberos?
- What remote administration services exist?
- What does Active Directory look like?
- What certificate infrastructure exists?
- What changed since the previous assessment?
- What evidence supports each observation?
- What requires further security review?

The end goal is a **living, evidence-backed map of an organization's internal infrastructure**.

---


## Disclaimer

IRIS is a security tool intended for **authorized use only**. The authors assume no liability for misuse, unauthorized scanning, or damage caused by this software. You are responsible for ensuring you have explicit permission before running IRIS against any network or system.

> **Map the inside. Understand the exposure. Track the change.**
```
