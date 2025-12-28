# CyberArk PAS 13.2: Enterprise Implementation & Disaster Recovery Lab

**Role:** Identity Security Engineer (Lab Simulation)
**Architecture:** Multi-Node High Availability (HA) Cluster
**Version:** CyberArk PAS 13.2 (Self-Hosted)

---

&nbsp;

## 1. Executive Summary
This project documents the end-to-end deployment of a **CyberArk Privileged Access Security (PAS)** environment, simulating a banking-grade infrastructure. The objective was to architect a resilient Identity Security platform capable of **Session Management**, **Automatic Password Rotation**, and **Disaster Recovery (DR)** failover.

**Key Engineering Achievements:**
* **Infrastructure:** Deployed Primary Vault, DR Vault, and hardened Component Servers on Windows Server 2019.
* **Security:** Implemented "Least Privilege" access models and enforced CyberArk hardening scripts (GPO/AppLocker).
* **Resiliency:** Successfully configured `PADR` replication for <15 minute RPO (Recovery Point Objective).
* **Integration:** Integrated PSM/CPM components with Active Directory for enterprise identity management.

&nbsp;

---

## 2. Infrastructure Architecture

| Node | Role | IP Address | Status |
| :--- | :--- | :--- | :--- |
| **Vault-Primary** | Core Database (Active) | `10.0.0.1` | 🟢 Online |
| **Vault-DR** | Disaster Recovery (Passive) | `10.0.0.4` | 🟢 Replicating |
| **Component-Svr** | PVWA / CPM / PSM | `10.0.0.2` | 🟢 Connected |
| **UVBank-DC** | Domain Controller (Identity Source) | `10.0.0.3` | 🟢 Active |

&nbsp;

---

## 3. Build Log: Primary Core (The Vault)
*Objective: Deploy the "Digital Vault" - the hardened, isolated storage engine for all privileged credentials.*

**Step 1:** Installation of `.NET Framework 4.8` prerequisites to support the Vault service logic.
**[Pre-requisites]**
<img width="565" height="491" alt="v13-installation-1" src="https://github.com/user-attachments/assets/90c6e4aa-e5a0-423b-a572-a21c25044118" />


&nbsp;


**Step 2:** Defining the **Master** and **Administrator** cryptographic keys. The Master Key is stored offline (air-gapped) for emergency recovery.
**[Master Password Setup]**
<img width="514" height="390" alt="built-in-user-2" src="https://github.com/user-attachments/assets/e79b2b42-68d1-491c-ad9a-ed3e12f201fe" />


&nbsp;


**Step 3:** Post-installation hardening. The installer automatically stripped unnecessary Windows services to reduce the attack surface.
**[Vault Installation Complete]**
<img width="522" height="396" alt="vault-setup-complete-3" src="https://github.com/user-attachments/assets/c1f153cb-9eb5-4b76-afe9-a6ab50ad5563" />


&nbsp;


**Step 4:** Service Verification. The `PrivateArk Server` service is active (`ITADB313I Server is up`).
**[Service Status]**
<img width="947" height="304" alt="Vault-ser-running-4" src="https://github.com/user-attachments/assets/ec3a3def-978d-4a08-92c5-3e5391998315" />


&nbsp;


**Step 5:** Management Access. Establishing the first connection via the **PrivateArk Client** (Thick Client) to verify safe structure.
**[PrivateArk Client]**
<img width="810" height="482" alt="Dr-vault-health-25" src="https://github.com/user-attachments/assets/01d57fb1-122e-488b-9b4b-b5842b358bc5" />


---
&nbsp;


## 4. Web Access Layer (PVWA)
*Objective: Deploy the "Single Pane of Glass" for end-users and auditors.*

&nbsp;

**Step 6:** Automation. Executed the `PVWA_Prerequisites.ps1` script to automatically configure IIS and Web Roles.
**[Hardening Script]**
<img width="783" height="369" alt="PVWA-Script-6" src="https://github.com/user-attachments/assets/ac799d37-c472-4a0b-b518-6c48a7cbf114" />

&nbsp;


**Step 7:** Validation of the automated pre-req script execution.
**[Script Success]**
<img width="782" height="354" alt="PVWA-pre-req-iis-installed-7" src="https://github.com/user-attachments/assets/b9f227f3-fd0e-4c8b-8aab-49a20e5b1b6b" />


&nbsp;


**Step 8:** Binding the Web App to the Vault. Port `1858` (CyberArk Protocol) is opened for encrypted communication.
**[Vault Connection]**
<img width="505" height="385" alt="pvwa-setup-address-url-8" src="https://github.com/user-attachments/assets/35242b86-bde0-4ca1-aa00-47ad2e2ca293" />



&nbsp;


**Step 9:** Completion of the Web Application deployment.
**[PVWA Setup Complete]**
<img width="506" height="384" alt="PVWA-installment-complete-9" src="https://github.com/user-attachments/assets/f1efc4bd-8bf3-4b07-9099-e09786223fc1" />


&nbsp;


**Step 10:** First Login. Accessing the web interface via `https://10.0.0.2/PasswordVault`.
**[Login Screen]**
<img width="731" height="427" alt="PVWA-web-login-10" src="https://github.com/user-attachments/assets/b6e4542c-1d65-4253-9c23-1f4715d8d7f6" />


&nbsp;


**Step 11:** **Health Check.** The PVWA successfully established a heartbeat with the Primary Vault.
**[System Health Green]**
<img width="688" height="488" alt="PVWA-successful-login-11" src="https://github.com/user-attachments/assets/a6730c6f-830a-4bce-aa66-6fd834213af9" />


---


&nbsp;


## 5. The Policy Engine (CPM)
*Objective: Automate the rotation of passwords to eliminate static credentials.*

