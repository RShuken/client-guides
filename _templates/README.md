# OpenClaw Install Client Templates

## Standard Deliverables

Every OpenClaw Install client receives these three documents:

### 1. 📖 Getting Started Guide (`guide.html`)
- Introduction to the bot
- How to use it (Telegram, commands)
- Capabilities (what it can/can't do)
- System info (models, channels, server)

### 2. 🛡️ Security Report (`security-report.html`)
- Security score (A-F, 0-100)
- Issues fixed during audit
- Passed checks
- Optional improvements
- Summary

### 3. 📋 Work Done (`work-done.html`)
- Complete timeline of installation
- Every task with details
- Configuration values
- Results achieved

## How to Create Client Guides

1. Copy an existing client folder (e.g., `dan-garfinkel/` or `josh-v/`)
2. Rename to `<client-slug>/`
3. Update all three HTML files with client-specific info:
   - Bot name and username
   - Server details (GCP, Mac, VPS, etc.)
   - AI model configuration
   - Security audit results
   - Work performed

4. Commit and push:
   ```bash
   git add <client-slug>/
   git commit -m "Add <Client Name> client guides"
   git push origin main
   ```

5. Pages are live at:
   - `https://rshuken.github.io/client-guides/<client-slug>/guide.html`
   - `https://rshuken.github.io/client-guides/<client-slug>/security-report.html`
   - `https://rshuken.github.io/client-guides/<client-slug>/work-done.html`

## Client List

| Client | Bot Name | Platform | Date |
|--------|----------|----------|------|
| dan-garfinkel | Mini | GCP (Debian) | Jan 2026 |
| ian-ferguson | Vishnu | Mac Mini | Jan 2026 |
| josh-v | Woodhouse | Mac Studio | Feb 2026 |
