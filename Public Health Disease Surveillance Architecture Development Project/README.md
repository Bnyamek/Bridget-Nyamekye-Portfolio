# 🏥 Public Health Disease Surveillance Architecture Development Project

> A multi-component health informatics infrastructure simulating a regional disease outbreak surveillance system across the Upper Peninsula of Michigan — built on virtual machines, open-source EHRs, synthetic patient data, and a FHIR-compliant Health Information Exchange.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Modules](#project-modules)
  - [Part 1 – VM Environment Setup](#part-1--vm-environment-setup)
  - [Part 2 – OpenEMR Installation & Security](#part-2--openemr-installation--security)
  - [Part 3 – Synthetic Patient Data Generation (Synthea)](#part-3--synthetic-patient-data-generation-synthea)
  - [Part 4 – HAPI-FHIR Server Installation & Configuration](#part-4--hapi-fhir-server-installation--configuration)
  - [Part 5 – FHIR Data Exchange & Interoperability](#part-5--fhir-data-exchange--interoperability)
- [Disease Outbreak Simulation Results](#disease-outbreak-simulation-results)
- [Key Commands & Troubleshooting](#key-commands--troubleshooting)
- [Security Implementation](#security-implementation)
- [Challenges & Lessons Learned](#challenges--lessons-learned)
- [Outcomes](#outcomes)
- [Author](#author)

---

## Project Overview

This project simulates a **real-world regional public health disease surveillance system** for the Upper Peninsula of Michigan. It models a COVID-19 outbreak scenario across four hospitals feeding syndromic surveillance data into a central **Health Information Exchange (HIE)**.

The infrastructure was built entirely using **open-source tools** on a virtualized environment, demonstrating the full data lifecycle from patient record generation to FHIR-compliant health data exchange — a critical workflow in modern public health informatics.

**Core Objectives:**
- Design and configure a multi-VM health network architecture
- Deploy and secure electronic health records (EHRs) at each hospital
- Generate realistic synthetic patient and COVID-19 syndromic surveillance data
- Stand up a FHIR-compliant Health Information Exchange server
- Demonstrate interoperability through HL7 FHIR data exchange via REST API

---

## Architecture

The system consists of **five virtual machines** (Ubuntu Server) hosted on the College of Computing Cluster (CCC):

```
┌─────────────────────────────────────────────────────────────┐
│            Upper Peninsula Health Information Exchange       │
│                      (UPHIE) — HAPI-FHIR Server             │
│                       Port: 8090 / 8080                     │
└──────────────┬───────────┬──────────────┬───────────────────┘
               │           │              │
    ┌──────────▼──┐  ┌─────▼──────┐  ┌───▼──────────┐  ┌────────────────┐
    │   Aspirus   │  │  Portage   │  │    BCMH      │  │      MGH       │
    │  Hospital   │  │   Health   │  │  (Baraga Co. │  │  (Marquette    │
    │  OpenEMR    │  │  OpenEMR   │  │  Memorial)   │  │   General)     │
    │             │  │            │  │  OpenEMR     │  │  OpenEMR       │
    └─────────────┘  └────────────┘  └──────────────┘  └────────────────┘
```

Each hospital VM runs **OpenEMR** (web-based EHR with Apache + MySQL). The UPHIE VM hosts the **HAPI-FHIR** server, acting as the central repository for interoperable FHIR-formatted health data.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| **Ubuntu Server (Linux)** | Operating system for all 5 VMs |
| **OpenEMR 6.0.1** | Open-source EHR deployed at each hospital |
| **Apache2** | Web server for OpenEMR |
| **MySQL** | Relational database backend for OpenEMR |
| **Synthea** | Synthetic patient and syndromic surveillance data generation |
| **HAPI-FHIR** | HL7 FHIR-compliant server for the HIE (UPHIE) |
| **HL7 FHIR (R4)** | Standard for healthcare data exchange |
| **Postman** | REST API client used for FHIR resource testing and exchange |
| **UFW (Uncomplicated Firewall)** | Firewall configuration and port management |
| **Java** | Runtime environment required by HAPI-FHIR |

---

## Project Modules

### Part 1 – VM Environment Setup

Configured five Ubuntu Server virtual machines on the College of Computing Cluster (CCC), each mapped to a real-world entity in the Upper Peninsula health network:

- **Aspirus Hospital** — `aspirus` VM
- **Portage Health Hospital** — `Bridget-portage` VM
- **Baraga County Memorial Hospital (BCMH)** — `Bridget-bcmh` VM
- **Marquette General Hospital (MGH)** — `mgh` VM
- **Upper Peninsula Health Information Exchange (UPHIE)** — `Bridget-uphie` VM

Each VM was provisioned with network connectivity and verified for compatibility with its target software stack (OpenEMR or HAPI-FHIR).

---

### Part 2 – OpenEMR Installation & Security

**OpenEMR 6.0.1** was installed and configured on all four hospital VMs as the primary Electronic Health Record (EHR) system.

**Installation stack per hospital VM:**
- Apache2 web server
- MySQL database
- PHP
- OpenEMR web application (accessed via browser at the VM's IP address)

**Security hardening steps applied to each instance:**

1. **System updates** — Applied all latest security patches prior to installation:
   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

2. **Automatic security updates** — Configured unattended upgrades:
   ```bash
   sudo apt-get install unattended-upgrades
   sudo dpkg-reconfigure --priority=low unattended-upgrades
   ```

3. **Firewall configuration (UFW)** — Restricted traffic to essential services only:
   ```bash
   sudo apt-get install ufw
   sudo ufw allow http
   sudo ufw allow ssh
   sudo ufw enable
   ```

4. **Apache hardening** — Configured security headers to protect against common web-based attacks:
   ```bash
   sudo nano /etc/apache2/conf-available/security.conf
   ```

5. **MySQL hardening** — Secured the database by removing anonymous users, disabling remote root login, and setting a strong root password:
   ```bash
   sudo mysql_secure_installation
   ```

**Remaining attack surfaces identified:**
- **DDoS (Distributed Denial-of-Service)** attacks — flooding the server with traffic to disrupt services
- **Brute-force login attacks** — repeated credential guessing to gain unauthorized access

---

### Part 3 – Synthetic Patient Data Generation (Synthea)

Used **Synthea** (open-source synthetic patient data generator) to simulate a COVID-19 disease outbreak across the Upper Peninsula, generating realistic patient records and HL7-formatted syndromic surveillance messages for each hospital.

**What Synthea generates:**
- Patient demographics
- Medical conditions and diagnoses
- Clinical observations and lab results
- Medication and treatment histories
- COVID-19 syndromic surveillance HL7 FHIR messages

**Output format:** FHIR JSON bundles (stored in `~/synthea/output/fhir/` on each hospital VM)

**Outbreak simulation parameters** — each hospital was assigned a COVID-19 patient load proportional to its real-world service population:

| Hospital | % of Population Served | Simulated COVID-19 Patients |
|---|---|---|
| Aspirus Hospital | 1% | 20 |
| Portage Health Hospital | 1% | 40 |
| Baraga County Memorial (BCMH) | 3% | 210 |
| Marquette General Hospital (MGH) | 5% | 1,400 |

Synthea output was confirmed on each VM (screenshots of FHIR output directories visible on `mgh`, `Bridget-bcmh`, and `Bridget-portage` VMs).

---

### Part 4 – HAPI-FHIR Server Installation & Configuration

Installed and configured **HAPI-FHIR** on the UPHIE (`Bridget-uphie`) VM to serve as the central Health Information Exchange repository.

**Server details:**
- Default port: `8090` (configured, verified open)
- Web UI accessible at: `http://localhost:8090/`
- Backend: Java-based Spring Boot application

**Diagnostic commands used:**

**1. Check total, used, and free memory:**
```bash
sudo free -h
```

**2. Verify HAPI-FHIR port is open through firewall:**
```bash
sudo ss -tulnp | grep ':8090'
```

**3. Check if port 8090 is in use by another process:**
```bash
sudo lsof -i :8090
```

**Bonus – Custom UI Branding:**
The default HAPI-FHIR web UI was customized to reflect the UPHIE identity, replacing "Company Name" with **"Upper Peninsula Health Care Network"** and "Your Sample Text Here" with **"The Health Information Exchange for the Upper Peninsula"**.

---

### Part 5 – FHIR Data Exchange & Interoperability

Demonstrated live HL7 FHIR data exchange between hospital EHRs and the UPHIE HAPI-FHIR server using **Postman** as the REST API client.

#### Creating a FHIR Resource (POST)

1. Open Postman and set the HTTP method to `POST`
2. Set the request URI to:
   ```
   http://localhost:8090/fhir/Practitioner
   ```
3. Set the body to `raw` / `JSON` and paste a valid FHIR Practitioner resource
4. Click **Send**
5. A successful response returns HTTP `201 Created` along with the new resource ID

**Example request body (Practitioner resource):**
```json
{
  "resourceType": "Practitioner",
  "active": true,
  "name": [
    {
      "family": "Smith",
      "given": ["John"]
    }
  ]
}
```

#### Managing Authentication in Postman

1. In Postman, open the **Authorization** tab for the request
2. Select **Basic Auth** from the dropdown
3. Enter your HAPI-FHIR server **username and password**
4. Postman automatically attaches the Authorization header to all requests

#### Handling FHIR Server Error Responses

| HTTP Status Code | Meaning | Resolution |
|---|---|---|
| `400 Bad Request` | Malformed input or missing required fields | Review and correct the FHIR resource JSON |
| `404 Not Found` | Resource does not exist at the given ID | Verify the resource ID and endpoint |
| `422 Unprocessable Entity` | Validation error in the FHIR resource | Check the `OperationOutcome` response body |
| `500 Internal Server Error` | Server-side failure | Review HAPI-FHIR server logs |

When errors occur, the HAPI-FHIR server returns an **OperationOutcome** resource in the response body containing `severity`, `error code`, and a human-readable `description` — use this to diagnose and fix the request before retrying.

**Successful GET request** (retrieving a Practitioner resource) returns HTTP `200 OK` with the full resource JSON.

---

## Disease Outbreak Simulation Results

The full simulation modeled a regional COVID-19 outbreak across the Upper Peninsula health network. Synthea-generated FHIR bundles were uploaded to the UPHIE HAPI-FHIR server, simulating the real-time flow of syndromic surveillance data from hospital EHRs to a central HIE.

This workflow enables public health analysts to:
- Detect geographic clusters of disease in near real-time
- Monitor outbreak progression by hospital catchment area
- Allocate resources based on aggregated patient burden data
- Trigger evidence-based public health interventions

---

## Key Commands & Troubleshooting Reference

```bash
# System maintenance
sudo apt-get update && sudo apt-get upgrade

# Firewall management
sudo ufw status
sudo ufw allow 8090/tcp
sudo ufw enable

# HAPI-FHIR diagnostics
sudo free -h                          # Check memory
sudo ss -tulnp | grep ':8090'         # Check if port is open
sudo lsof -i :8090                    # Check what process is using port 8090
sudo systemctl status hapi-fhir       # Check HAPI-FHIR service status

# OpenEMR / Apache
sudo systemctl status apache2
sudo systemctl restart apache2

# MySQL security
sudo mysql_secure_installation
```

---

## Security Implementation

| Security Layer | Tool/Method | Purpose |
|---|---|---|
| System patching | `apt-get upgrade` + `unattended-upgrades` | Prevent known vulnerability exploitation |
| Network firewall | UFW | Restrict access to HTTP (80) and SSH (22) only |
| Web server hardening | Apache security headers | Mitigate XSS, clickjacking, and info leakage |
| Database security | `mysql_secure_installation` | Remove anonymous users, restrict remote root access |
| Port monitoring | `ss`, `lsof` | Detect unauthorized services on critical ports |

---

## Challenges & Lessons Learned

- **MGH VM configuration** required troubleshooting due to a configuration error during initial setup, which required identifying the root cause and reconfiguring before continuing — reinforcing the importance of methodical setup documentation.
- **HAPI-FHIR memory management** is critical: the server requires sufficient heap memory (configured via JVM flags), and the `free -h` command was essential for diagnosing startup failures.
- **Port conflicts** between services (particularly when running multiple network services on a single VM) required careful use of `ss` and `lsof` to ensure the FHIR server had exclusive access to its assigned port.
- **Synthea data volume** — generating realistic patient populations for MGH (1,400 COVID patients) produced a large number of FHIR JSON files, highlighting the importance of storage planning in real-world HIE deployments.
- **FHIR resource validation** — learning to interpret `OperationOutcome` error responses from HAPI-FHIR was key to understanding how FHIR servers enforce data quality and schema compliance.

---

## Outcomes

By the end of this project, the following was achieved:

- ✅ Designed and deployed a 5-node virtual health network architecture
- ✅ Successfully installed, configured, and secured **OpenEMR 6.0.1** on four independent hospital VMs
- ✅ Generated synthetic COVID-19 outbreak data for **1,670+ patients** across four hospital catchment areas using Synthea
- ✅ Installed and configured a **HAPI-FHIR server** as a fully operational regional Health Information Exchange
- ✅ Demonstrated end-to-end **HL7 FHIR interoperability** via Postman (POST, GET operations with 201/200 responses)
- ✅ Customized the HAPI-FHIR web UI to represent the Upper Peninsula Health Care Network
- ✅ Applied multi-layer security hardening across all VMs

---

## Author

**Bridget Nyamekye**  
Health Informatics | Public Health Surveillance | Health IT Architecture  

*Michigan Technological University — College of Computing*

---

> **Note:** All patient data used in this project is entirely **synthetic**, generated by Synthea for simulation and research purposes only. No real patient data was used at any point.