**Step 12:** Initiating the Central Policy Manager (CPM) installation agent.
**[CPM Wizard]**
<img width="504" height="386" alt="CPM-initial-install-12" src="https://github.com/user-attachments/assets/a632f494-0680-4cfc-96c1-358667cf4455" />


&nbsp;


**Step 13:** The CPM service was installed and hardened (Scanner service disabled by default for security).
**[CPM Complete]**
<img width="506" height="386" alt="CPM-installed-13" src="https://github.com/user-attachments/assets/64b57764-2a59-4c17-9d86-ae7839bdb5f4" />


&nbsp;


**Step 14:** **Verification.** The CPM is now reported as "Connected" in the System Health dashboard, ready to rotate keys.
**[CPM Health]**
<img width="810" height="319" alt="cpm-pvwa-connect-complete-14" src="https://github.com/user-attachments/assets/f83f0b76-8372-4a3a-a28a-41e8f04a2bbe" />


---

&nbsp;

## 6. The Session Manager (PSM)
*Objective: Isolate end-user sessions and record privileged activity.*

**Step 15:** Domain Integration. Joined the Component Server to the `UVBANK.com` domain to support LDAP authentication.
**[Domain Join]**
<img width="407" height="438" alt="cmponent-server-join-bank-domain-15" src="https://github.com/user-attachments/assets/56235adc-cac2-4343-aa23-33118c6316c9" />


&nbsp;


**Step 16:** RDS Automation. Used PowerShell to deploy Remote Desktop Services and disable NLA (Network Level Authentication) for the PSM Connect workflow.
**[PSM Pre-reqs]**
<img width="817" height="524" alt="PSM-pre-req-installment-16" src="https://github.com/user-attachments/assets/9f466eb5-72e6-4555-904f-371cc1abfeac" />

&nbsp;


**Step 17:** Launching the PSM Installation Wizard.
**[PSM Wizard]**
<img width="694" height="518" alt="pms-phase1-17" src="https://github.com/user-attachments/assets/5b84b264-9cb9-49d9-a0ca-ddefe7d2a04c" />

&nbsp;


**Step 18:** Vault Handshake. The PSM installer authenticates as `Administrator` to create its own environment users (`PSMApp_`, `PSMGw_`).
**[Vault Authentication]**
<img width="693" height="521" alt="psm-phase2-18" src="https://github.com/user-attachments/assets/b2e3195a-d0de-41a9-b222-3481da5b5943" />

&nbsp;


**Step 19:** Hardening. The installer applied `AppLocker` rules, locking down the server to only allow whitelisted applications.
**[PSM Complete]**
<img width="696" height="529" alt="psm-phase3-19" src="https://github.com/user-attachments/assets/9d468a45-dfc2-4f7f-b597-c5195b79b53a" />

&nbsp;


**Step 20:** **Operational Status.** The PSM is fully active and listening for RDP/SSH proxy connections.
**[PSM Connected]**
<img width="738" height="246" alt="PSM-connected-20" src="https://github.com/user-attachments/assets/a1d8ed34-0971-45a8-aae8-a0da6ad7ba24" />

---
&nbsp;


## 7. Disaster Recovery (The Resiliency Layer)
*Objective: Build a failover site to ensure Business Continuity (BCP).*

**Step 21:** Initiating the **DR Vault** installation on a secondary node (`10.0.0.4`).
**[DR Wizard]**
<img width="511" height="391" alt="DR-initial-install-21" src="https://github.com/user-attachments/assets/6ac13d1e-aea1-4abd-bf13-d0073f96a843" />


&nbsp;


**Step 22:** User Provisioning. Enabling the specific `DR` user on the Primary Vault to allow backup replication.
**[Enable DR User]**
<img width="532" height="470" alt="Enable-DrUser-for-vault-rep-22" src="https://github.com/user-attachments/assets/2a605d71-81d7-4675-8ca3-1c0aee646770" />


&nbsp;


**Step 23:** Handshake. The DR Vault authenticates to the Primary using the `DR` credential.
**[DR User Input]**
<img width="510" height="388" alt="Dr-rep-user-23" src="https://github.com/user-attachments/assets/a73307a9-b2e9-4462-866b-765bd45dbba9" />


&nbsp;


**Step 24:** Success. The Disaster Recovery service is installed and set to "Automatic" startup.
**[DR Complete]**
<img width="506" height="388" alt="Dr-vault-complete-24" src="https://github.com/user-attachments/assets/be6e3d96-5159-4126-92df-de03fc03c6d8" />


&nbsp;


**Step 25:** **Architecture Validation.** The System Health Dashboard now confirms a **Primary -> DR** replication link.
**[Health Dashboard]**
<img width="810" height="482" alt="Dr-vault-health-25" src="https://github.com/user-attachments/assets/c92f5fba-fa9d-441e-9afa-6443ada786d6" />


&nbsp;


**Step 26:** **Proof of Replication.**
*Log Analysis (`padr.log`):*
> `PADR0010I Replicate ended.`
> `PADR0099I Metadata Replication is running successfully.`

This log confirms that the DR Vault is successfully pulling encrypted data from the Primary Vault every 5 minutes.
**[Replication Log]**
<img width="812" height="273" alt="Proof-vault-works-26" src="https://github.com/user-attachments/assets/5fdee587-948f-4007-80b9-8722f02826c2" />


---
&nbsp;


### Project Outcome
This lab successfully emulates a **Tier-1 Banking Environment**. The architecture supports:
1.  **High Availability:** Data is replicated to a secondary site.
2.  **Compliance:** All sessions are recorded (PSM) and passwords are rotated (CPM).
3.  **Security:** All components are hardened according to CyberArk vendor standards.
