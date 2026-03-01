# SOCRadar Multi-Tenant Incident Management for Jira Service Management

[![Atlassian Marketplace](https://img.shields.io/badge/Atlassian-Marketplace-blue)](https://marketplace.atlassian.com/)
[![Forge](https://img.shields.io/badge/Forge-App-orange)](https://developer.atlassian.com/platform/forge/)
[![Security](https://img.shields.io/badge/Security-A+-green)](https://www.socradar.io)
[![License](https://img.shields.io/badge/License-ISC-blue.svg)](LICENSE)

> **BETA VERSION** - Early access release for evaluation and feedback

Seamlessly integrate SOCRadar's security intelligence platform with Jira Service Management to automatically create and manage security incidents based on real-time threat alerts.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Security](#security)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Support](#support)
- [License](#license)

---

## 🎯 Overview

**SOCRadar Multi-Tenant Incident Management** is a Forge-based Atlassian app that bridges the gap between SOCRadar's threat intelligence platform and Jira Service Management. This integration automates the creation of security incidents in JSM based on real-time security alarms from SOCRadar, enabling security teams to:

- **Automate incident response** workflows
- **Centralize security operations** in Jira Service Management
- **Reduce manual data entry** and human error
- **Improve response times** with real-time synchronization
- **Maintain audit trails** of all security incidents

### Key Benefits

✅ **Zero Code Deployment** - Forge-based app requiring no server infrastructure  
✅ **Enterprise Security** - All credentials encrypted using Forge secret storage  
✅ **Multi-Tenant Support** - Designed for SOCRadar multi-tenant architecture  
✅ **Automated Sync** - Hourly scheduled synchronization + optional webhook triggers  
✅ **Admin Controls** - Granular permission-based configuration and management  

---

## ✨ Features

### 🔄 Automated Synchronization

- **Scheduled Sync**: Hourly automatic synchronization of new security alarms
- **Real-time Webhook**: Optional webhook endpoint for instant incident creation
- **Smart Deduplication**: Tracks last synced alarm ID to prevent duplicates
- **Batch Processing**: Efficiently handles multiple alarms per sync cycle

### 🎨 Admin Configuration UI

- **React-based Admin Panel**: Modern, intuitive configuration interface
- **Project Selection**: Auto-populated Jira project dropdown
- **Connection Testing**: One-click SOCRadar API connection validation
- **Sync Logs**: Real-time view of synchronization history and results
- **Manual Sync**: On-demand synchronization with dry-run capability

### 🛡️ Security Features

- **Encrypted Storage**: API keys stored using `storage.setSecret()`
- **Permission Gating**: Admin-only access via Jira ADMINISTER permission
- **Webhook Authentication**: Configurable token-based webhook security
- **Input Sanitization**: HTML cleaning and text validation on all inputs
- **Masked Credentials**: API keys never exposed in UI (displayed as `****xxxx`)
- **Audit Logging**: Comprehensive sync logs stored in Forge storage

### 🎯 Incident Management

- **Priority Mapping**: Automatic Jira priority assignment based on SOCRadar severity
  - CRITICAL → Highest
  - HIGH → High
  - MEDIUM → Medium
  - LOW → Low
  - INFO → Lowest
- **Auto-Assignment**: Optional automatic assignee configuration
- **Labels**: Automatic `socradar` and `security-alert` labels
- **Rich Descriptions**: ADF-formatted descriptions with sanitized HTML content
- **Custom Issue Types**: Configurable issue type (Task, Bug, Incident, etc.)

---

## 🏗️ Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     SOCRadar Platform                        │
│                 (Multi-Tenant Security API)                  │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ REST API (HTTPS)
                        │ + Optional Webhook
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Forge App (Atlassian)                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Scheduled Trigger (Hourly)                          │   │
│  │  • Fetches new alarms from SOCRadar                  │   │
│  │  • Creates Jira issues via asApp()                   │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Webhook Handler (Optional)                          │   │
│  │  • Receives real-time notifications                  │   │
│  │  • Token-based authentication                        │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Admin UI (React)                                    │   │
│  │  • Configuration management                          │   │
│  │  • Connection testing                                │   │
│  │  • Manual sync + dry run                             │   │
│  │  • Sync logs viewer                                  │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Forge Secret Storage                                │   │
│  │  • Encrypted API keys                                │   │
│  │  • Configuration data                                │   │
│  │  • Sync state (last alarm ID)                        │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ Jira REST API v3
                        ▼
┌─────────────────────────────────────────────────────────────┐
│              Jira Service Management                         │
│                    (Issues Created)                          │
└─────────────────────────────────────────────────────────────┘
```

### Component Breakdown

#### 1. **Backend Functions** (`src/index.js`)

**Resolvers** (UI ↔ Backend communication):
- `saveConfig`: Store encrypted configuration (admin only)
- `getConfig`: Retrieve configuration with masked API key
- `testConnection`: Validate SOCRadar API connectivity
- `runManualSync`: Trigger manual synchronization (admin only)
- `getSyncLogs`: Retrieve synchronization history
- `resetLastAlarmId`: Reset sync state
- `getProjects`: Fetch available Jira projects

**Scheduled Trigger**:
- `scheduledSync`: Runs hourly, fetches new alarms and creates Jira issues

**Webhook Handler**:
- `webhookHandler`: Processes external webhook calls from SOCRadar

#### 2. **Frontend UI** (`static/frontend/`)

React-based admin configuration panel with:
- Configuration form
- Connection test button
- Manual sync controls
- Sync logs display
- Project selection dropdown

#### 3. **Security Layer**

- **Authentication**: All resolvers validate `context.accountId`
- **Authorization**: Admin endpoints check `ADMINISTER` permission via Jira API
- **Secret Storage**: `storage.setSecret()` for API keys
- **Input Validation**: HTML sanitization, length limits, required field checks

---

## 🔒 Security

### Implemented Security Measures

#### Authentication & Authorization

| Feature | Implementation | Status |
|---------|---------------|--------|
| User Authentication | `context.accountId` validation | ✅ |
| Admin Permissions | Jira ADMINISTER permission check | ✅ |
| asUser() Usage | All user-facing resolvers | ✅ |
| asApp() Scope | Limited to scheduled triggers only | ✅ |
| Webhook Authentication | Token-based (`x-socradar-webhook-token`) | ✅ |

#### Data Security

| Feature | Implementation | Status |
|---------|---------------|--------|
| Secret Storage | `storage.setSecret()` for API keys | ✅ |
| TLS Encryption | HTTPS only (platform.socradar.com) | ✅ |
| Data at Rest | Forge encrypted storage | ✅ |
| Least Privilege | Minimal required scopes | ✅ |
| No Sensitive Logging | Only alarm IDs and issue keys logged | ✅ |

#### Application Security

| Feature | Implementation | Status |
|---------|---------------|--------|
| Input Validation | Required field checks, type validation | ✅ |
| HTML Sanitization | `cleanHtml()` function removes tags | ✅ |
| Text Truncation | 3000 char limit on descriptions | ✅ |
| API Key Masking | Display as `****xxxx` in UI | ✅ |
| Dependency Scanning | npm audit (SCA) | ✅ |

### Required Permissions

```yaml
permissions:
  scopes:
    - read:jira-work      # Read Jira issues and projects
    - write:jira-work     # Create and update Jira issues
    - read:jira-user      # Read user information for assignees
    - storage:app         # Access Forge app storage
  external:
    fetch:
      backend:
        - platform.socradar.com  # SOCRadar API access
```

### Security Contact

For security issues or vulnerabilities, please contact:  
📧 **integration@socradar.io**

---

## 📦 Installation

### Prerequisites

- Jira Service Management Cloud instance
- SOCRadar Multi-Tenant account with API access
- Jira Administrator permissions
- Node.js 18+ (for development)
- Forge CLI (for deployment)

### From Atlassian Marketplace

1. **Navigate to Marketplace**
   ```
   Jira Settings → Apps → Find new apps
   ```

2. **Search for "SOCRadar"**
   ```
   Search: "SOCRadar Multi-Tenant Incident Management"
   ```

3. **Install the App**
   - Click "Get app" or "Try it free"
   - Accept permissions
   - Wait for installation to complete

### Manual Installation (Development)

```bash
# Clone repository
git clone https://github.com/your-org/socradar-jira-jsm.git
cd socradar-jira-jsm

# Install dependencies
npm install --ignore-scripts

# Build frontend
cd static/frontend
npm install
npm run build
cd ../..

# Login to Forge
forge login

# Deploy to development
forge deploy --no-verify

# Install to your site
forge install

# Deploy to production
forge deploy --environment production --no-verify
```

---

## ⚙️ Configuration

### Step 1: Access Admin Panel

1. Navigate to **Jira Settings** (⚙️)
2. Go to **Apps** → **SOCRadar Integration Settings**

### Step 2: Configure SOCRadar Connection

Fill in the following fields:

| Field | Description | Required | Example |
|-------|-------------|----------|---------|
| **SOCRadar Tenant ID** | Your multi-tenant company ID | ✅ | `123456` |
| **SOCRadar API Key** | API key from SOCRadar dashboard | ✅ | `sk_live_...` |
| **SOCRadar Base URL** | API endpoint (default is pre-filled) | ❌ | `https://platform.socradar.com` |

### Step 3: Configure Jira Settings

| Field | Description | Required | Example |
|-------|-------------|----------|---------|
| **Jira Project** | Target project for incidents | ✅ | `SEC` (Security) |
| **Issue Type** | Type of issues to create | ❌ | `Incident` (default: Task) |
| **Default Assignee** | Auto-assign incidents | ❌ | John Doe |
| **Show SOCRadar Source** | Include "SOCRadar" in summary | ❌ | ✅ (recommended) |

### Step 4: Test Connection

Click **"Test Connection"** to verify:
- SOCRadar API connectivity
- API key validity
- Multi-tenant access
- Number of available alarms

Expected response:
```
✅ Connection successful. 142 open alarms found.
```

### Step 5: Initial Sync

**Option A: Dry Run (Recommended)**
```
1. Click "Run Manual Sync"
2. Enable "Dry Run Mode"
3. Review which alarms would be imported
4. Check logs for any issues
```

**Option B: Full Sync**
```
1. Click "Run Manual Sync"
2. Wait for completion
3. Verify issues created in Jira project
```

### Step 6: Configure Webhook (Optional)

For real-time synchronization:

1. **Get Webhook URL** from app settings
2. **Configure in SOCRadar**:
   - Navigate to SOCRadar → Settings → Webhooks
   - Add new webhook with URL from step 1
   - Set webhook token (custom secret)
3. **Update App Configuration**:
   - Add webhook token to app settings
   - Enable webhook authentication

---

## 🚀 Usage

### Automatic Synchronization

Once configured, the app automatically:
- ✅ Runs hourly scheduled sync
- ✅ Fetches new alarms from SOCRadar (OPEN status only)
- ✅ Creates Jira issues with proper priority mapping
- ✅ Tracks last synced alarm ID to prevent duplicates
- ✅ Logs all synchronization activity

### Manual Synchronization

#### Full Sync
```
Admin Panel → Run Manual Sync → Execute
```
Creates Jira issues for all new alarms since last sync.

#### Dry Run
```
Admin Panel → Run Manual Sync → Enable "Dry Run" → Execute
```
Shows which alarms would be synced without creating issues.

### Viewing Sync Logs

```
Admin Panel → Sync Logs Tab
```

Log entries include:
- Timestamp
- Sync type (scheduled/manual/dry-run)
- Alarms found
- Issues created/failed
- Last alarm ID processed

### Resetting Sync State

To re-import all alarms:
```
Admin Panel → Advanced → Reset Last Alarm ID
```
⚠️ **Warning**: This will reimport ALL alarms on next sync, potentially creating duplicates.

---

## 📚 API Reference

### SOCRadar API Integration

**Endpoint**: `/api/v1/multi-tenant/{tenant_id}/incidents`

**Request Parameters**:
```javascript
{
  key: 'api_key',
  status: 'OPEN',
  limit: 50,
  page: 1,
  start_date: 1234567890  // Optional: Unix epoch
}
```

**Response Format**:
```json
{
  "data": {
    "alarms": [
      {
        "alarm_id": 12345,
        "alarm_text": "<p>Security incident description...</p>",
        "severity": "HIGH",
        "alarm_type_details": {
          "alarm_generic_title": "Suspicious Activity Detected",
          "severity": "HIGH"
        }
      }
    ],
    "total_pages": 5,
    "total_records": 142
  }
}
```

### Jira Issue Creation

**API**: Jira REST API v3  
**Method**: `POST /rest/api/3/issue`

**Issue Fields**:
```json
{
  "fields": {
    "project": { "key": "SEC" },
    "summary": "SOCRadar - 12345 - Suspicious Activity Detected",
    "description": {
      "type": "doc",
      "version": 1,
      "content": [
        {
          "type": "paragraph",
          "content": [
            { "type": "text", "text": "Sanitized alarm description..." }
          ]
        }
      ]
    },
    "issuetype": { "name": "Incident" },
    "priority": { "name": "High" },
    "assignee": { "accountId": "..." },
    "labels": ["socradar", "security-alert"]
  }
}
```

### Webhook Payload

**Endpoint**: `https://your-site.atlassian.net/gateway/api/webhook/...`  
**Method**: `POST`  
**Headers**:
```
X-SOCRadar-Webhook-Token: your_webhook_secret
Content-Type: application/json
```

**Body**: Empty (triggers sync on receipt)

---

## 🔧 Troubleshooting

### Common Issues

#### 1. "SOCRadar configuration not found"

**Cause**: App not configured yet  
**Solution**:
```
1. Navigate to Admin Panel
2. Fill in all required fields
3. Click "Save Configuration"
4. Test connection
```

#### 2. "Failed to create issue for alarm X"

**Possible Causes**:
- Invalid Jira project key
- Missing issue type
- Insufficient permissions
- Network connectivity

**Solution**:
```
1. Check sync logs for specific error message
2. Verify project exists and is accessible
3. Confirm issue type is available in project
4. Test connection to both SOCRadar and Jira
```

#### 3. "SOCRadar API connection failed"

**Possible Causes**:
- Invalid API key
- Incorrect tenant ID
- Network firewall blocking requests
- SOCRadar API downtime

**Solution**:
```
1. Verify API key in SOCRadar dashboard
2. Confirm tenant ID is correct
3. Check Atlassian network status
4. Contact SOCRadar support
```

#### 4. Duplicate Issues Created

**Cause**: Last alarm ID was reset or corrupted  
**Solution**:
```
1. Check sync logs for last alarm ID
2. If incorrect, manually update via admin panel
3. Run dry run to verify before full sync
```

#### 5. Webhook Not Working

**Possible Causes**:
- Incorrect webhook URL
- Mismatched webhook token
- Firewall blocking incoming requests

**Solution**:
```
1. Verify webhook URL in SOCRadar matches app URL
2. Confirm webhook token matches in both systems
3. Test webhook with manual trigger
4. Check app logs for authentication errors
```

### Debug Mode

Enable debug logging:
```bash
# Development environment
forge tunnel --debug

# Check logs
forge logs --tail
```

### Getting Help

1. **Check Sync Logs**: Admin Panel → Sync Logs
2. **Review App Logs**: `forge logs --tail`
3. **Test Components**:
   - SOCRadar connection
   - Jira project access
   - Webhook authentication
4. **Contact Support**: integration@socradar.io

---

## 📊 Performance & Limits

### Sync Performance

| Metric | Value | Notes |
|--------|-------|-------|
| Alarms per Page | 50 | SOCRadar API limit |
| Sync Frequency | 1 hour | Scheduled trigger interval |
| Max Log Entries | 100 | Rotating log storage |
| API Call Timeout | 30s | Forge default |

### Jira Limits

| Limit | Value | Impact |
|-------|-------|--------|
| Summary Length | 255 chars | Auto-truncated |
| Description Length | 3000 chars | Truncated with "...[TRUNCATED]" |
| Labels per Issue | Unlimited | 2 default labels added |
| API Rate Limit | Varies | Handled by Forge |

### Storage Limits

| Item | Storage Type | Encryption |
|------|--------------|------------|
| API Keys | `storage.setSecret()` | ✅ Encrypted |
| Configuration | `storage.setSecret()` | ✅ Encrypted |
| Sync Logs | `storage.get/set()` | ❌ Not encrypted |
| Last Alarm ID | `storage.get/set()` | ❌ Not encrypted |

---

## 🗺️ Roadmap

### Planned Features

- [ ] **Bi-directional Sync**: Update SOCRadar when Jira status changes
- [ ] **Custom Field Mapping**: Map SOCRadar fields to custom Jira fields
- [ ] **Filtering Rules**: Sync only specific alarm types or severities
- [ ] **Email Notifications**: Alert admins on sync failures
- [ ] **Dashboard Widget**: Display SOCRadar metrics in Jira dashboard
- [ ] **Bulk Operations**: Batch update/close incidents
- [ ] **Advanced Logging**: Exportable CSV sync reports

### Beta Feedback

We're actively seeking feedback on:
- Configuration experience
- Sync performance
- Error handling
- UI/UX improvements
- Feature requests

Please submit feedback to: **integration@socradar.io**

---

## 🤝 Support

### Documentation

- [Forge Documentation](https://developer.atlassian.com/platform/forge/)
- [SOCRadar API Docs](https://platform.socradar.com/docs/api)
- [Jira REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

### Community

- [Atlassian Community](https://community.atlassian.com)
- [SOCRadar Support Portal](https://support.socradar.com)

### Contact

For technical support or questions:

📧 **Email**: integration@socradar.io  
🌐 **Website**: https://www.socradar.io  
💬 **Support Portal**: https://support.socradar.com

### Enterprise Support

For enterprise customers, dedicated support is available:
- Priority email support
- Custom integration assistance
- Training and onboarding
- SLA-backed response times

Contact your SOCRadar account manager for details.

---

## 📄 License

Copyright © 2025 SOCRadar Cyber Intelligence Inc.

Licensed under the ISC License.

---

## 🙏 Acknowledgments

- Built on [Atlassian Forge](https://developer.atlassian.com/platform/forge/)
- Powered by [SOCRadar](https://www.socradar.io) security intelligence
- UI components from [React](https://react.dev/)

---

## 📝 Changelog

### Version 1.0.0-beta (Current)

**Released**: March 2025

**New Features**:
- ✨ Initial beta release
- ✨ Scheduled hourly synchronization
- ✨ React-based admin configuration UI
- ✨ Manual sync with dry-run mode
- ✨ Webhook support for real-time sync
- ✨ Priority mapping from SOCRadar severity
- ✨ Sync logs viewer
- ✨ Connection testing

**Security**:
- 🔒 Encrypted secret storage for API keys
- 🔒 Admin permission validation
- 🔒 Webhook token authentication
- 🔒 Input sanitization and validation
- 🔒 asUser()/asApp() proper scoping

**Technical**:
- ⚡ Node.js 22.x runtime
- ⚡ Jira REST API v3 integration
- ⚡ ADF-formatted issue descriptions
- ⚡ HTML sanitization with cleanHtml()
- ⚡ Pagination support for large alarm sets

---

**Made with ❤️ by SOCRadar**

*Protecting organizations from cyber threats, one alert at a time.*
