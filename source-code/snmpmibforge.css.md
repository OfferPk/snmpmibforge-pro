:root {  
  \--bg-dark: \#050811;  
  \--bg-card: \#0b1124;  
  \--bg-card-hover: \#101935;  
  \--border-color: \#1a264a;  
  \--border-focus: \#06b6d4;  
  \--text-main: \#f8fafc;  
  \--text-muted: \#94a3b8;  
  \--accent-cyan: \#06b6d4;  
  \--accent-blue: \#3b82f6;  
  \--accent-green: \#10b981;  
  \--accent-amber: \#f59e0b;  
  \--accent-red: \#ef4444;  
  \--accent-purple: \#8b5cf6;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-dark);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding: 24px;  
}

.app-container {  
  max-width: 1400px;  
  margin: 0 auto;  
  display: flex;  
  flex-direction: column;  
  gap: 20px;  
}

/\* Header \*/  
header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  padding: 18px 24px;  
  border-radius: 12px;  
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);  
}

.logo-area {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.logo-badge {  
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));  
  color: \#04101e;  
  font-weight: 900;  
  font-size: 13px;  
  padding: 6px 12px;  
  border-radius: 8px;  
  letter-spacing: 1px;  
}

header h1 {  
  font-size: 20px;  
  font-weight: 800;  
  letter-spacing: \-0.5px;  
}

.version-pill {  
  font-size: 11px;  
  background: \#0f2442;  
  color: var(--accent-cyan);  
  padding: 2px 8px;  
  border-radius: 12px;  
  font-weight: 600;  
  border: 1px solid rgba(6, 182, 212, 0.3);  
}

.header-badges {  
  display: flex;  
  gap: 10px;  
}

.badge {  
  font-size: 11px;  
  padding: 5px 12px;  
  border-radius: 20px;  
  font-weight: 700;  
  letter-spacing: 0.5px;  
}

.badge-cyan {  
  background: rgba(6, 182, 212, 0.15);  
  color: var(--accent-cyan);  
  border: 1px solid rgba(6, 182, 212, 0.4);  
}

.badge-green {  
  background: rgba(16, 185, 129, 0.15);  
  color: var(--accent-green);  
  border: 1px solid rgba(16, 185, 129, 0.4);  
}

/\* Presets Bar \*/  
.presets-bar {  
  display: flex;  
  align-items: center;  
  justify-content: space-between;  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  padding: 12px 20px;  
  border-radius: 10px;  
}

.preset-selector-group {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
}

.preset-selector-group label {  
  font-size: 13px;  
  font-weight: 600;  
  color: var(--text-muted);  
}

select, input\[type="text"\], input\[type="number"\], input\[type="password"\] {  
  background: \#060b18;  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 12px;  
  border-radius: 6px;  
  font-size: 13px;  
  font-family: inherit;  
  outline: none;  
  transition: border-color 0.2s;  
}

select:focus, input:focus {  
  border-color: var(--border-focus);  
}

/\* Main Grid \*/  
.main-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 20px;  
}

@media (max-width: 1024px) {  
  .main-grid {  
    grid-template-columns: 1fr;  
  }  
}

.card {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 20px;  
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);  
}

.card-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-bottom: 16px;  
  padding-bottom: 12px;  
  border-bottom: 1px solid var(--border-color);  
}

.card-title {  
  font-size: 15px;  
  font-weight: 700;  
  color: var(--text-main);  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

/\* Form Styles \*/  
.form-row {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 12px;  
  margin-bottom: 12px;  
}

.form-group {  
  display: flex;  
  flex-direction: column;  
  gap: 6px;  
}

.form-group label {  
  font-size: 11px;  
  font-weight: 700;  
  color: var(--text-muted);  
  text-transform: uppercase;  
  letter-spacing: 0.5px;  
}

/\* Toggles Grid \*/  
.toggle-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 10px;  
  margin-top: 10px;  
}

.toggle-card {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  background: \#080c1a;  
  border: 1px solid var(--border-color);  
  padding: 10px 14px;  
  border-radius: 8px;  
}

.toggle-label {  
  font-size: 12px;  
  font-weight: 600;  
}

.toggle-desc {  
  font-size: 10px;  
  color: var(--text-muted);  
}

