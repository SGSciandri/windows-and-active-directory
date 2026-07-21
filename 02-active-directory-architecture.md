# Active Directory (AD) Architecture & Authentication Mechanics

## 1. Domain Hierarchy
* **Domains:** Core security boundary containing users, hosts, and Group Policy Objects (GPOs).
* **Trees:** Continuous namespace collection of domains bound by automatic two-way transitive trusts (`corp.local` $\rightarrow$ `uk.corp.local`).
* **Forests:** Global security boundary uniting multi-tree non-contiguous namespaces. Shares a single **Schema** (object definitions) and **Global Catalog** (directory index).

### Privilege Boundaries
* **Domain Admins (DA):** Full administrative authority restricted to a single domain.
* **Enterprise Admins (EA):** Unrestricted "God Mode" authority across every domain in the forest. Lives exclusively in the forest root domain.

## 2. Group Policy Objects (GPOs)
* **Replication Location:** Stored on Domain Controllers inside the `SYSVOL` network share (`\\<Domain>\SYSVOL`).
* **Force Sync:** `gpupdate /force`
* **GPO Misconfiguration Vulnerability:** Scheduled tasks or backup scripts deployed via GPO execute as `NT AUTHORITY\SYSTEM`. If permissions on the deployment script/share grant `Write/Modify` to low-privileged users, attackers can inject malicious code for enterprise-wide privilege escalation.

## 3. Authentication Protocol Comparison

### Kerberos (Modern Default)
Ticket-based mutual authentication managed by the Key Distribution Center (KDC) on Domain Controllers.
1. **AS Exchange:** Client sends `AS-REQ` (timestamp encrypted with password hash). KDC returns `AS-REP` containing a Ticket Granting Ticket (TGT) encrypted with the KDC's secret key (`krbtgt`).
2. **TGS Exchange:** Client presents TGT to request access to a Service Principal Name (SPN). KDC returns a Service Ticket (TGS) encrypted with the target service account's password hash.
3. **AP Exchange:** Client presents TGS ticket directly to the application server for access validation.
* **Primary Vulnerability (Kerberoasting):** Any authenticated user can request TGS tickets for SPNs, extract the ticket from local memory, and perform offline password cracking against the service account's hash.

### NetNTLM (Legacy / Fallback)
Challenge-response protocol maintained for backward compatibility.
1. Client requests access to a resource.
2. Server responds with a random 8-byte **Challenge**.
3. Client encrypts the challenge using a key derived from their password hash (**Response**).
4. Server forwards Challenge + Response to Domain Controller for verification.
* **Primary Vulnerability (NTLM Relaying):** Lack of mutual authentication allows Man-in-the-Middle tools (e.g., Responder) to intercept challenge-response traffic and relay it to another target server to gain unauthorized access. *Mitigation: Require SMB Signing or disable NTLM.*

## 4. Trust Boundaries & Lateral Movement
* **Direction:** Access flows opposite to trust direction. (If Domain A trusts Domain B, users in Domain B access resources in Domain A).
* **Transitivity:** Transitive trusts pass down the chain (A $\rightarrow$ B $\rightarrow$ C means A trusts C). Non-transitive trusts restrict access solely to the two participating domains.
* **Forest Hopping:** Attackers leverage tools like BloodHound to map misconfigured child-to-parent trusts to escalate from a compromised child domain up to the Forest Root.
