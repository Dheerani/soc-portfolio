# SOC Analyst Portfolio — Wazuh Lab

## About Me
Aspiring SOC Analyst with hands-on experience in Wazuh SIEM, custom detection rules, MITRE ATT&CK mapping, and incident response.

## Lab Environment
- **SIEM:** Wazuh 4.14.7
- **Attacker:** Kali Linux
- **Target:** Kali (localhost)
- **Web Server:** Apache2
- **Period:** 6 days
- **Honeypot:** Cowrie (Docker, Port 2222)
- **Log Ingestion:** `cowrie.json` (Custom JSON Decoder & Rules)

## Detections Implemented (37+)

| # | Detection | Rule ID | MITRE | Status |
|---|-----------|---------|-------|--------|
| 1 | SSH Brute Force | 100002 | T1110 | ✅ |
| 2 | Nmap Scan | 100003 | T1046 | ✅ |
| 3 | File Integrity | 550, 553, 554 | T1565 | ✅ |
| 4 | Rootkit | Rootcheck | T1014 | ✅ |
| 5 | Web Attack (XSS) | 31106 | T1190 | ✅ |
| 6 | SQL Injection | 31106 | T1190 | ✅ |
| 7 | Path Traversal | 31106 | T1083 | ✅ |
| 8 | Command Injection | 31106 | T1059 | ✅ |
| 9 | File Upload (Web Shell) | 31514 | T1505.003 | ✅ |
| 10 | File Inclusion | 31106 | T1083 | ✅ |
| 11 | SSRF | 31106 | T1190 | ✅ |
| 12 | XXE | 31122 | T1190 | ✅ |
| 13 | SSTI | 31122 | T1190 | ✅ |
| 14 | Open Redirect | 31105 | T1190 | ✅ |
| 15 | Insecure Deserialization | 31122 | T1190 | ✅ |
| 16 | Log4Shell | 31122 | T1190 | ✅ |
| 17 | Spring4Shell | 31122 | T1190 | ✅ |
| 18 | LDAP Injection | 31122 | T1190 | ✅ |
| 19 | NoSQL Injection | 31122 | T1190 | ✅ |
| 20 | GraphQL Injection | 31122 | T1190 | ✅ |
| 21 | Advanced SSRF | 31122 | T1190 | ✅ |
| 22 | Billion Laughs | 31122 | T1499 | ✅ |
| 23 | HTTP Smuggling | 31101 | T1190 | ✅ |
| 24 | Web Cache Poisoning | 31122 | T1190 | ✅ |
| 25 | Host Header Injection | 31122 | T1190 | ✅ |
| 26 | Netcat (auditd) | 100020 | T1059 | ✅ |
| 27 | Cryptominer | 100021 | T1496 | ✅ |
| 28 | Recon Tools | 100022 | T1046 | ✅ |
| 29 | Correlation: Brute→Success | 100100 | T1110, T1078 | ✅ |
| 30 | Correlation: Sudo after breach | 100101 | T1548.003, T1078 | ✅ |
| 31 | VirusTotal: High-confidence | 100200 | T1204 | ✅ |
| 32 | VirusTotal: Medium | 100201 | T1204 | ✅ |
| 33 | VirusTotal: Low | 100202 | T1204 | ✅ |
| 34 | MyApp: Login failed | 100300 | T1110 | ✅ |
| 35 | MyApp: Brute force | 100302 | T1110 | ✅ |
| 36 | Cowrie: Successful Login | 100500 | T1078 | ✅ |
| 37 | Cowrie: Failed Login | 100501 | T1110 | ✅ |

## Active Response Implemented

| # | Trigger | Rule ID | AR Command | Action | Status |
|---|---------|---------|------------|--------|--------|
| 1 | SSH Failed Login | 100001 | firewall-drop | Block IP (600s) | ✅ |
| 2 | SSH Brute Force | 100002 | firewall-drop | Block IP (600s) | ✅ |
| 3 | SSH Brute Force Success | 100100 | firewall-drop + disable-account | Block IP (180s) + Lock account | ✅ |

## Honeypot Integration (Cowrie & Wazuh)

Deployed a Dockerized Cowrie SSH Honeypot integrated with Wazuh SIEM to capture and analyze attacker behavior in real-time.

