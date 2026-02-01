# NextStepAI - Product Specification

> **Version:** 1.0.0-draft
> **Status:** Planning Phase
> **Last Updated:** 2026-02-01

---

## Table of Contents

1. [Goals & Non-Goals](#1-goals--non-goals)
2. [User Experiences](#2-user-experiences)
3. [Functional Requirements](#3-functional-requirements)
4. [Non-Functional Requirements](#4-non-functional-requirements)
5. [Data Model](#5-data-model)
6. [APIs / Interfaces](#6-apis--interfaces)
7. [Security Specification](#7-security-specification)
8. [Acceptance Criteria Checklist](#8-acceptance-criteria-checklist)

---

## 1. Goals & Non-Goals

### 1.1 Product Vision

NextStepAI is a **security-first AI agent for macOS** that enables users to automate complex workflows through natural language commands while maintaining complete control over sensitive operations. It embodies the principle: **"Access control before intelligence."**

### 1.2 Goals

| ID | Goal | Success Metric |
|----|------|----------------|
| G1 | **Secure-by-default agent automation** | Zero credential leaks in production use |
| G2 | **Natural language workflow execution** | 90% of common workflows completable via voice/text |
| G3 | **Transparent operation** | User can understand and approve every action |
| G4 | **Browser automation with safety** | Login workflows on allowlisted sites with user control |
| G5 | **Extensible via signed plugins** | Third-party skills without compromising security |
| G6 | **Privacy-preserving model interaction** | Sensitive data never sent to cloud without consent |
| G7 | **Audit trail for all actions** | Complete, tamper-evident log of agent activity |
| G8 | **Portable core architecture** | Core logic reusable on Linux/Windows in future |

### 1.3 Non-Goals (Explicitly Out of Scope)

| ID | Non-Goal | Rationale |
|----|----------|-----------|
| NG1 | **Full web browser replacement** | Browser Mode is for agent-assisted tasks, not general browsing |
| NG2 | **Captcha solving/bypass** | Security and ToS concerns; always hand off to user |
| NG3 | **Fully autonomous operation** | Human-in-the-loop for sensitive actions is a feature |
| NG4 | **Multi-user/team features** | v1 is single-user; team features are future |
| NG5 | **Mobile apps (iOS/Android)** | macOS-first; mobile is future with portable core |
| NG6 | **Enterprise SSO/SCIM** | Individual/small team focus for v1 |
| NG7 | **Cryptocurrency/trading automation** | High-risk category excluded from v1 |
| NG8 | **Medical/healthcare automation** | Regulatory complexity excluded from v1 |

### 1.4 Design Principles

1. **Security First:** Every feature is designed with threat model in mind
2. **Explicit Over Implicit:** User must opt-in to capabilities, not opt-out
3. **Fail Secure:** On error or ambiguity, deny access and ask user
4. **Minimal Trust:** Assume model output could be adversarial
5. **Defense in Depth:** Multiple layers protect sensitive operations
6. **Transparency:** User can always see what agent is doing and why
7. **Reversibility:** Actions should be undoable where possible

---

## 2. User Experiences

### 2.1 Application Layout

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  NextStepAI                                              [−] [□] [×]        │
├────────────┬────────────────────────────────────┬───────────────────────────┤
│            │                                    │                           │
│  LEFT NAV  │         CENTER PANEL               │      RIGHT PANEL          │
│  (Modes)   │      (Content Area)                │   (Context + Actions)     │
│            │                                    │                           │
│  ┌──────┐  │  ┌────────────────────────────┐   │  ┌─────────────────────┐  │
│  │ Chat │  │  │                            │   │  │ Credentials Manager │  │
│  │ Mode │◀─┼──│   Chat Interface           │   │  │                     │  │
│  └──────┘  │  │   (when Chat Mode)         │   │  │ ┌─────────────────┐ │  │
│            │  │                            │   │  │ │ github.com      │ │  │
│  ┌──────┐  │  │   OR                       │   │  │ │ ├── Personal    │ │  │
│  │Browse│  │  │                            │   │  │ │ └── Work        │ │  │
│  │ Mode │  │  │   Embedded WebKit          │   │  │ ├─────────────────┤ │  │
│  └──────┘  │  │   (when Browser Mode)      │   │  │ │ google.com      │ │  │
│            │  │                            │   │  │ │ └── Main        │ │  │
│  ┌──────┐  │  └────────────────────────────┘   │  │ └─────────────────┘ │  │
│  │Histor│  │                                    │  │                     │  │
│  │  y   │  │  ┌────────────────────────────┐   │  │ [+ Add Credential]  │  │
│  └──────┘  │  │ ▶ Type or speak command... │   │  └─────────────────────┘  │
│            │  └────────────────────────────┘   │                           │
│  ┌──────┐  │                                    │  ┌─────────────────────┐  │
│  │Skills│  │                                    │  │ Approvals Queue     │  │
│  └──────┘  │                                    │  │                     │  │
│            │                                    │  │ ⚠ Pending (2)       │  │
│  ┌──────┐  │                                    │  │ ┌─────────────────┐ │  │
│  │Settin│  │                                    │  │ │ Fill login form │ │  │
│  │  gs  │  │                                    │  │ │ github.com      │ │  │
│  └──────┘  │                                    │  │ │ [Approve][Deny] │ │  │
│            │                                    │  │ ├─────────────────┤ │  │
│            │                                    │  │ │ Run shell cmd   │ │  │
│  ┌──────┐  │                                    │  │ │ ls ~/Documents  │ │  │
│  │ 🔒   │  │                                    │  │ │ [Approve][Deny] │ │  │
│  │Status│  │                                    │  │ └─────────────────┘ │  │
│  └──────┘  │                                    │  │                     │  │
│            │                                    │  │ ✓ Approved Today: 5 │  │
│            │                                    │  │ ✗ Denied Today: 1   │  │
│            │                                    │  └─────────────────────┘  │
└────────────┴────────────────────────────────────┴───────────────────────────┘
```

### 2.2 Mode Descriptions

#### Chat Mode
- **Purpose:** Natural language interaction with the agent
- **Layout:** Conversation thread in center, credentials + approvals in right panel
- **Input:** Text field with voice input support (macOS dictation)
- **Output:** Streaming text responses with inline tool execution indicators

#### Browser Mode
- **Purpose:** Agent-assisted web browsing and automation
- **Layout:** Embedded WebKit browser in center, same right panel
- **Navigation:** URL bar, back/forward, refresh, allowlist indicator
- **Automation:** Visual indicators when agent interacts with page

#### History Mode
- **Purpose:** Review past sessions and workflows
- **Layout:** Session list in center, session details on selection
- **Features:** Search, filter by date/type, replay workflow

#### Skills Mode
- **Purpose:** Manage installed plugins/skills
- **Layout:** Skill list with status, configuration options
- **Features:** Install, enable/disable, configure, view permissions

#### Settings Mode
- **Purpose:** Application configuration
- **Sections:** Security, Providers, Privacy, Appearance, Shortcuts

### 2.3 User Journeys

#### Journey 1: First-Time Setup

```
1. User launches NextStepAI for first time
2. Welcome screen explains security model
3. User configures model provider:
   - Option A: Cloud (Anthropic/OpenAI) - requires API key
   - Option B: Local (Ollama) - requires local setup
   - Option C: Skip for now (limited functionality)
4. User sets security preferences:
   - Default security level (Standard recommended)
   - Biometric unlock preference
5. User creates first credential (optional)
6. Brief tutorial of main features
7. User arrives at Chat Mode, ready to use
```

#### Journey 2: Login Workflow (with Credential Fill)

```
1. User types: "Log me into GitHub"
2. Agent parses intent: Login(domain: "github.com")
3. Agent checks:
   - Is github.com in domain allowlist? Yes ✓
   - Does user have stored credentials for github.com? Yes ✓
4. Agent switches to Browser Mode
5. Agent navigates to github.com/login
6. Agent detects login form
7. APPROVAL REQUEST appears in right panel:
   "Fill login form on github.com using credential 'GitHub Personal'?"
   [Approve] [Deny] [Approve for session]
8. User clicks [Approve]
9. Agent fills username field
10. Agent fills password field (redacted in UI)
11. Agent clicks submit button
12. GitHub shows 2FA prompt
13. Agent detects 2FA, notifies user: "Please complete 2FA"
14. User completes 2FA
15. Agent detects successful login
16. Agent reports: "Successfully logged into GitHub"
17. User can continue browsing or return to Chat Mode
```

#### Journey 3: Captcha Handoff

```
1. Agent is filling a form on allowed site
2. Agent detects CAPTCHA challenge
3. Agent pauses automation
4. HANDOFF notification appears:
   "CAPTCHA detected. Please solve it to continue."
   [I've solved it] [Cancel workflow]
5. User solves CAPTCHA manually
6. User clicks [I've solved it]
7. Agent resumes automation
```

#### Journey 4: Risky Action Approval

```
1. User types: "Delete all files in my Downloads folder"
2. Agent parses intent: DeleteFiles(path: "~/Downloads/*")
3. Policy engine flags as HIGH RISK
4. Agent responds: "I can help with that, but this is a destructive action."
5. APPROVAL REQUEST with elevated warning:
   ⚠️ DESTRUCTIVE ACTION
   "Delete all files in ~/Downloads (47 files, 2.3 GB)"
   This action cannot be undone.
   [Confirm Deletion] [Cancel]
6. User must type "DELETE" to confirm (not just click)
7. Agent executes deletion
8. Agent reports result with deleted file count
9. Audit log records: action, approval, files affected
```

### 2.4 Voice Command Flow

```
User presses hotkey (⌘+Shift+Space by default)
     │
     ▼
┌─────────────────┐
│  Listening...   │
│  [waveform viz] │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ "Log into my bank and   │
│  check my balance"      │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Agent processes:        │
│ 1. Intent: Login + Query│
│ 2. Domain: ??? (unknown)│
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Agent asks:             │
│ "Which bank would you   │
│  like to log into?"     │
│ [Bank of America]       │
│ [Chase]                 │
│ [Other...]              │
└─────────────────────────┘
```

---

## 3. Functional Requirements

### 3.1 Policy & Capabilities

#### FR-POL-001: Capability-Based Permission Model
- Every tool requires explicit capabilities to execute
- Capabilities are fine-grained (e.g., `readFile(~/Documents)` not just `readFile`)
- Capabilities must be granted before first use
- Capabilities can have scopes: `once`, `session`, `untilRevoked`

#### FR-POL-002: Security Level Enforcement
- System operates in one of four security levels: Lockdown, Read-Only, Standard, Elevated
- Each level defines which tool categories are available
- Transitioning to Elevated requires biometric authentication
- Lockdown can be triggered manually or automatically (on security incident)

#### FR-POL-003: Domain Allowlists
- Browser automation only works on allowlisted domains
- Default allowlist includes major trusted sites
- User can add/remove domains with approval
- Adding a domain requires understanding risks (confirmation dialog)

#### FR-POL-004: Rate Limiting
- Tool executions are rate-limited per category
- Credential access limited to 10 per minute
- Shell commands limited to 5 per minute
- Limits adjustable in Elevated mode

### 3.2 Approvals

#### FR-APR-001: Approval Queue
- Pending approvals displayed in right panel
- Each approval shows: action description, tool, target, risk level
- User can approve, deny, or approve with conditions

#### FR-APR-002: Approval Scopes
- `approve_once`: Valid for this specific invocation only
- `approve_session`: Valid until app restart
- `approve_domain`: Valid for this domain until revoked
- `approve_always`: Valid until explicitly revoked (requires elevated confirmation)

#### FR-APR-003: Approval History
- All approvals and denials are logged
- User can review approval history
- User can revoke previously granted approvals

#### FR-APR-004: Batch Approvals
- Workflows can request batch approval for multiple steps
- User sees full workflow before approving
- User can approve workflow with "stop at any error" option

### 3.3 Credential Storage & Retrieval

#### FR-CRD-001: Keychain Integration
- Credentials stored in macOS Keychain with access control
- Keychain item requires user presence (password or biometric) to access
- No automatic sync to iCloud Keychain (local only by default)

#### FR-CRD-002: Credential Types
- Username/Password pairs
- API keys/tokens
- SSH keys (reference only, not copied)
- Client certificates
- Custom key-value pairs

#### FR-CRD-003: Secret Handles
- Model never receives raw credential values
- Model receives opaque handles (UUIDs)
- Handles resolved at execution time by trusted code
- Handle resolution requires fresh approval (or session approval)

#### FR-CRD-004: Credential Metadata
- Domain association (which sites can use this credential)
- Creation/modification timestamps
- Last used timestamp
- Usage count
- Notes (user-provided, not sent to model)

#### FR-CRD-005: Credential Management UI
- Add new credentials with secure input
- Edit existing credentials (re-authenticate required)
- Delete credentials (confirmation required)
- Import from browser (Safari/Chrome) with approval
- Export encrypted backup

### 3.4 Browser Automation

#### FR-BRW-001: Embedded WebKit
- WKWebView embedded in Browser Mode
- Separate cookie/storage from system Safari
- Content Security Policy enforced
- Mixed content blocked

#### FR-BRW-002: Navigation Control
- Navigate to URLs (allowlisted domains only)
- Back, forward, refresh, stop
- URL bar with security indicators
- Page load status and progress

#### FR-BRW-003: Page Interaction
- Read page content (text, structure)
- Click elements
- Fill form fields
- Submit forms
- Scroll to elements
- Wait for elements/conditions

#### FR-BRW-004: Form Detection
- Detect login forms (username/password fields)
- Detect registration forms
- Detect payment forms (flag as sensitive, extra approval)
- Detect search forms

#### FR-BRW-005: Credential Fill
- Match page domain to credential domain
- Fill only with user approval
- Mask password in UI during fill
- Log credential usage (metadata only)

#### FR-BRW-006: CAPTCHA Handling
- Detect common CAPTCHA patterns (reCAPTCHA, hCaptcha, etc.)
- Pause automation on CAPTCHA detection
- Notify user with visual indicator
- Wait for user confirmation to continue
- Never attempt to solve or bypass CAPTCHA

#### FR-BRW-007: Session Management
- Persist sessions for allowlisted domains
- Clear sessions on demand
- Session isolation between domains
- Export session (for debugging, encrypted)

### 3.5 Tool System

#### FR-TOL-001: Tool Protocol
- Tools are typed Swift protocols
- Tools declare required capabilities
- Tools have structured input/output types
- Tools report status (pending, running, success, failure)

#### FR-TOL-002: Built-in Tools

| Tool | Category | Risk Level | Capabilities Required |
|------|----------|------------|----------------------|
| `NotesTool` | Productivity | Low | `.notes` |
| `RemindersTool` | Productivity | Low | `.reminders` |
| `CalendarTool` | Productivity | Low | `.calendar` |
| `CalculatorTool` | Utility | None | (none) |
| `ClipboardTool` | System | Medium | `.clipboard` |
| `LocalSearchTool` | File | Medium | `.readFile(scope)` |
| `FileReadTool` | File | Medium | `.readFile(scope)` |
| `FileWriteTool` | File | High | `.writeFile(scope)` |
| `HTTPTool` | Network | Medium-High | `.network(domain, methods)` |
| `BrowserTool` | Browser | High | `.browser(domain, actions)` |
| `ShellTool` | System | Critical | `.shell(sandboxProfile)` |

#### FR-TOL-003: Tool Discovery
- Tools are registered at startup
- Plugins can register additional tools
- Tools can be enabled/disabled per security level
- Tool metadata available for model context

#### FR-TOL-004: Tool Execution
- All tool calls go through policy engine
- Execution is async with cancellation support
- Timeouts enforced per tool category
- Results sanitized before returning to model

### 3.6 Skills/Plugins

#### FR-PLG-001: Plugin Bundle Format
```
ExamplePlugin.nextstep/
├── manifest.json          # Plugin metadata and capabilities
├── signature              # Code signature
├── Resources/
│   └── icon.png
└── PluginBinary           # Compiled XPC service
```

#### FR-PLG-002: Plugin Manifest
```json
{
  "id": "com.example.myplugin",
  "name": "Example Plugin",
  "version": "1.0.0",
  "minAppVersion": "1.0.0",
  "author": "Example Corp",
  "description": "An example plugin",
  "capabilities": [
    { "type": "network", "domains": ["api.example.com"] },
    { "type": "readFile", "scope": "userDocuments" }
  ],
  "tools": [
    {
      "name": "exampleTool",
      "description": "Does example things",
      "inputSchema": { ... },
      "outputSchema": { ... }
    }
  ]
}
```

#### FR-PLG-003: Signing Requirements
- Plugins must be signed with Apple Developer ID
- Plugins must be notarized by Apple (if distributed outside App Store)
- Signature verified before any code execution
- Tampered plugins rejected with clear error

#### FR-PLG-004: Sandboxing
- Each plugin runs in isolated XPC service
- Sandbox profile allows only declared capabilities
- Network access limited to declared domains
- File access limited to declared scopes
- No access to Keychain or other plugins

#### FR-PLG-005: Plugin Lifecycle
- Install: Verify signature, show capabilities, get user approval
- Enable: Start XPC service, register tools
- Disable: Stop XPC service, unregister tools
- Update: Verify new signature, re-approve if capabilities changed
- Uninstall: Stop service, remove bundle, clean data

### 3.7 Remote Access

#### FR-REM-001: Default Loopback-Only
- HTTP server binds to `127.0.0.1` only
- External connections rejected
- This cannot be changed without Elevated mode

#### FR-REM-002: SSH Tunnel Mode
- Documentation for SSH tunnel setup
- Helper scripts for common SSH configurations
- Connection status indicator in UI

#### FR-REM-003: Tailnet Mode
- Integration with Tailscale daemon (if installed)
- Automatic service advertisement on Tailnet
- Authentication via Tailscale identity

#### FR-REM-004: mTLS Mode (Enterprise)
- Direct HTTPS with client certificate authentication
- Certificate management in Settings
- Certificate revocation checking

#### FR-REM-005: Remote Session Management
- View active remote sessions
- Session shows: source, connection time, activity
- Terminate individual sessions
- Terminate all sessions (panic button)

### 3.8 Logs & Audit

#### FR-AUD-001: Audit Event Types

| Event Type | Logged Data |
|------------|-------------|
| `session.start` | Session ID, timestamp |
| `session.end` | Session ID, duration, summary |
| `intent.parsed` | Intent type, parameters (redacted) |
| `tool.requested` | Tool name, input (redacted) |
| `tool.approved` | Tool name, approval scope, approver |
| `tool.denied` | Tool name, denial reason |
| `tool.executed` | Tool name, duration, result status |
| `credential.accessed` | Credential ID, domain, purpose |
| `capability.granted` | Capability, scope, expiration |
| `capability.revoked` | Capability, reason |
| `security.levelChange` | Old level, new level, trigger |
| `security.lockdown` | Trigger reason, state snapshot |
| `remote.connected` | Source, auth method |
| `remote.disconnected` | Source, duration |
| `plugin.loaded` | Plugin ID, version |
| `plugin.error` | Plugin ID, error type |

#### FR-AUD-002: Hash Chain
- Each log entry includes hash of previous entry
- Chain enables tamper detection
- Verification tool included in app

#### FR-AUD-003: Log Storage
- Encrypted at rest (AES-256-GCM)
- Retained for 90 days by default (configurable)
- Rotation preserves chain integrity
- Export in signed JSON format

#### FR-AUD-004: Log Viewer
- Filter by event type, time range, severity
- Search within log entries
- Highlight security-relevant events
- Verify chain integrity on demand

### 3.9 Model Provider Selection

#### FR-PRV-001: Supported Providers
- Anthropic (Claude 3.5 Sonnet, Claude 3 Opus)
- OpenAI (GPT-4, GPT-4 Turbo)
- Local (Ollama with compatible models)
- Custom (user-provided endpoint with compatible API)

#### FR-PRV-002: Provider Configuration
- API key storage (in Keychain)
- Endpoint configuration
- Model selection
- Temperature and other parameters
- Cost tracking (token usage)

#### FR-PRV-003: Fallback Chain
- User configures primary and fallback providers
- Automatic fallback on provider errors
- Fallback notification in UI
- Different fallback for different task types (optional)

#### FR-PRV-004: Local-Only Mode
- All inference runs locally via Ollama
- No network requests to cloud providers
- Reduced capability (may not support all workflows)
- Clear indicator in UI

---

## 4. Non-Functional Requirements

### 4.1 Security Guarantees

| ID | Guarantee | Verification Method |
|----|-----------|---------------------|
| SEC-001 | Credentials never sent to model provider in raw form | Code audit, network inspection |
| SEC-002 | All tool executions logged with tamper-evident audit | Chain verification, forensic analysis |
| SEC-003 | Plugins cannot escape sandbox to access host resources | Sandbox testing, escape attempt tests |
| SEC-004 | Remote access impossible without explicit configuration | Default bind verification, port scanning |
| SEC-005 | Secrets automatically redacted from all logs | Pattern matching, log inspection |
| SEC-006 | Failed authentication attempts rate-limited | Rate limit testing |
| SEC-007 | Biometric required for Elevated mode | UI testing, bypass attempts |
| SEC-008 | No capability escalation without user action | Policy engine tests |

### 4.2 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| App launch time | < 2 seconds | Cold start to interactive |
| Chat response start | < 500ms | After model response begins |
| Tool execution overhead | < 50ms | Policy check + dispatch |
| Browser page load | Native WebKit | Same as Safari |
| Credential retrieval | < 200ms | With biometric cached |
| Audit log write | < 10ms | Async, non-blocking |
| Plugin load time | < 1 second | Signature verify + XPC start |

### 4.3 Offline/Local-Only Mode

#### Requirements:
- App must function with no internet connection
- Local model (Ollama) provides inference
- Credentials accessible (with biometric)
- Audit logs continue working
- Browser Mode shows offline indicator

#### Limitations in Offline Mode:
- Cloud providers unavailable
- Plugin updates not checked
- No remote access (expected)
- Some workflows may fail (graceful degradation)

### 4.4 Privacy Rules

#### What Can Be Sent to Model Providers

| Data Type | Cloud Provider | Local Provider | Condition |
|-----------|----------------|----------------|-----------|
| User messages | Yes | Yes | Always |
| Tool outputs (redacted) | Yes | Yes | After redaction |
| Page content (text) | Yes | Yes | Allowlisted domains |
| File contents | With approval | Yes | Per-file approval |
| Credential values | Never | Never | N/A |
| System information | Limited | Full | Minimal needed |
| Audit logs | Never | Never | N/A |

#### Data Minimization Rules:
- Send only information needed for current task
- Strip personally identifiable information where possible
- Classify content sensitivity before sending
- User can review what will be sent (transparency mode)

### 4.5 Observability

#### Metrics Collected (Local Only):
- Tool usage counts
- Approval/denial rates
- Error counts by category
- Response latencies
- Token usage per provider
- Session durations

#### Health Indicators:
- Model provider connectivity
- Keychain accessibility
- Disk space for logs
- Plugin health
- Security level status

#### Alerting:
- Anomalous behavior detected
- High rate of denials
- Provider errors
- Audit chain integrity issue
- Approaching storage limits

### 4.6 Reliability

| Requirement | Target |
|-------------|--------|
| Crash-free sessions | 99.9% |
| Data durability (credentials) | 100% (Keychain-backed) |
| Data durability (logs) | 99.99% |
| Recovery from crash | Resume session state |
| Recovery from lockdown | Clear path with auth |

#### Failure Modes and Recovery:

| Failure | Detection | Recovery |
|---------|-----------|----------|
| App crash | Crash reporter | Restore session, log incident |
| Provider timeout | Timeout > 30s | Fallback or local, notify user |
| Plugin crash | XPC termination | Isolate plugin, continue app |
| Keychain unavailable | Access error | Prompt unlock, degrade gracefully |
| Disk full | Write error | Warn user, pause logging (not stop) |
| Network loss | Reachability monitor | Switch to local mode |

---

## 5. Data Model

### 5.1 Core Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Entity Relationship Diagram                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐                         │
│  │ Session  │ 1───* │ Workflow │ 1───* │ToolCall  │                         │
│  └────┬─────┘       └────┬─────┘       └────┬─────┘                         │
│       │                  │                  │                                │
│       │                  │                  │                                │
│       │ 1                │ 1                │ *                              │
│       │                  │                  │                                │
│       ▼ *                ▼ *                ▼ 1                              │
│  ┌──────────┐       ┌──────────┐       ┌──────────────┐                     │
│  │  Intent  │       │ Approval │       │CapabilityGrant│                    │
│  └──────────┘       │ Request  │       └──────────────┘                     │
│                     └────┬─────┘                                             │
│                          │                                                   │
│                          │ *                                                 │
│                          ▼ 1                                                 │
│                     ┌──────────────┐                                         │
│                     │  Credential  │                                         │
│                     │   Handle     │                                         │
│                     └──────────────┘                                         │
│                                                                              │
│  ┌──────────┐                                                                │
│  │AuditEvent│  (Append-only, references other entities by ID)               │
│  └──────────┘                                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Entity Definitions

#### Session
```swift
struct Session: Identifiable, Codable {
    let id: UUID
    let createdAt: Date
    var endedAt: Date?
    var securityLevel: SecurityLevel
    var state: SessionState // .active, .paused, .ended

    // Relationships
    var intents: [Intent.ID]
    var workflows: [Workflow.ID]
    var activeCapabilities: [CapabilityGrant.ID]
}
```

#### Workflow
```swift
struct Workflow: Identifiable, Codable {
    let id: UUID
    let sessionID: Session.ID
    let createdAt: Date
    var completedAt: Date?

    let intent: Intent
    var steps: [WorkflowStep]
    var state: WorkflowState // .pending, .running, .paused, .completed, .failed
    var result: WorkflowResult?
}

struct WorkflowStep: Identifiable, Codable {
    let id: UUID
    let toolCall: ToolCall
    var state: StepState
    var approvalRequest: ApprovalRequest.ID?
}
```

#### Intent
```swift
struct Intent: Identifiable, Codable {
    let id: UUID
    let sessionID: Session.ID
    let rawInput: String // User's original input
    let parsedAt: Date

    let type: IntentType
    let parameters: [String: IntentParameter]
    let confidence: Double // 0.0-1.0

    var resolved: Bool
    var clarificationNeeded: [ClarificationRequest]?
}

enum IntentType: String, Codable {
    case login
    case search
    case createNote
    case setReminder
    case readFile
    case writeFile
    case browseWeb
    case runCommand
    case custom(String)
}
```

#### ToolCall
```swift
struct ToolCall: Identifiable, Codable {
    let id: UUID
    let workflowID: Workflow.ID
    let stepIndex: Int

    let toolName: String
    let input: ToolInput // Type-erased, serializable
    var output: ToolOutput?

    var state: ToolCallState // .pending, .awaitingApproval, .running, .completed, .failed
    var startedAt: Date?
    var completedAt: Date?
    var error: ToolError?

    let requiredCapabilities: [Capability]
    var usedCapabilityGrants: [CapabilityGrant.ID]
}
```

#### CapabilityGrant
```swift
struct CapabilityGrant: Identifiable, Codable {
    let id: UUID
    let capability: Capability
    let scope: GrantScope

    let grantedAt: Date
    let expiresAt: Date?
    let grantedBy: GrantSource // .user, .policy, .plugin

    var revokedAt: Date?
    var revokedReason: String?

    let signature: Data // Signed with device key

    func isValid(at date: Date = .now) -> Bool {
        revokedAt == nil && (expiresAt == nil || expiresAt! > date)
    }
}

enum Capability: Hashable, Codable {
    case readFile(scope: FileScope)
    case writeFile(scope: FileScope)
    case network(domain: String, methods: Set<HTTPMethod>)
    case credential(id: Credential.ID, usage: CredentialUsage)
    case browser(domain: String, actions: Set<BrowserAction>)
    case shell(sandboxProfile: String)
    case clipboard(access: ClipboardAccess)
    case notes
    case reminders
    case calendar
    case plugin(id: String)
}

enum GrantScope: Codable {
    case once
    case session
    case domain(String)
    case untilRevoked
    case duration(TimeInterval)
}
```

#### ApprovalRequest
```swift
struct ApprovalRequest: Identifiable, Codable {
    let id: UUID
    let toolCallID: ToolCall.ID
    let createdAt: Date

    let title: String
    let description: String
    let riskLevel: RiskLevel // .low, .medium, .high, .critical
    let requiredCapabilities: [Capability]
    let context: ApprovalContext // What triggered this, relevant data

    var decision: ApprovalDecision?
    var decidedAt: Date?

    var expiresAt: Date // Auto-deny after timeout
}

struct ApprovalDecision: Codable {
    let approved: Bool
    let scope: GrantScope?
    let reason: String?
    let decidedBy: DecisionSource // .user, .policy, .timeout
}
```

#### CredentialHandle
```swift
struct CredentialHandle: Identifiable, Codable {
    let id: UUID // This is what the model sees
    let credentialID: Credential.ID // Internal reference
    let createdAt: Date
    let expiresAt: Date

    let purpose: String // Why this handle was created
    let allowedDomains: [String] // Where this can be used

    // Handle cannot be used to retrieve credential without:
    // 1. Valid handle (not expired)
    // 2. Domain match
    // 3. Fresh approval (or session approval)
    // 4. Biometric (depending on security level)
}
```

#### AuditEvent
```swift
struct AuditEvent: Identifiable, Codable {
    let id: UUID
    let timestamp: Date
    let eventType: AuditEventType
    let severity: AuditSeverity // .debug, .info, .warning, .error, .critical

    let sessionID: Session.ID?
    let workflowID: Workflow.ID?
    let toolCallID: ToolCall.ID?

    let data: [String: AuditValue] // Structured, redacted data

    let previousHash: Data // Hash of previous event
    let hash: Data // Hash of this event (including previousHash)
}
```

### 5.3 Storage Approach

#### Storage Strategy by Entity

| Entity | Storage | Encryption | Backup |
|--------|---------|------------|--------|
| Session | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| Workflow | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| Intent | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| ToolCall | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| CapabilityGrant | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| ApprovalRequest | SQLite (GRDB) | At-rest (SQLCipher) | Optional |
| Credential | macOS Keychain | Hardware (Secure Enclave) | Via Keychain |
| CredentialHandle | SQLite (GRDB) | At-rest (SQLCipher) | No (ephemeral) |
| AuditEvent | Append-only file | At-rest (AES-256-GCM) | Recommended |

#### Encryption Keys

```
┌─────────────────────────────────────────────────────────────────┐
│                        Key Hierarchy                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │        Secure Enclave Master Key         │                   │
│  │   (Non-exportable, biometric-gated)      │                   │
│  └────────────────────┬─────────────────────┘                   │
│                       │                                          │
│          ┌────────────┼────────────┐                            │
│          │            │            │                            │
│          ▼            ▼            ▼                            │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │  Database    │ │  Audit Log   │ │  Export      │            │
│  │  Encryption  │ │  Encryption  │ │  Encryption  │            │
│  │  Key (DEK)   │ │  Key (LEK)   │ │  Key (XEK)   │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│                                                                  │
│  Credentials stored directly in Keychain with Secure Enclave    │
│  access control - no additional encryption layer needed          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Database Schema (Conceptual)

```sql
-- Sessions table
CREATE TABLE sessions (
    id TEXT PRIMARY KEY,
    created_at INTEGER NOT NULL,
    ended_at INTEGER,
    security_level TEXT NOT NULL,
    state TEXT NOT NULL
);

-- Workflows table
CREATE TABLE workflows (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL REFERENCES sessions(id),
    created_at INTEGER NOT NULL,
    completed_at INTEGER,
    intent_json TEXT NOT NULL,
    state TEXT NOT NULL,
    result_json TEXT
);

-- Tool calls table
CREATE TABLE tool_calls (
    id TEXT PRIMARY KEY,
    workflow_id TEXT NOT NULL REFERENCES workflows(id),
    step_index INTEGER NOT NULL,
    tool_name TEXT NOT NULL,
    input_json TEXT NOT NULL,
    output_json TEXT,
    state TEXT NOT NULL,
    started_at INTEGER,
    completed_at INTEGER,
    error_json TEXT
);

-- Capability grants table
CREATE TABLE capability_grants (
    id TEXT PRIMARY KEY,
    capability_json TEXT NOT NULL,
    scope_json TEXT NOT NULL,
    granted_at INTEGER NOT NULL,
    expires_at INTEGER,
    granted_by TEXT NOT NULL,
    revoked_at INTEGER,
    revoked_reason TEXT,
    signature BLOB NOT NULL
);

-- Approval requests table
CREATE TABLE approval_requests (
    id TEXT PRIMARY KEY,
    tool_call_id TEXT NOT NULL REFERENCES tool_calls(id),
    created_at INTEGER NOT NULL,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    risk_level TEXT NOT NULL,
    decision_json TEXT,
    decided_at INTEGER,
    expires_at INTEGER NOT NULL
);

-- Credential handles table
CREATE TABLE credential_handles (
    id TEXT PRIMARY KEY,
    credential_id TEXT NOT NULL,
    created_at INTEGER NOT NULL,
    expires_at INTEGER NOT NULL,
    purpose TEXT NOT NULL,
    allowed_domains_json TEXT NOT NULL
);

-- Indices for common queries
CREATE INDEX idx_workflows_session ON workflows(session_id);
CREATE INDEX idx_tool_calls_workflow ON tool_calls(workflow_id);
CREATE INDEX idx_capability_grants_expires ON capability_grants(expires_at);
CREATE INDEX idx_approval_requests_pending ON approval_requests(decided_at) WHERE decided_at IS NULL;
```

---

## 6. APIs / Interfaces

### 6.1 Tool Invocation Interface

```swift
/// Protocol that all tools must implement
protocol Tool {
    /// Unique identifier for this tool
    static var id: String { get }

    /// Human-readable name
    static var name: String { get }

    /// Description for model context
    static var description: String { get }

    /// JSON Schema for input parameters
    static var inputSchema: JSONSchema { get }

    /// JSON Schema for output
    static var outputSchema: JSONSchema { get }

    /// Capabilities required to execute this tool
    static var requiredCapabilities: [Capability] { get }

    /// Risk level of this tool
    static var riskLevel: RiskLevel { get }

    /// Categories this tool belongs to
    static var categories: Set<ToolCategory> { get }

    /// Execute the tool with given input
    func execute(input: ToolInput, context: ToolContext) async throws -> ToolOutput

    /// Validate input before execution
    func validate(input: ToolInput) -> ValidationResult

    /// Cancel ongoing execution
    func cancel() async
}

/// Context provided to tool during execution
struct ToolContext {
    let sessionID: Session.ID
    let workflowID: Workflow.ID
    let toolCallID: ToolCall.ID
    let securityLevel: SecurityLevel
    let grantedCapabilities: [CapabilityGrant]
    let credentialResolver: CredentialResolver
    let auditLogger: AuditLogger
}

/// Type-erased tool input
struct ToolInput: Codable {
    let parameters: [String: JSONValue]

    func decode<T: Decodable>(_ type: T.Type) throws -> T
}

/// Type-erased tool output
struct ToolOutput: Codable {
    let success: Bool
    let result: JSONValue?
    let error: ToolError?
    let metadata: [String: JSONValue]
}
```

### 6.2 Policy Engine API

```swift
/// Main policy engine actor
actor PolicyEngine {
    /// Evaluate whether a tool can be executed
    func evaluate(
        tool: any Tool,
        input: ToolInput,
        context: PolicyContext
    ) async -> PolicyDecision

    /// Check if a specific capability is granted
    func hasCapability(
        _ capability: Capability,
        in context: PolicyContext
    ) async -> Bool

    /// Get all active capability grants for a session
    func activeGrants(for sessionID: Session.ID) async -> [CapabilityGrant]

    /// Request a capability grant (creates approval request if needed)
    func requestCapability(
        _ capability: Capability,
        scope: GrantScope,
        reason: String,
        context: PolicyContext
    ) async -> CapabilityRequestResult

    /// Revoke a capability grant
    func revokeGrant(_ grantID: CapabilityGrant.ID, reason: String) async

    /// Get current security level
    func currentSecurityLevel() async -> SecurityLevel

    /// Request security level change (may require biometric)
    func requestSecurityLevelChange(
        to level: SecurityLevel,
        reason: String
    ) async throws -> SecurityLevelChangeResult

    /// Trigger lockdown mode
    func triggerLockdown(reason: String) async
}

/// Result of policy evaluation
enum PolicyDecision {
    case allow(grants: [CapabilityGrant])
    case deny(reason: String, missingCapabilities: [Capability])
    case requireApproval(request: ApprovalRequest)
    case blocked(reason: String) // Security level doesn't allow
}

/// Context for policy evaluation
struct PolicyContext {
    let sessionID: Session.ID
    let workflowID: Workflow.ID?
    let securityLevel: SecurityLevel
    let requestSource: RequestSource // .user, .workflow, .plugin
}
```

### 6.3 Credentials Manager API

```swift
/// Credentials manager interface
protocol CredentialsManager {
    /// List all credentials (metadata only)
    func listCredentials() async throws -> [CredentialMetadata]

    /// Get credential metadata by ID
    func getMetadata(id: Credential.ID) async throws -> CredentialMetadata?

    /// Create a new credential
    func createCredential(
        type: CredentialType,
        metadata: CredentialMetadata,
        value: CredentialValue
    ) async throws -> Credential.ID

    /// Update credential value (requires re-auth)
    func updateCredential(
        id: Credential.ID,
        value: CredentialValue
    ) async throws

    /// Delete a credential
    func deleteCredential(id: Credential.ID) async throws

    /// Create a handle for credential access
    func createHandle(
        credentialID: Credential.ID,
        purpose: String,
        allowedDomains: [String],
        expiresIn: TimeInterval
    ) async throws -> CredentialHandle

    /// Resolve a handle to actual credential value
    /// Requires valid handle + domain match + approval
    func resolveHandle(
        _ handle: CredentialHandle,
        forDomain: String,
        withApproval: ApprovalDecision
    ) async throws -> CredentialValue

    /// Find credentials matching a domain
    func findCredentials(forDomain: String) async throws -> [CredentialMetadata]

    /// Export credentials (encrypted backup)
    func exportCredentials(
        ids: [Credential.ID],
        encryptedWith password: String
    ) async throws -> Data

    /// Import credentials from backup
    func importCredentials(
        from data: Data,
        decryptedWith password: String
    ) async throws -> [Credential.ID]
}

/// Credential metadata (safe to expose)
struct CredentialMetadata: Identifiable, Codable {
    let id: Credential.ID
    let type: CredentialType
    let name: String
    let domains: [String]
    let createdAt: Date
    let modifiedAt: Date
    let lastUsedAt: Date?
    let usageCount: Int
    let notes: String?
}

/// Credential value (secret, never exposed to model)
enum CredentialValue {
    case usernamePassword(username: String, password: String)
    case apiKey(key: String)
    case token(token: String, refreshToken: String?)
    case certificate(data: Data, password: String?)
    case custom([String: String])
}
```

### 6.4 Browser Control API

```swift
/// Browser controller interface
protocol BrowserController {
    /// Current URL
    var currentURL: URL? { get async }

    /// Current page title
    var currentTitle: String? { get async }

    /// Loading state
    var isLoading: Bool { get async }

    /// Navigate to URL (must be allowlisted)
    func navigate(to url: URL) async throws

    /// Go back in history
    func goBack() async throws

    /// Go forward in history
    func goForward() async throws

    /// Refresh current page
    func refresh() async throws

    /// Stop loading
    func stop() async

    /// Get page content as text
    func getPageText() async throws -> String

    /// Get page structure (simplified DOM)
    func getPageStructure() async throws -> PageStructure

    /// Find element by selector
    func findElement(_ selector: String) async throws -> WebElement?

    /// Find all elements matching selector
    func findElements(_ selector: String) async throws -> [WebElement]

    /// Click an element
    func click(_ element: WebElement) async throws

    /// Fill a text field
    func fill(_ element: WebElement, text: String) async throws

    /// Submit a form
    func submitForm(_ form: WebElement) async throws

    /// Scroll to element
    func scrollTo(_ element: WebElement) async throws

    /// Wait for element to appear
    func waitForElement(
        _ selector: String,
        timeout: TimeInterval
    ) async throws -> WebElement

    /// Wait for navigation to complete
    func waitForNavigation(timeout: TimeInterval) async throws

    /// Detect forms on current page
    func detectForms() async throws -> [DetectedForm]

    /// Detect CAPTCHAs on current page
    func detectCaptcha() async throws -> CaptchaDetection?

    /// Fill credential into form (requires approval)
    func fillCredential(
        form: DetectedForm,
        credential: CredentialHandle,
        approval: ApprovalDecision
    ) async throws
}

/// Detected form information
struct DetectedForm {
    let id: String
    let type: FormType // .login, .registration, .search, .payment, .other
    let fields: [FormField]
    let submitButton: WebElement?
}

/// CAPTCHA detection result
struct CaptchaDetection {
    let type: CaptchaType // .recaptcha, .hcaptcha, .cloudflare, .custom
    let element: WebElement
    let requiresUserInteraction: Bool
}
```

### 6.5 Plugin Interface

```swift
/// Plugin protocol that all plugins must implement
protocol Plugin {
    /// Plugin identifier (reverse domain notation)
    static var id: String { get }

    /// Plugin version
    static var version: String { get }

    /// Minimum app version required
    static var minAppVersion: String { get }

    /// Plugin manifest
    static var manifest: PluginManifest { get }

    /// Initialize plugin with configuration
    init(configuration: PluginConfiguration) throws

    /// Called when plugin is enabled
    func activate() async throws

    /// Called when plugin is disabled
    func deactivate() async

    /// Get tools provided by this plugin
    func getTools() -> [any Tool]

    /// Handle a tool invocation
    func handleToolInvocation(
        toolID: String,
        input: ToolInput,
        context: PluginContext
    ) async throws -> ToolOutput
}

/// Plugin manifest
struct PluginManifest: Codable {
    let id: String
    let name: String
    let version: String
    let minAppVersion: String
    let author: String
    let description: String
    let homepage: URL?
    let requiredCapabilities: [PluginCapability]
    let tools: [PluginToolManifest]
    let entryPoint: String
}

/// Plugin capability request
struct PluginCapability: Codable {
    let type: PluginCapabilityType
    let scope: String?
    let reason: String
}

enum PluginCapabilityType: String, Codable {
    case network // Requires domains specification
    case readFile // Requires scope specification
    case writeFile // Requires scope specification
    case clipboard
    case notifications
}

/// Context provided to plugin during tool execution
struct PluginContext {
    let sessionID: Session.ID
    let securityLevel: SecurityLevel

    /// Make HTTP request (only to declared domains)
    func httpRequest(_ request: HTTPRequest) async throws -> HTTPResponse

    /// Read file (only in declared scope)
    func readFile(at path: String) async throws -> Data

    /// Write file (only in declared scope)
    func writeFile(_ data: Data, to path: String) async throws

    /// Log to audit trail
    func log(_ message: String, level: LogLevel)
}
```

### 6.6 API Versioning

All interfaces are versioned. Plugin manifest includes `minAppVersion` to ensure compatibility.

```swift
/// API version information
struct APIVersion {
    static let current = APIVersion(major: 1, minor: 0, patch: 0)

    let major: Int // Breaking changes
    let minor: Int // Backward-compatible additions
    let patch: Int // Bug fixes

    var string: String { "\(major).\(minor).\(patch)" }

    func isCompatible(with other: APIVersion) -> Bool {
        major == other.major && minor >= other.minor
    }
}
```

---

## 7. Security Specification

### 7.1 High-Risk Action Categories

#### Category: Credential Access
| Action | Risk Level | Controls |
|--------|------------|----------|
| View credential metadata | Low | Authentication only |
| Create credential handle | Medium | Purpose + domains required |
| Resolve handle to value | High | Approval + domain match + biometric |
| Export credentials | Critical | Approval + biometric + password |
| Delete credential | High | Approval + confirmation |

**Rules:**
- Credential values never appear in model context
- Handle creation logged with purpose
- Handle resolution logged with destination
- Export limited to 1 per hour
- Failed resolution attempts rate-limited (3 per minute)

#### Category: File System Access
| Action | Risk Level | Controls |
|--------|------------|----------|
| Read user documents | Medium | Capability grant |
| Read other locations | High | Per-file approval |
| Write user documents | High | Approval + backup |
| Write other locations | Critical | Elevated mode + approval |
| Delete files | Critical | Elevated mode + explicit confirmation |

**Rules:**
- No access outside user home directory without Elevated mode
- Sensitive directories (`.ssh`, `.gnupg`, `Library/Keychains`) blocked entirely
- Binary file reads logged but not sent to model
- Write operations create automatic backup (where possible)
- Delete operations require typing confirmation text

#### Category: Network Access
| Action | Risk Level | Controls |
|--------|------------|----------|
| GET to allowlisted API | Low | Capability grant |
| GET to other domains | Medium | Per-domain approval |
| POST/PUT to allowlisted | Medium | Approval |
| POST/PUT to other domains | High | Elevated mode + approval |
| Outbound with credentials | Critical | Credential resolution controls apply |

**Rules:**
- All requests logged with URL, method, response status
- Request bodies over 10KB summarized, not logged in full
- Response bodies containing secrets redacted
- No access to localhost/private IPs except designated endpoints
- Certificate pinning for known critical APIs

#### Category: Browser Automation
| Action | Risk Level | Controls |
|--------|------------|----------|
| Navigate to allowlisted | Low | Automatic |
| Navigate to other sites | Medium | User confirmation |
| Read page content | Low | Within domain allowlist |
| Fill non-credential fields | Medium | Per-form approval |
| Fill credential fields | High | Credential resolution controls |
| Submit forms | Medium-High | Approval with preview |
| Execute JavaScript | Blocked | Never allowed |

**Rules:**
- JavaScript execution by agent never permitted
- Page content extraction limited to text (no scripts)
- Form submissions logged with field summary (values redacted)
- Payment forms require Elevated mode
- Phishing detection before credential fill

#### Category: System Commands
| Action | Risk Level | Controls |
|--------|------------|----------|
| Read-only commands | High | Elevated mode + sandboxed |
| Modification commands | Critical | Elevated mode + approval + audit |
| Privileged commands | Blocked | Never allowed |

**Rules:**
- All shell commands run in sandboxed profile
- No network access from shell
- No access to credential storage from shell
- Command output size limited (100KB)
- Dangerous commands blocked: `rm -rf /`, `sudo`, `su`, etc.
- Command and output logged

### 7.2 Secrets Redaction

#### Redaction Patterns

```swift
enum RedactionPattern: CaseIterable {
    // API Keys
    case openAIKey      // sk-[a-zA-Z0-9]{48}
    case anthropicKey   // sk-ant-[a-zA-Z0-9-]{90,}
    case awsAccessKey   // AKIA[0-9A-Z]{16}
    case awsSecretKey   // [a-zA-Z0-9/+=]{40}
    case stripeKey      // sk_live_[a-zA-Z0-9]{24,}
    case githubToken    // gh[ps]_[a-zA-Z0-9]{36,}

    // Credentials
    case password       // Common password field patterns
    case bearer         // Bearer [a-zA-Z0-9-._~+/]+=*
    case basicAuth      // Basic [a-zA-Z0-9+/=]+

    // Secrets
    case privateKey     // -----BEGIN.*PRIVATE KEY-----
    case jwtToken       // eyJ[a-zA-Z0-9-_]+\.eyJ[a-zA-Z0-9-_]+\.[a-zA-Z0-9-_]+
    case uuid           // Context-aware (only when marked as secret)

    // PII
    case ssn            // \d{3}-\d{2}-\d{4}
    case creditCard     // \d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}

    var regex: Regex<Substring> { ... }
    var replacement: String { "[REDACTED:\(self)]" }
}
```

#### Redaction Points

| Location | Redaction Applied | Verification |
|----------|-------------------|--------------|
| Audit log entries | All patterns | Log review tests |
| Model request context | All patterns + credential handles | Request inspection |
| Tool outputs to model | All patterns | Output sanitization tests |
| UI display | Credential values (masked) | UI tests |
| Error messages | All patterns | Error handling tests |
| Network request logging | All patterns | Network tests |

### 7.3 Data Minimization

#### Before Sending to Model Provider

1. **Content Classification:**
   - Identify sensitive content (credentials, PII, financial data)
   - Flag content for user review if sensitivity > threshold

2. **Stripping Rules:**
   - Remove full file paths (replace with relative or placeholder)
   - Remove timestamps where not relevant
   - Summarize large content blocks (> 10KB)
   - Exclude binary data entirely

3. **User Review Option:**
   - "Transparency Mode" shows what will be sent
   - User can edit before sending
   - User can cancel send

### 7.4 Network Security

#### Default: Loopback-Only Binding

```
┌─────────────────────────────────────────────────────────────────┐
│                    Default Network Configuration                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │ NextStepAI  │                                                │
│  │   Server    │──── Binds to 127.0.0.1:9147 only              │
│  └─────────────┘                                                │
│        ▲                                                         │
│        │ Only local connections allowed                         │
│        │                                                         │
│  ┌─────────────┐                                                │
│  │  Local      │                                                │
│  │  Client     │                                                │
│  └─────────────┘                                                │
│                                                                  │
│  External connections: REJECTED (not even listening)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### SSH Tunnel Mode

```
┌─────────────────────────────────────────────────────────────────┐
│                      SSH Tunnel Configuration                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Remote Machine                      Local Machine               │
│  ┌─────────────┐                    ┌─────────────┐             │
│  │   Client    │                    │ NextStepAI  │             │
│  │   App       │                    │   Server    │             │
│  └──────┬──────┘                    └──────▲──────┘             │
│         │                                   │                    │
│         │ localhost:9147                    │ 127.0.0.1:9147    │
│         ▼                                   │                    │
│  ┌─────────────┐      SSH Tunnel     ┌─────────────┐            │
│  │   SSH       │◄───────────────────►│   SSHD      │            │
│  │   Client    │  -L 9147:127...:9147│   Server    │            │
│  └─────────────┘                     └─────────────┘            │
│                                                                  │
│  Auth: SSH key authentication                                    │
│  Encryption: SSH (AES-256-GCM)                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Tailnet Mode

```
┌─────────────────────────────────────────────────────────────────┐
│                      Tailnet Configuration                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Tailscale Network                     │    │
│  │                                                          │    │
│  │   Remote Device              Local Device                │    │
│  │   ┌─────────────┐           ┌─────────────┐             │    │
│  │   │   Client    │           │ NextStepAI  │             │    │
│  │   │   App       │           │   Server    │             │    │
│  │   └──────┬──────┘           └──────▲──────┘             │    │
│  │          │                          │                    │    │
│  │          │                          │ 127.0.0.1:9147    │    │
│  │          ▼                          │                    │    │
│  │   ┌─────────────┐           ┌─────────────┐             │    │
│  │   │  Tailscale  │◄─────────►│  Tailscale  │             │    │
│  │   │  Daemon     │ WireGuard │  Daemon     │             │    │
│  │   └─────────────┘           └─────────────┘             │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Auth: Tailscale identity (SSO-backed)                           │
│  Encryption: WireGuard                                           │
│  Service discovery: MagicDNS                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### mTLS Mode (Enterprise)

```
┌─────────────────────────────────────────────────────────────────┐
│                       mTLS Configuration                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client                              Server                      │
│  ┌─────────────────┐                ┌─────────────────┐         │
│  │ Client App      │                │ NextStepAI      │         │
│  │ ┌─────────────┐ │                │ ┌─────────────┐ │         │
│  │ │Client Cert  │ │ TLS Handshake  │ │Server Cert  │ │         │
│  │ │Private Key  │◄┼───────────────►│ │Private Key  │ │         │
│  │ └─────────────┘ │                │ └─────────────┘ │         │
│  │ ┌─────────────┐ │                │ ┌─────────────┐ │         │
│  │ │ CA Cert     │ │  Mutual Auth   │ │ CA Cert     │ │         │
│  │ │ (Verify Svr)│ │                │ │ (Verify Cli)│ │         │
│  │ └─────────────┘ │                │ └─────────────┘ │         │
│  └─────────────────┘                └─────────────────┘         │
│                                                                  │
│  Requirements:                                                   │
│  - Both parties have certificates from same CA                   │
│  - Server verifies client certificate                            │
│  - Client verifies server certificate                            │
│  - Certificate revocation checking enabled (OCSP)                │
│                                                                  │
│  Bind: 0.0.0.0:9147 (only in Elevated mode)                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.5 Audit Log Integrity

#### Hash Chain Structure

```
Event 1              Event 2              Event 3
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ id: uuid1     │    │ id: uuid2     │    │ id: uuid3     │
│ timestamp: t1 │    │ timestamp: t2 │    │ timestamp: t3 │
│ type: ...     │    │ type: ...     │    │ type: ...     │
│ data: {...}   │    │ data: {...}   │    │ data: {...}   │
│               │    │               │    │               │
│ prevHash: 0x0 │───►│ prevHash: H1  │───►│ prevHash: H2  │
│ hash: H1      │    │ hash: H2      │    │ hash: H3      │
└───────────────┘    └───────────────┘    └───────────────┘
                                                   │
                                                   ▼
                                          Current chain head

H1 = SHA256(event1.id || event1.timestamp || event1.type ||
            event1.data || event1.prevHash)

Verification: Recompute all hashes from genesis, compare to stored.
Any modification breaks the chain at that point.
```

#### Verification Process

```swift
func verifyAuditChain() async throws -> ChainVerificationResult {
    var previousHash = Data() // Genesis hash is empty
    var eventCount = 0
    var lastVerifiedTimestamp: Date?

    for try await event in auditLog.allEvents() {
        // Recompute hash
        let computedHash = computeHash(
            id: event.id,
            timestamp: event.timestamp,
            type: event.eventType,
            data: event.data,
            previousHash: previousHash
        )

        // Compare to stored hash
        guard computedHash == event.hash else {
            return .broken(
                atEvent: event.id,
                position: eventCount,
                expected: computedHash,
                found: event.hash
            )
        }

        // Compare previous hash
        guard event.previousHash == previousHash else {
            return .broken(
                atEvent: event.id,
                position: eventCount,
                message: "Previous hash mismatch"
            )
        }

        previousHash = event.hash
        eventCount += 1
        lastVerifiedTimestamp = event.timestamp
    }

    return .valid(
        eventCount: eventCount,
        lastTimestamp: lastVerifiedTimestamp
    )
}
```

---

## 8. Acceptance Criteria Checklist

### 8.1 Core Functionality

#### Chat Interface
- [ ] User can send text messages
- [ ] User can use voice input (macOS dictation)
- [ ] Responses stream in real-time
- [ ] Tool execution indicated inline
- [ ] Errors displayed with recovery options
- [ ] Conversation history persisted
- [ ] Conversation searchable

#### Browser Mode
- [ ] WebKit view renders pages correctly
- [ ] Navigation controls work (URL, back, forward, refresh)
- [ ] Only allowlisted domains accessible
- [ ] Non-allowlisted domains show block message
- [ ] User can add/remove domains from allowlist
- [ ] Page content extractable for model context
- [ ] Cookies isolated to NextStepAI

### 8.2 Security Requirements

#### Credentials
- [ ] Credentials stored in Keychain with biometric access
- [ ] Raw credential values never sent to model
- [ ] Credential handles are opaque and time-limited
- [ ] Handle resolution requires domain match
- [ ] Handle resolution requires approval
- [ ] Credential access fully logged
- [ ] Export is encrypted and rate-limited

#### Policy Engine
- [ ] All tool executions evaluated by policy engine
- [ ] Capabilities must be explicitly granted
- [ ] Expired grants rejected
- [ ] Tampered grants rejected (signature verification)
- [ ] Security levels enforced correctly
- [ ] Elevated mode requires biometric
- [ ] Lockdown mode disables risky features

#### Secrets Redaction
- [ ] API keys redacted in all logs
- [ ] Passwords redacted in all logs
- [ ] Private keys redacted in all logs
- [ ] PII redacted where configured
- [ ] Redaction tested with known secret patterns
- [ ] No secrets in model context

#### Audit Logging
- [ ] All security events logged
- [ ] Hash chain maintained correctly
- [ ] Chain verification tool works
- [ ] Logs encrypted at rest
- [ ] Log tampering detectable
- [ ] Log rotation preserves integrity

### 8.3 Browser Automation

#### Form Handling
- [ ] Login forms detected on major sites
- [ ] Registration forms detected
- [ ] Search forms detected
- [ ] Payment forms flagged as sensitive
- [ ] Form fields identifiable

#### Credential Fill
- [ ] Credential fill requires approval
- [ ] Domain verification before fill
- [ ] Phishing detection warns user
- [ ] Password masked during fill
- [ ] Fill logged with domain context

#### CAPTCHA Handling
- [ ] reCAPTCHA detected
- [ ] hCaptcha detected
- [ ] Cloudflare challenges detected
- [ ] Automation pauses on CAPTCHA
- [ ] User notified to solve manually
- [ ] Agent waits for user confirmation
- [ ] No bypass attempts ever made

### 8.4 Plugin System

#### Security
- [ ] Only signed plugins load
- [ ] Notarization verified for external plugins
- [ ] XPC sandbox enforced
- [ ] Network access limited to declared domains
- [ ] File access limited to declared scopes
- [ ] Plugin crashes isolated from main app

#### Lifecycle
- [ ] Plugin installation shows capabilities
- [ ] User must approve capabilities
- [ ] Plugin can be enabled/disabled
- [ ] Plugin updates re-verified
- [ ] Capability changes require re-approval
- [ ] Uninstall removes all plugin data

### 8.5 Remote Access

#### Default Security
- [ ] Server binds to 127.0.0.1 only by default
- [ ] External connection attempts logged
- [ ] Cannot change bind without Elevated mode

#### Authentication
- [ ] SSH tunnel mode works
- [ ] Tailnet mode works (when Tailscale present)
- [ ] mTLS mode verifies client certificates
- [ ] All modes rate-limit auth attempts
- [ ] Failed auth triggers progressive delays

#### Session Management
- [ ] Active sessions visible in UI
- [ ] Sessions show source information
- [ ] Individual sessions terminable
- [ ] "Panic button" terminates all sessions
- [ ] Session events logged

### 8.6 Performance

- [ ] App launches in < 2 seconds
- [ ] Chat response starts in < 500ms (after model starts)
- [ ] Tool execution overhead < 50ms
- [ ] Credential retrieval < 200ms (with biometric cached)
- [ ] Audit log write < 10ms (async)
- [ ] No UI freeze during tool execution
- [ ] No UI freeze during model calls

### 8.7 Privacy

- [ ] Local-only mode functions without internet
- [ ] Model provider selection configurable
- [ ] Transparency mode shows what will be sent
- [ ] Sensitive content flagged before sending
- [ ] No telemetry without consent
- [ ] Data minimization applied before model calls

### 8.8 Reliability

- [ ] App handles crash gracefully (state recovery)
- [ ] Provider timeout triggers fallback
- [ ] Plugin crash doesn't crash app
- [ ] Keychain unavailable handled gracefully
- [ ] Disk full handled (warning, not crash)
- [ ] Network loss switches to local mode

### 8.9 Accessibility

- [ ] VoiceOver navigation works
- [ ] Dynamic Type respected
- [ ] Keyboard navigation complete
- [ ] Color contrast meets WCAG AA
- [ ] Focus indicators visible
- [ ] Screen reader announces approvals

### 8.10 Documentation

- [ ] User guide complete
- [ ] Security model documented
- [ ] API documentation complete
- [ ] Plugin development guide complete
- [ ] Troubleshooting guide complete
- [ ] Security advisory process defined

---

## Appendix A: Domain Allowlist Defaults

### Default Allowlisted Domains (Browser Mode)

```
# Search engines
google.com
bing.com
duckduckgo.com

# Code hosting
github.com
gitlab.com
bitbucket.org

# Productivity
notion.so
linear.app
asana.com
trello.com
monday.com

# Documentation
stackoverflow.com
developer.apple.com
docs.microsoft.com
developer.mozilla.org

# Communication
slack.com
discord.com
teams.microsoft.com

# Cloud providers (console access)
console.aws.amazon.com
portal.azure.com
console.cloud.google.com

# Note: Login sites are allowlisted for navigation only.
# Credential fill still requires per-use approval.
```

### Categories for User Addition

When users add domains, they must select a category:

- **Productivity:** Work-related SaaS tools
- **Development:** Coding and DevOps tools
- **Finance:** Banking and financial services (extra warnings)
- **Social:** Social media platforms
- **Shopping:** E-commerce sites
- **Other:** User-specified

---

## Appendix B: Error Codes

| Code | Name | Description | User Action |
|------|------|-------------|-------------|
| E001 | `CAPABILITY_DENIED` | Required capability not granted | Approve capability or cancel |
| E002 | `SECURITY_LEVEL_BLOCKED` | Action not allowed at current level | Elevate security level |
| E003 | `DOMAIN_NOT_ALLOWED` | Domain not in allowlist | Add domain to allowlist |
| E004 | `CREDENTIAL_NOT_FOUND` | No credential for domain | Add credential |
| E005 | `CREDENTIAL_RESOLVE_FAILED` | Handle resolution failed | Re-authenticate |
| E006 | `TOOL_EXECUTION_FAILED` | Tool encountered error | See details, retry |
| E007 | `PLUGIN_NOT_SIGNED` | Plugin signature invalid | Use signed plugin |
| E008 | `PLUGIN_SANDBOX_VIOLATION` | Plugin exceeded permissions | Report to developer |
| E009 | `PROVIDER_UNAVAILABLE` | Model provider not reachable | Check connection, try local |
| E010 | `LOCKDOWN_ACTIVE` | System in lockdown mode | Authenticate to exit |
| E011 | `RATE_LIMITED` | Too many requests | Wait and retry |
| E012 | `AUDIT_CHAIN_BROKEN` | Log integrity compromised | Contact support |

---

## Appendix C: Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| New chat | ⌘N |
| Voice input | ⌘⇧Space |
| Switch to Chat Mode | ⌘1 |
| Switch to Browser Mode | ⌘2 |
| Switch to History | ⌘3 |
| Switch to Skills | ⌘4 |
| Switch to Settings | ⌘, |
| Quick approve (if one pending) | ⌘⏎ |
| Quick deny (if one pending) | ⌘⌫ |
| Trigger lockdown | ⌘⇧L |
| Search history | ⌘F |
| Close window | ⌘W |
| Quit app | ⌘Q |

---

*End of Product Specification*