/\* Scorecard Banner \*/  
.score-banner {  
  display: flex;  
  align-items: center;  
  gap: 20px;  
  background: \#080e22;  
  border: 1px solid var(--border-color);  
  padding: 16px 20px;  
  border-radius: 10px;  
  margin-bottom: 16px;  
}

.score-circle {  
  width: 72px;  
  height: 72px;  
  border-radius: 50%;  
  display: flex;  
  flex-direction: column;  
  align-items: center;  
  justify-content: center;  
  background: \#050811;  
  border: 3px solid var(--accent-green);  
  flex-shrink: 0;  
}

.score-num {  
  font-size: 24px;  
  font-weight: 800;  
  color: var(--accent-green);  
  line-height: 1;  
}

.score-label {  
  font-size: 9px;  
  font-weight: 700;  
  color: var(--text-muted);  
  text-transform: uppercase;  
}

.score-summary h3 {  
  font-size: 16px;  
  font-weight: 700;  
  margin-bottom: 4px;  
}

.score-summary p {  
  font-size: 12px;  
  color: var(--text-muted);  
}

/\* Checklist Items \*/  
.checklist {  
  display: flex;  
  flex-direction: column;  
  gap: 8px;  
  max-height: 380px;  
  overflow-y: auto;  
  padding-right: 4px;  
}

.check-item {  
  display: flex;  
  align-items: flex-start;  
  gap: 12px;  
  background: \#080c1a;  
  border: 1px solid var(--border-color);  
  padding: 10px 14px;  
  border-radius: 8px;  
  font-size: 12px;  
}

.check-icon {  
  font-size: 14px;  
  line-height: 1.2;  
}

.check-content {  
  flex: 1;  
}

.check-title-row {  
  display: flex;  
  justify-content: space-between;  
  margin-bottom: 2px;  
}

.check-title {  
  font-weight: 700;  
  color: var(--text-main);  
}

.check-pts {  
  font-weight: 700;  
  font-family: var(--font-mono);  
  font-size: 11px;  
}

.check-pts.pass { color: var(--accent-green); }  
.check-pts.warn { color: var(--accent-amber); }  
.check-pts.fail { color: var(--accent-red); }

.check-desc {  
  font-size: 11px;  
  color: var(--text-muted);  
}

/\* Key Localization Box \*/  
.key-box {  
  background: \#070c1e;  
  border: 1px solid var(--border-color);  
  padding: 14px;  
  border-radius: 8px;  
  margin-top: 14px;  
}

.code-snippet {  
  font-family: var(--font-mono);  
  font-size: 11px;  
  background: \#03050c;  
  padding: 8px 12px;  
  border-radius: 6px;  
  border: 1px solid var(--border-color);  
  margin-top: 6px;  
  overflow-x: auto;  
  white-space: pre-wrap;  
  word-break: break-all;  
}

/\* Synthesizer Tabs \*/  
.tabs-bar {  
  display: flex;  
  gap: 8px;  
  overflow-x: auto;  
}

.tab-btn {  
  background: \#070c1e;  
  border: 1px solid var(--border-color);  
  color: var(--text-muted);  
  padding: 8px 14px;  
  border-radius: 6px;  
  font-size: 12px;  
  font-weight: 600;  
  cursor: pointer;  
  white-space: nowrap;  
  transition: all 0.2s;  
}

.tab-btn:hover {  
  color: var(--text-main);  
  background: var(--bg-card-hover);  
}

.tab-btn.active {  
  background: \#0f2442;  
  color: var(--accent-cyan);  
  border-color: var(--accent-cyan);  
}

.code-output {  
  background: \#040714;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 16px;  
  font-family: var(--font-mono);  
  font-size: 12px;  
  color: \#38bdf8;  
  height: 280px;  
  overflow: auto;  
  white-space: pre;  
  margin-top: 14px;  
}

/\* Buttons \*/  
.btn {  
  background: \#0f1c3a;  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 16px;  
  border-radius: 6px;  
  font-size: 12px;  
  font-weight: 700;  
  cursor: pointer;  
  display: inline-flex;  
  align-items: center;  
  gap: 6px;  
  transition: all 0.2s;  
}

.btn:hover {  
  background: \#15264f;  
  border-color: var(--accent-cyan);  
}

.btn-primary {  
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));  
  color: \#04101e;  
  border: none;  
}

.btn-primary:hover {  
  filter: brightness(1.1);  
}

.copy-row {  
  display: flex;  
  justify-content: flex-end;  
  margin-top: 12px;  
}  