### Architecture
1. **Attacker** → SSH attempts to Cowrie (Port 2222).
2. **Cowrie** → Generates JSON logs (`cowrie.json`) for every action (KEX, Login, Commands).
3. **Wazuh** → Ingests via `localfile` (log format: JSON).
4. **Custom Decoder** → `0001-cowrie_decoders.xml` (Standalone JSON_Decoder for Cowrie events).
5. **Custom Rules** → 100500 & 100501 (Level 8 alerts).

### Key Technical Challenges Solved
- **Decoder Priority:** Default `json` decoder was overriding the custom decoder. Fixed by renaming the custom decoder file to `0001-cowrie_decoders.xml`.
- **Parent Decoder Failure:** JSON logs lack a `program_name`, so the `<parent>` tag failed. Solved by making the decoder standalone.
- **Wazuh User Permissions:** Wazuh runs as the `wazuh` user. The `cowrie.json` file permissions and ownership had to be changed (`chown wazuh:wazuh`).
- **Log Visibility:** Enabled `logall_json` to ensure all Cowrie logs were captured in `archives.json` before rule matching.
- **Honeypot Behavior:** By default, Cowrie accepts *any* password. Therefore, detection rules were created for both `cowrie.login.success` and `cowrie.login.failed`.

### Proof of Concept
- **Attack Simulated:** SSH Brute-force to port 2222.
- **Alert Triggered:** Rule 100500 fired at Level 8.
- **Dashboard Visibility:** Alert visible in Wazuh Threat Hunting with extracted fields (`src_ip`, `username`, `password`).


## Alerting & Integrations (Discord)

To simulate a real-world SOC environment, I integrated Wazuh with Discord for real-time alerting.

- **Custom Python Script:** Wrote a script (`custom-discord.py`) that hooks into Wazuh's integration module.
- **Webhook:** Configured a Discord Webhook to receive alerts.
- **Threshold:** Set the integration to trigger only for `Level 8` and above alerts to prevent noise.
- **Result:** Whenever an attacker interacts with the Cowrie Honeypot, an immediate "🚨 SOC ALERT 🚨" notification is sent to Discord with the Rule ID, Description, and Attacker IP.

## Reports

| # | Report | Attack Type | Alerts |
|---|--------|-------------|--------|
| 1 | [SSH Brute Force](incident-report-002.md) | Brute Force | 19 |
| 2 | [File Integrity](fim-incident-report.md) | FIM | 117 |
| 3 | [MITRE Coverage](mitre-coverage-report.md) | Analysis | 686 |
| 4 | [Nmap Detection](nmap-detection-report.md) | Recon | 7 |
| 5 | [Web Attack (XSS)](web-attack-report.md) | XSS | 3 |
| 6 | [SQL Injection](sql-injection-report.md) | SQLi | 2 |
| 7 | [Path Traversal](path-traversal-report.md) | Traversal | 15 |
| 8 | [Command Injection](command-injection-report.md) | Command Injection | 2 |
| 9 | [File Upload](file-upload-report.md) | File Upload | 12 |
| 10 | [File Inclusion](file-inclusion-report.md) | File Inclusion | 8 |
| 11 | [SSRF](ssrf-report.md) | SSRF | 2 |
| 12 | [XXE](xxe-report.md) | XXE | 7 |
| 13 | [SSTI](ssti-report.md) | SSTI | 5 |
| 14 | [Open Redirect](open-redirect-report.md) | Open Redirect | 3 |
| 15 | [Deserialization](deserialization-report.md) | Insecure Deserialization | 5 |
| 16 | [Log4Shell](log4shell-report.md) | Log4Shell | 5 |
| 17 | [Spring4Shell](spring4shell-report.md) | Spring4Shell | 3 |
| 18 | [LDAP Injection](ldap-injection-report.md) | LDAP Injection | 5 |
| 19 | [NoSQL Injection](nosql-injection-report.md) | NoSQL Injection | 5 |
| 20 | [GraphQL Injection](graphql-injection-report.md) | GraphQL Injection | 5 |
| 21 | [Advanced SSRF](advanced-ssrf-report.md) | Advanced SSRF | 3 |
| 22 | [Billion Laughs](billion-laughs-report.md) | Billion Laughs | 3 |
| 23 | [HTTP Smuggling](smuggling-report.md) | HTTP Smuggling | 1 |
| 24 | [Web Cache Poisoning](cache-poisoning-report.md) | Web Cache Poisoning | 0 |
| 25 | [Host Header Injection](host-header-report.md) | Host Header Injection | 0 |
| 26 | [Process Monitoring](process-monitoring-report.md) | Process Monitoring | — |
| 27 | [Web Shell](web-shell-report.md) | Web Shell | 3 |
| 28 | [Cryptominer](cryptominer-report.md) | Cryptominer | 5 |
| 29 | [Reverse Shell](reverse-shell-report.md) | Reverse Shell | 5 |
| 30 | [Data Exfiltration](data-exfiltration-report.md) | Data Exfiltration | 5 |
| 31 | [Privilege Escalation](priv-esc-report.md) | Privilege Escalation | 5 |
| 32 | [Persistence](persistence-report.md) | Persistence | 5 |
| 33 | [**Active Response Test**](active-response-test-report.md) | **Brute Force + AR** | **100+** |
| 34 | [Cowrie Honeypot Login](cowrie-incident-report.md) | Honeypot Attack | 2 |

