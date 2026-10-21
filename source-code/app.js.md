/\*\*  
 \* SNMPMibForge Pro \- Application Controller  
 \* Handles UI interactions, live SNMP policy mutation, security audits, key localization, and multi-vendor synthesis  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const engine \= new SNMPEngine();  
  const auditor \= new SNMPSecurityAuditor();  
  const synthesizer \= new SNMPConfigSynthesizer();

  let activeTab \= 'net\_snmp';  
  let currentConfig \= null;

  // DOM Elements  
  const presetSelect \= document.getElementById('presetSelect');  
  const usernameInput \= document.getElementById('usernameInput');  
  const secLevelSelect \= document.getElementById('secLevelSelect');  
  const authProtoSelect \= document.getElementById('authProtoSelect');  
  const authPassInput \= document.getElementById('authPassInput');  
  const privProtoSelect \= document.getElementById('privProtoSelect');  
  const privPassInput \= document.getElementById('privPassInput');  
  const engineIdInput \= document.getElementById('engineIdInput');  
  const subnetInput \= document.getElementById('subnetInput');

  const toggleVersion \= document.getElementById('toggleVersion');  
  const toggleReadOnly \= document.getElementById('toggleReadOnly');  
  const toggleVacm \= document.getElementById('toggleVacm');  
  const toggleInforms \= document.getElementById('toggleInforms');

  const localizedAuthKeySnippet \= document.getElementById('localizedAuthKeySnippet');  
  const engineIdBreakdownSnippet \= document.getElementById('engineIdBreakdownSnippet');

  const scoreNum \= document.getElementById('scoreNum');  
  const scoreGrade \= document.getElementById('scoreGrade');  
  const scoreCircle \= document.getElementById('scoreCircle');  
  const scoreHeadline \= document.getElementById('scoreHeadline');  
  const scoreSubtitle \= document.getElementById('scoreSubtitle');  
  const checklistContainer \= document.getElementById('checklistContainer');  
  const synthesizerOutput \= document.getElementById('synthesizerOutput');

  const btnExportConfig \= document.getElementById('btnExportConfig');  
  const btnCopyConfig \= document.getElementById('btnCopyConfig');  
  const tabButtons \= document.querySelectorAll('.tab-btn');

  // Built-in presets for offline / file:// protocol  
  const PRESETS \= {  
    'enterprise-snmpv3': {  
      profileName: "Enterprise Zero-Trust SNMPv3 Campus Core",  
      version: 3,  
      usm: {  
        username: "secops\_monitor",  
        securityLevel: "authPriv",  
        authProtocol: "SHA-256",  
        authPassphrase: "AuthPass\#2026\!EnterpriseKey\$",  
        privProtocol: "AES-256-CFB",  
        privPassphrase: "PrivPass\#2026\!EncryptedSecret%"  
      },  
      engine: {  
        engineId: "800000090300005e005301",  
        engineIdFormat: "enterprise\_hex",  
        enterpriseNumber: 9  
      },  
      transport: {  
        port: 161,  
        allowedSubnet: "10.200.0.0/24",  
        dtlsEnabled: false  
      },  
      vacm: {  
        groupName: "GRP\_SECOPS\_READONLY",  
        viewName: "VIEW\_SECURE\_TELEMETRY",  
        accessMode: "ReadOnly",  
        includedOids: \[  
          { oid: ".1.3.6.1.2.1.1", name: "system", desc: "SNMPv2-MIB System Info (Uptime, Description)" },  
          { oid: ".1.3.6.1.2.1.2", name: "interfaces", desc: "IF-MIB Interfaces & Traffic Counters" },  
          { oid: ".1.3.6.1.2.1.31", name: "ifXTable", desc: "64-bit High-Capacity Interface Octets" },  
          { oid: ".1.3.6.1.2.1.4", name: "ip", desc: "IP Routing & Address Tables" }  
        \],  
        excludedOids: \[  
          { oid: ".1.3.6.1.4.1.9.9.42", name: "ciscoIpSla", desc: "Sensitive Cisco IP SLA Probes" }  
        \]  
      },  
      notifications: {  
        type: "Inform",  
        targetHost: "10.200.0.50",  
        targetPort: 162,  
        securityLevel: "authPriv"  
      }  
    },  
    'cloud-prometheus': {  
      profileName: "Cloud Datacenter Infrastructure Telemetry",  
      version: 3,  
      usm: {  
        username: "prom\_collector",  
        securityLevel: "authPriv",  
        authProtocol: "SHA-512",  
        authPassphrase: "PromAuth\#9988\!CloudInfraPass@",  
        privProtocol: "AES-128",  
        privPassphrase: "PromPriv\#9988\!CloudInfraEnc\#"  
      },  
      engine: {  
        engineId: "80001f88803a6bc01481f349",  
        engineIdFormat: "enterprise\_hex",  
        enterpriseNumber: 8072  
      },  
      transport: {  
        port: 161,  
        allowedSubnet: "172.16.100.0/24",  
        dtlsEnabled: false  
      },  
      vacm: {  
        groupName: "GRP\_PROMETHEUS\_TELEMETRY",  
        viewName: "VIEW\_PROMETHEUS",  
        accessMode: "ReadOnly",  
        includedOids: \[  
          { oid: ".1.3.6.1.2.1.1", name: "system", desc: "System Identifiers & Hostname" },  
          { oid: ".1.3.6.1.2.1.2", name: "interfaces", desc: "Network Interfaces Bandwidth" },  
          { oid: ".1.3.6.1.2.1.25", name: "hostResources", desc: "Host Resources (CPU, Memory, Storage)" }  
        \],  
        excludedOids: \[\]  
      },  
      notifications: {  
        type: "Trap",  
        targetHost: "172.16.100.10",  
        targetPort: 162,  
        securityLevel: "authPriv"  
      }  
    },  
    'vulnerable-snmpv2': {  
      profileName: "Hazardous Legacy Insecure SNMPv2c Configuration",  
      version: 2,  
      usm: {  
        username: "public",  
        securityLevel: "noAuthNoPriv",  
        authProtocol: "None",  
        authPassphrase: "",  
        privProtocol: "None",  
        privPassphrase: ""  
      },  
      engine: {  
        engineId: "",  
        engineIdFormat: "none",  
        enterpriseNumber: 0  
      },  
      transport: {  
        port: 161,  
        allowedSubnet: "0.0.0.0/0",  
        dtlsEnabled: false  
      },  
      vacm: {  
        groupName: "public\_group",  
        viewName: "all\_internet",  
        accessMode: "ReadWrite",  
        includedOids: \[  
          { oid: ".1.3.6.1", name: "internet", desc: "Entire Internet MIB Root (Unrestricted)" }  
        \],  
        excludedOids: \[\]  
      },  
      notifications: {  
        type: "Trap",  
        targetHost: "0.0.0.0",  
        targetPort: 162,  
        securityLevel: "noAuthNoPriv"  
      }  
    }  
  };

  function loadPreset(key) {  
    if (PRESETS\[key\]) {  
      currentConfig \= JSON.parse(JSON.stringify(PRESETS\[key\]));  
      syncInputsFromConfig();  
      recalculateAndRender();  
    }  
  }

  function syncInputsFromConfig() {  
    usernameInput.value \= currentConfig.usm?.username || 'secops\_monitor';  
    secLevelSelect.value \= currentConfig.usm?.securityLevel || 'authPriv';  
    authProtoSelect.value \= currentConfig.usm?.authProtocol || 'SHA-256';  
    authPassInput.value \= currentConfig.usm?.authPassphrase || '';  
    privProtoSelect.value \= currentConfig.usm?.privProtocol || 'AES-256-CFB';  
    privPassInput.value \= currentConfig.usm?.privPassphrase || '';  
    engineIdInput.value \= currentConfig.engine?.engineId || '800000090300005e005301';  
    subnetInput.value \= currentConfig.transport?.allowedSubnet || '10.200.0.0/24';

    toggleVersion.checked \= currentConfig.version \=== 3;  
    toggleReadOnly.checked \= currentConfig.vacm?.accessMode \=== 'ReadOnly';  
    toggleVacm.checked \= \!((currentConfig.vacm?.includedOids || \[\]).some(o \=\> o.oid \=== '.1.3.6.1'));  
    toggleInforms.checked \= currentConfig.notifications?.type \=== 'Inform';  
  }

  function syncConfigFromInputs() {  
    currentConfig.usm \= currentConfig.usm || {};  
    currentConfig.usm.username \= usernameInput.value.trim() || 'secops';  
    currentConfig.usm.securityLevel \= secLevelSelect.value;  
    currentConfig.usm.authProtocol \= authProtoSelect.value;  
    currentConfig.usm.authPassphrase \= authPassInput.value;  
    currentConfig.usm.privProtocol \= privProtoSelect.value;  
    currentConfig.usm.privPassphrase \= privPassInput.value;

    currentConfig.engine \= currentConfig.engine || {};  
    currentConfig.engine.engineId \= engineIdInput.value.trim();

    currentConfig.transport \= currentConfig.transport || {};  
    currentConfig.transport.allowedSubnet \= subnetInput.value.trim() || '10.200.0.0/24';

    currentConfig.version \= toggleVersion.checked ? 3 : 2;  
    currentConfig.vacm \= currentConfig.vacm || {};  
    currentConfig.vacm.accessMode \= toggleReadOnly.checked ? 'ReadOnly' : 'ReadWrite';

    if (toggleVacm.checked) {  
      if ((currentConfig.vacm.includedOids || \[\]).some(o \=\> o.oid \=== '.1.3.6.1')) {  
        currentConfig.vacm.includedOids \= \[  
          { oid: ".1.3.6.1.2.1.1", name: "system", desc: "System Info" },  
          { oid: ".1.3.6.1.2.1.2", name: "interfaces", desc: "Network Interfaces" }  
        \];  
      }  
    } else {  
      currentConfig.vacm.includedOids \= \[  
        { oid: ".1.3.6.1", name: "internet", desc: "Unrestricted Internet Root" }  
      \];  
    }

    currentConfig.notifications \= currentConfig.notifications || {};  
    currentConfig.notifications.type \= toggleInforms.checked ? 'Inform' : 'Trap';  
  }

  function recalculateAndRender() {  
    // 1\. Calculate & Render Key Localization Sandbox  
    const authPass \= authPassInput.value;  
    const engineId \= engineIdInput.value;  
    const localKey \= engine.localizeKeySHA256(authPass, engineId);  
    localizedAuthKeySnippet.textContent \= localKey || '(Provide Auth Passphrase & EngineID to calculate localized key)';

    const eidBreakdown \= engine.parseEngineId(engineId);  
    if (eidBreakdown.valid) {  
      engineIdBreakdownSnippet.textContent \= \`Enterprise \#\${eidBreakdown.enterpriseNum} | Format: \${eidBreakdown.format} | Octets: \${eidBreakdown.octetLength} bytes\`;  
    } else {  
      engineIdBreakdownSnippet.textContent \= eidBreakdown.reason || 'Invalid EngineID';  
    }

    // 2\. Run 10-Point Security Audit  
    const audit \= auditor.auditConfig(currentConfig);  
    renderScorecard(audit);

    // 3\. Render Synthesizer  
    renderSynthesizer();  
  }

  function renderScorecard(audit) {  
    scoreNum.textContent \= audit.totalScore;  
    scoreGrade.textContent \= audit.grade;  
    scoreCircle.style.borderColor \= audit.gradeColor;  
    scoreNum.style.color \= audit.gradeColor;

    if (audit.grade \=== 'A+') {  
      scoreHeadline.textContent \= 'Zero-Trust Verified • RFC 3414/7860 Compliant';  
      scoreHeadline.style.color \= '\#10b981';  
      scoreSubtitle.textContent \= 'SNMPv3 authPriv, SHA-256/512 auth, AES-128/256 encryption, restricted VACM, and Inform notifications.';  
    } else if (audit.grade \=== 'B') {  
      scoreHeadline.textContent \= 'Production Solid • Minor Hardening';  
      scoreHeadline.style.color \= '\#06b6d4';  
      scoreSubtitle.textContent \= 'Core USM encryption active. Consider tightening VACM MIB view scope or upgrading ciphers.';  
    } else if (audit.grade \=== 'C') {  
      scoreHeadline.textContent \= 'Suboptimal SNMP Posture';  
      scoreHeadline.style.color \= '\#f59e0b';  
      scoreSubtitle.textContent \= 'authNoPriv (unencrypted telemetry) or open internet MIB views detected.';  
    } else {  
      scoreHeadline.textContent \= 'Critical SNMP Defect • Immediate Risk';  
      scoreHeadline.style.color \= '\#ef4444';  
      scoreSubtitle.textContent \= 'Legacy SNMPv1/v2c cleartext community strings, read-write enabled, or weak DES ciphers in use\!';  
    }

    checklistContainer.innerHTML \= '';  
    audit.results.forEach(item \=\> {  
      const el \= document.createElement('div');  
      el.className \= 'check-item';

      const icon \= item.status \=== 'PASS' ? '✅' : (item.status \=== 'WARN' ? '⚠️' : '❌');  
      const ptsClass \= item.status.toLowerCase();

      el.innerHTML \= \`  
        \<div class="check-icon"\>\${icon}\</div\>  
        \<div class="check-content"\>  
          \<div class="check-title-row"\>  
            \<span class="check-title"\>\${item.title}\</span\>  
            \<span class="check-pts \${ptsClass}"\>\${item.score}/\${item.maxScore} pts\</span\>  
          \</div\>  
          \<div class="check-desc"\>\${item.details}\</div\>  
        \</div\>  
      \`;  
      checklistContainer.appendChild(el);  
    });  
  }

  function renderSynthesizer() {  
    synthesizerOutput.textContent \= synthesizer.synthesize(currentConfig, activeTab);  
  }

  // Event Listeners  
  presetSelect.addEventListener('change', (e) \=\> {  
    loadPreset(e.target.value);  
  });

  \[usernameInput, secLevelSelect, authProtoSelect, authPassInput, privProtoSelect, privPassInput, engineIdInput, subnetInput\].forEach(inp \=\> {  
    inp.addEventListener('input', () \=\> {  
      syncConfigFromInputs();  
      recalculateAndRender();  
    });  
  });

  \[toggleVersion, toggleReadOnly, toggleVacm, toggleInforms\].forEach(tog \=\> {  
    tog.addEventListener('change', () \=\> {  
      syncConfigFromInputs();  
      recalculateAndRender();  
    });  
  });

  btnExportConfig.addEventListener('click', () \=\> {  
    const netSnmpContent \= synthesizer.generateNetSNMP(currentConfig);  
    const dataStr \= "data:text/plain;charset=utf-8," \+ encodeURIComponent(netSnmpContent);  
    const downloadAnchor \= document.createElement('a');  
    downloadAnchor.setAttribute("href", dataStr);  
    downloadAnchor.setAttribute("download", \`snmpd-\${currentConfig.usm?.username || 'config'}.conf\`);  
    document.body.appendChild(downloadAnchor);  
    downloadAnchor.click();  
    downloadAnchor.remove();  
  });

  btnCopyConfig.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(synthesizerOutput.textContent).then(() \=\> {  
      showToast('Configuration copied to clipboard\!');  
    });  
  });

  tabButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeTab \= btn.getAttribute('data-tab');  
      renderSynthesizer();  
    });  
  });

  function showToast(msg) {  
    const toast \= document.createElement('div');  
    toast.textContent \= msg;  
    toast.style.position \= 'fixed';  
    toast.style.bottom \= '24px';  
    toast.style.right \= '24px';  
    toast.style.background \= '\#10b981';  
    toast.style.color \= '\#04101e';  
    toast.style.padding \= '10px 18px';  
    toast.style.borderRadius \= '6px';  
    toast.style.fontWeight \= '700';  
    toast.style.fontSize \= '12px';  
    toast.style.zIndex \= '9999';  
    toast.style.boxShadow \= '0 4px 12px rgba(0,0,0,0.5)';  
    document.body.appendChild(toast);  
    setTimeout(() \=\> toast.remove(), 2500);  
  }

  // Initial Load  
  loadPreset('enterprise-snmpv3');  
});  
