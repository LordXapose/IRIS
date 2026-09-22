IRIS

Internal Reconnaissance & Infrastructure Security

See the inside. Understand the exposure. Track the change.

IRIS is a local-first Internal Attack Surface Management (IASM) platform for authorized security assessments of LAN, Windows, and Active Directory environments.

It is designed as the internal counterpart to an external attack-surface platform such as SCOPEX.

IRIS turns internal network and directory observations into a structured inventory of:

Live hosts

Open ports and services

Windows infrastructure

SMB exposure

LDAP and LDAPS

Kerberos

RDP / SSH / WinRM

Active Directory structure

Domain controllers

Internal DNS

DHCP infrastructure

Certificate services / AD CS metadata

Users, groups, computers, and relationships where authorized

Evidence and confidence

Historical changes

The goal is visibility and evidence, not automatic exploitation.

Authorized Use Only

IRIS is intended only for:

Networks you own

Lab environments

CTFs

Authorized penetration tests

Authorized internal security assessments

Never scan a network without explicit authorization.

IRIS should enforce strict scope controls, including explicit CIDR allowlists, before active network discovery.

The project intentionally avoids uncontrolled scanning, stealth/evasion, credential theft, persistence, and automatic exploitation in the core platform.

Why IRIS?

External attack-surface management focuses primarily on what an organization exposes to the internet.

Internal environments contain a different class of security information:

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

A useful internal security platform therefore needs to understand more than:

10.10.10.15 → 445/tcp OPEN

It should build a picture such as:

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

IRIS turns those observations into an understandable internal attack-surface map.

Core Principles

1. Evidence First

Every important observation should have supporting evidence.

Example:

Asset:
DC01

Service:
Kerberos

Port:
88/tcp

Evidence:
Observed TCP service + protocol response

Confidence:
HIGH

2. Confidence Over Guessing

Use:

HIGH
MEDIUM
LOW
UNKNOWN

Direct observations should carry more confidence than inferred information.

3. Relationships Matter

Internal security is relationship-driven.

IRIS should model relationships such as:

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

Architecture

IRIS is designed as a native Windows application.

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

Technology Stack

Core

Python 3.11+

SQLite

SQLAlchemy

Pydantic

asyncio

CLI

Typer

Rich

API

FastAPI

Uvicorn

Network / Protocol Collection

Use established tools and Python libraries where appropriate, including:

Nmap

DNS libraries

SMB protocol libraries

LDAP libraries

Kerberos-aware collection tooling

Windows-native networking capabilities

IRIS should orchestrate and normalize information rather than unnecessarily reinvent mature protocol implementations.

Native Windows Design

IRIS runs directly on:

Windows 10 / Windows 11
Python 3.11+
PowerShell
VS Code

No Docker

The core application does not require:

Docker Desktop

Kubernetes

PostgreSQL

Redis

Neo4j

Linux VM

WSL

Cloud infrastructure

Initial database:

data/iris.db

Local dashboard:

http://127.0.0.1:8000

Installation

git clone <repository>
cd iris

python -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt

iris doctor

Example:

IRIS SYSTEM CHECK

Python          ✓ 3.11.9
SQLite          ✓
Nmap            ✓
Configuration   ✓
Database        ✓

IRIS is ready.

Scope Model

IRIS requires explicit scope controls.

Example:

iris recon 10.10.10.0/24

Before scanning, IRIS should:

Parse the target.

Validate the CIDR.

Confirm the authorized scope.

Create a scan ID.

Apply concurrency and rate limits.

Perform discovery.

Ensure active operations remain within scope.

Dry Run

iris recon 10.10.10.0/24 --dry-run

Example:

Target scope:
10.10.10.0/24

Hosts that would be scanned:
256 addresses

Network activity:
DISABLED

Dry-run complete.

Core Capabilities

1. Host Discovery

Collect:

IP address

MAC address where available

Vendor

Hostname

Reverse DNS

OS indicators

Discovery method

Timestamp

Confidence

Example:

10.10.10.15
Hostname: DC01
Vendor: Microsoft
OS: Windows Server
Confidence: HIGH

2. Port and Service Discovery

Example:

HOST: DC01

PORT      STATE     SERVICE

53/tcp    open      DNS
88/tcp    open      Kerberos
389/tcp   open      LDAP
445/tcp   open      SMB
636/tcp   open      LDAPS
3389/tcp  open      RDP

Store each observation as structured data.

3. SMB Intelligence

Where authorized, collect:

SMB availability

SMB protocol/version indicators

Server identity

Domain/workgroup information

Share information

Authentication requirements

Anonymous-access indicators where safely testable

Evidence and confidence

Distinguish between:

Not tested
Not observed
Confirmed
Unknown

4. Active Directory Intelligence

Where authorized credentials and permissions are available, collect:

Domain
Domain Controllers
Users
Groups
Computers
Organizational Units
Group Policy metadata
Trust relationships
Service-related identity information

Do not automatically perform exploitation.

5. LDAP / LDAPS

Collect authorized information from:

389/tcp
636/tcp