## Active Response Highlights

### Multi-Layer Defense
- **Network Layer:** IP blocking via `firewall-drop` (iptables)
- **Account Layer:** Account locking via `disable-account`

### Complete Attack Chain

Failed Login (100001)
    ↓
Brute Force Detected (100002)
    ↓
IP Blocked (firewall-drop, 600s)
    ↓
Successful Login (5715)
    ↓
Brute Force SUCCESS (100100)
    ↓
IP Blocked (180s) + Account Disabled

### Key Technical Findings
- `same_source_ip` is reliable for correlation (not `same_field>srcip`)
- Rule 100001 covers both PAM (5503) and sshd (5760) alerts
- Defense-in-depth: network block + account lock
- Complete attack chain from failed login to account disable

## Detection Engineering — Tuning Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| alerts.log size | 131 MB | ~50 KB | **99.96%** |
| Alerts per day | 59,476 | ~120 | **99.8%** |
| Rule 100005 fires | 41,121 | 0 | **100%** |
| False positives | High | Near zero | — |

## Custom Rules Created (18 Total)

| Rule ID | Description | Level | MITRE |
|---------|-------------|-------|-------|
| 100001 | SSH failed login | 10 | T1110 |
| 100002 | SSH Brute Force | 12 | T1110 |
| 100003 | SSH connection (Nmap) | 10 | T1046 |
| 100010 | Suppress: Wazuh internal | 0 | — |
| 100011 | Suppress: dash shell | 0 | — |
| 100020 | Netcat via auditd | 10 | T1059 |
| 100021 | Cryptominer via auditd | 12 | T1496 |
| 100022 | Recon tools via auditd | 12 | T1046 |
| 100100 | SSH brute force → success | 13 | T1110, T1078 |
| 100101 | Sudo after SSH breach | 14 | T1548.003, T1078 |
| 100200 | VirusTotal: 50+ engines | 15 | T1204 |
| 100201 | VirusTotal: 10-49 engines | 12 | T1204 |
| 100202 | VirusTotal: 1-9 engines | 8 | T1204 |
| 100300 | MyApp: Login failed | 8 | T1110 |
| 100301 | MyApp: Successful login | 3 | — |
| 100302 | MyApp: Brute force | 12 | T1110 |
| 100500 | Cowrie: Successful Honeypot Login | 8 | T1078 |
| 100501 | Cowrie: Failed Honeypot Login | 8 | T1110 |

## Skills Demonstrated

- Wazuh installation & configuration
- Custom XML rule writing
- FIM (File Integrity Monitoring)
- MITRE ATT&CK mapping
- Dashboard (OpenSearch) usage
- DQL query syntax
- Incident report writing
- Web attack detection
- Active Response configuration
- Multi-layer defense implementation
- Attack chain correlation
- Rule debugging & tuning
- Detection engineering & optimization
- Log noise reduction (99.8%)
- Honeypot Deployment (Cowrie)
- Docker Log Integration
- JSON Log Parsing & Custom Decoders

## Contact
- GitHub: Lekhraj Dheerani
- LinkedIn: https://www.linkedin.com/in/lekhraj-dheerani-910a91285/?isSelfProfile=true
