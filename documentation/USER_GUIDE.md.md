\# SNMPMibForge Pro \- User Guide & DevSecOps Integration Manual

\#\# 1\. Web Studio Quickstart

1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an industry preset from the top dropdown:  
   \- \*\*Enterprise Zero-Trust SNMPv3 Campus Core (A+)\*\*: SNMPv3 \`authPriv\`, SHA-256 auth, AES-256-CFB privacy, restricted telemetry VACM view, and Inform notifications.  
   \- \*\*Cloud Datacenter Infrastructure Telemetry (A+)\*\*: Prometheus scraper preset with SHA-512, AES-128, Host Resources MIB, and Net-SNMP configuration.  
   \- \*\*Hazardous Legacy Insecure SNMPv2c (F)\*\*: Demonstrates vulnerabilities from plaintext community \`public\`, \`ReadWrite\` mode, and open internet root.  
3\. Review the \*\*Live RFC 3414 Localized Cryptographic Key Sandbox\*\*:  
   \- Computes the exact localized SHA-256 key derived from your passphrase and authoritative EngineID.  
4\. Review the \*\*10-Point Zero-Trust SNMPv3 Security Scorecard\*\* for compliance and security posture.  
5\. Switch between synthesizer tabs:  
   \- \*\*Net-SNMP\*\*: Copy production \`snmpd.conf\` createUser and rouser directives.  
   \- \*\*Cisco Catalyst / IOS-XE\*\*: Copy switch & router \`snmp-server\` configuration commands.  
   \- \*\*Juniper Junos\*\*: Copy Junos hierarchical \`set\` commands.  
   \- \*\*Prometheus SNMP Exporter\*\*: Copy \`snmp.yml\` YAML scrape configuration.  
   \- \*\*Python pysnmp\*\*: Copy test telemetry script.

\---

\#\# 2\. DevSecOps CLI Tool (\`snmpmibforge-cli.py\`)

The standalone Python CLI tool requires zero external dependencies and runs on Python 3.8+.

\#\#\# 2.1 Linting SNMP Configurations  
\`\`\`bash  
python3 build/snmpmibforge-cli.py lint \--preset presets/enterprise-zerotrust-snmpv3.json  
\`\`\`  
Output:  
\`\`\`  
\[LINT PASSED\] SNMP configuration is structurally valid (0 errors).  
\`\`\`

\#\#\# 2.2 Running CI/CD Pipeline Security Audits  
\`\`\`bash  
\# Clean production configuration \-\> Exits code 0  
python3 build/snmpmibforge-cli.py audit \--preset presets/enterprise-zerotrust-snmpv3.json \--strict

\# Insecure configuration \-\> Exits code 1 (Pipeline Fails)  
python3 build/snmpmibforge-cli.py audit \--preset presets/vulnerable-legacy-snmpv2.json \--strict  
\`\`\`

\#\#\# 2.3 Calculating Localized Keys (RFC 3414 / RFC 7860\)  
\`\`\`bash  
python3 build/snmpmibforge-cli.py localize-key \\  
    \--password "AuthPass\#2026\!EnterpriseKey\$" \\  
    \--engine-id "800000090300005e005301" \\  
    \--hash sha256  
\`\`\`

\#\#\# 2.4 Synthesizing Net-SNMP Configuration  
\`\`\`bash  
python3 build/snmpmibforge-cli.py synthesize \--preset presets/enterprise-zerotrust-snmpv3.json \-f net-snmp  
\`\`\`

\---

\#\# 3\. GitHub Actions CI/CD Pipeline Template

Add this workflow to your NetDevOps or Ansible infrastructure repository (\`.github/workflows/snmp-security-gate.yml\`) to validate SNMP policies before pushing to routers and firewalls:

\`\`\`yaml  
name: SNMPv3 Security & USM Policy CI/CD Gate

on:  
  pull\_request:  
    paths:  
      \- 'snmp/\*\*'  
      \- 'net-snmp/\*\*'  
      \- 'network/telemetry/\*\*'

jobs:  
  snmp-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- name: Checkout Code  
        uses: actions/checkout@v4

      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'

      \- name: Lint SNMP Configuration  
        run: |  
          python3 build/snmpmibforge-cli.py lint \--preset presets/enterprise-zerotrust-snmpv3.json

      \- name: Audit Zero-Trust SNMPv3 Compliance (Strict Gate)  
        run: |  
          python3 build/snmpmibforge-cli.py audit \--preset presets/enterprise-zerotrust-snmpv3.json \--strict  
\`\`\`  