Record:

Server

Naming context

Domain information

Supported capabilities

Directory objects where authorized

TLS information for LDAPS

Evidence

6. Kerberos

Identify Kerberos infrastructure, typically:

88/tcp
464/tcp

Correlate services with domains and domain controllers.

The objective is visibility, not automated credential attacks.

7. RDP / SSH / WinRM

Identify remote administration exposure:

DC01
├── RDP     3389
└── WinRM   5985

LINUX01
└── SSH     22

Track changes between scans.

8. Internal DNS

Collect:

A
AAAA
CNAME
MX
NS
SOA
SRV
TXT

Pay particular attention to authorized infrastructure records such as:

_ldap._tcp
_kerberos._tcp
_gc._tcp

9. DHCP

Where authorized, identify DHCP infrastructure and relevant metadata:

DHCP servers

Scope information

Network relationships

Lease-related observations

10. Certificate Services

Identify internal certificate infrastructure:

Certificate Authorities
AD CS
Certificate services
LDAP certificates
RDP certificates
Web certificates
Expiration
Issuer relationships

Focus initially on discovery and inventory rather than certificate abuse.

Asset Model

Core asset types:

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

Everything should be normalized.

Example:

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

Evidence Model

Each observation should contain:

asset_id
observation_type
value
source
method
timestamp
confidence
raw_evidence

Example:

{
  "asset": "DC01",
  "observation": "domain_controller",
  "value": true,
  "source": "LDAP",
  "confidence": "HIGH"
}

Historical Monitoring

IRIS should remember previous scans.

iris diff 41 42

Example:

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

Track:

New assets

Removed assets

New ports

Removed ports

Changed services

Changed technologies

Changed certificates

DNS changes

AD structure changes

Dashboard

The local dashboard should eventually provide:

Network Overview

Hosts                  418
Windows Hosts          302
Linux Hosts              76
Domain Controllers       2
Open Services          891
AD Users               842
AD Groups              126
Certificates            43
Potential Findings      17

Infrastructure Map

Network
   │
   ├── Domain
   │     ├── DC01
   │     └── DC02
   │
   ├── Servers
   ├── Workstations
   └── Network Devices

AD View

CORP.LOCAL

Users
Groups
Computers
OUs
Domain Controllers
Trusts
Services

Exposure View

RDP
SMB
LDAP
LDAPS
Kerberos
WinRM
SSH

CLI

iris --help
iris doctor
iris recon 10.10.10.0/24
iris hosts
iris services
iris ad
iris findings
iris scans
iris diff 41 42

Example:

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

Database

Use SQLite:

data/iris.db

Potential tables:

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

Use SQLAlchemy.

Keep the abstraction clean enough that PostgreSQL could be added later.

Project Structure

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

Development Roadmap

Phase 1 — Core

Target: 1–2 weeks

Build:

Native Windows CLI

Scope enforcement

SQLite

Scan management

Logging

iris doctor

Basic host discovery

Basic port discovery

Phase 2 — Network Intelligence

Service detection

OS fingerprinting

DNS

Reverse DNS

MAC/vendor

Infrastructure relationships

Phase 3 — Windows / SMB

SMB discovery

Share enumeration where authorized

Windows host information

Remote-access service detection

Phase 4 — Active Directory

Domain discovery

Domain controllers

LDAP

Kerberos

Users

Groups

Computers

OUs

Trust relationships

Phase 5 — Certificate & Infrastructure Intelligence

AD CS discovery

Certificate authorities

Certificate relationships

Internal DNS

DHCP

TLS/certificate monitoring

Phase 6 — Evidence & Historical Intelligence

Confidence engine

Evidence model

Scan comparison

Change detection

Asset history

Phase 7 — Dashboard

Network overview

Host inventory

Service inventory

AD explorer

Certificate view

Findings

Historical changes

Relationship visualization

What IRIS Should NOT Do Initially

Do not turn IRIS into an automated exploitation framework.

Avoid building automatic:

Credential attacks

Password spraying

Kerberoasting execution

NTLM relay

AD exploitation

Privilege escalation

Persistence

Lateral movement

Credential dumping

The initial platform should answer:

What exists inside this environment, how is it connected, what is exposed, and what changed?

Future Security Analysis

Once inventory is mature, IRIS can add an analysis layer that identifies relationships requiring security review.

For example:

User A
  ↓
Member of Group B
  ↓
Group B has access to Server C
  ↓
Server C exposes SMB

The platform should clearly distinguish:

Observed fact

from:

Security interpretation

Testing

Create tests for:

CIDR validation

Scope enforcement

Host parsing

Nmap XML parsing

DNS parsing

SMB parsing

LDAP parsing

AD object normalization

Kerberos service detection

Certificate parsing

Confidence scoring

Relationship creation

Historical diff

Database operations

API endpoints

Use mocked data wherever possible.

Do not make the test suite dependent on a live corporate network.

Future Relationship With SCOPEX

IRIS and SCOPEX can eventually share a common security-intelligence philosophy:

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

Both systems should remain independently useful.

