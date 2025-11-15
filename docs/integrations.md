# Integration Specifications

This document outlines the specifications for each platform integration, including authentication, data access, and implementation details.

## Integration Priority Tiers

### Tier 1: Essential (MVP)
Must-have for basic functionality:
- Slack
- Email (Gmail/Outlook)
- Calendar (Google Calendar)
- Document storage (Google Drive)

### Tier 2: High-Value
Significantly enhance capabilities:
- HR System (org chart, reporting lines)
- Project Management (Linear/Jira)
- CRM (Salesforce)

### Tier 3: Nice-to-Have
Valuable but not critical:
- GitHub/GitLab
- Data warehouse
- Finance systems

---

## Slack Integration

**Priority:** Tier 1 (Essential)

**Purpose:** Primary interface for team communication and Q&A.

### Features
- Respond to direct messages and mentions
- Answer questions in channels (when mentioned)
- Thread management for context
- Interactive buttons for escalation
- Status indicators (typing, etc.)

### APIs Required
- [Slack Web API](https://api.slack.com/web)
- [Events API](https://api.slack.com/events-api) for real-time events
- [Interactive Components](https://api.slack.com/interactivity) for buttons

### Authentication
- OAuth 2.0 with Slack
- Bot token with required scopes:
  - `chat:write` - Send messages
  - `channels:history` - Read channel messages
  - `im:history` - Read direct messages
  - `users:read` - Get user information
  - `reactions:write` - Add reactions to messages

### Data Access
```typescript
interface SlackMessage {
  channel: string;
  user: string;
  text: string;
  thread_ts?: string;  // For threading
  ts: string;          // Timestamp
}
```

### Implementation Notes
- Use Socket Mode for development (no public endpoint needed)
- Use Events API for production
- Implement retry logic for rate limits
- Support threaded conversations for context

### Configuration
```typescript
{
  appToken: 'xapp-...',      // App-level token
  botToken: 'xoxb-...',      // Bot user token
  signingSecret: '...',      // For request verification
  socketMode: true           // true for dev, false for prod
}
```

---

## Email Integration (Gmail/Outlook)

**Priority:** Tier 1 (Essential)

**Purpose:** Handle email delegation, draft responses, send on behalf of executive.

### Features
- Monitor executive's inbox
- Filter and classify incoming emails
- Draft responses for approval
- Send approved responses
- Create email summaries
- Flag urgent messages

### APIs Required

**Gmail:**
- [Gmail API](https://developers.google.com/gmail/api)
- Watch for new emails via push notifications
- Send emails using `users.messages.send`

**Outlook:**
- [Microsoft Graph API](https://docs.microsoft.com/en-us/graph/api/resources/mail-api-overview)
- Subscribe to mail events
- Send emails

### Authentication

**Gmail:**
- OAuth 2.0 with Google
- Scopes:
  - `https://www.googleapis.com/auth/gmail.readonly`
  - `https://www.googleapis.com/auth/gmail.send`
  - `https://www.googleapis.com/auth/gmail.modify`

**Outlook:**
- OAuth 2.0 with Microsoft
- Scopes:
  - `Mail.Read`
  - `Mail.Send`
  - `Mail.ReadWrite`

### Data Access
```typescript
interface EmailMessage {
  id: string;
  from: EmailAddress;
  to: EmailAddress[];
  subject: string;
  body: string;
  bodyHtml?: string;
  threadId: string;
  labels?: string[];
  receivedAt: Date;
}
```

### Implementation Notes
- Use push notifications instead of polling
- Parse HTML emails properly
- Handle attachments appropriately
- Maintain thread context
- Implement "draft mode" - always review before sending initially

### Security Considerations
- **Critical:** Never auto-send external emails without approval
- Redact sensitive information in drafts
- Log all email access
- Implement allow/block lists

---

## Calendar Integration

**Priority:** Tier 1 (Essential)

**Purpose:** Provide context about meetings, availability, and schedule.

### Features
- Fetch upcoming meetings for context
- Provide meeting participant information
- Generate pre-meeting briefs
- Track meeting attendance patterns
- Suggest meeting priority ("should I attend?")

### APIs Required

**Google Calendar:**
- [Google Calendar API](https://developers.google.com/calendar)

**Outlook Calendar:**
- [Microsoft Graph Calendar API](https://docs.microsoft.com/en-us/graph/api/resources/calendar)

### Authentication

**Google Calendar:**
- OAuth 2.0
- Scope: `https://www.googleapis.com/auth/calendar.readonly`

**Outlook Calendar:**
- OAuth 2.0
- Scope: `Calendars.Read`

### Data Access
```typescript
interface CalendarEvent {
  id: string;
  title: string;
  description?: string;
  startTime: Date;
  endTime: Date;
  attendees: Attendee[];
  location?: string;
  organizer: EmailAddress;
  isRecurring: boolean;
}
```

### Implementation Notes
- Cache calendar data to reduce API calls
- Update cache on calendar change webhooks
- Respect privacy - don't expose all calendar details
- Provide context for "Why is this meeting on my calendar?"

---

## Google Drive Integration

**Priority:** Tier 1 (Essential)

**Purpose:** Search company documents, strategy memos, policies.

### Features
- Search for documents by keyword
- Find recent documents on a topic
- Access shared team folders
- Provide document links in responses

### APIs Required
- [Google Drive API](https://developers.google.com/drive)
- [Google Docs API](https://developers.google.com/docs) for reading doc content

### Authentication
- OAuth 2.0
- Scopes:
  - `https://www.googleapis.com/auth/drive.readonly`
  - `https://www.googleapis.com/auth/documents.readonly`

### Data Access
```typescript
interface DriveDocument {
  id: string;
  name: string;
  mimeType: string;
  webViewLink: string;
  lastModified: Date;
  owners: User[];
  permissions: Permission[];
}
```

### Implementation Notes
- Index documents for faster search
- Cache document metadata
- Respect sharing permissions
- Extract text for embedding/semantic search

### Alternative: Notion/Confluence
Similar functionality for teams using Notion or Confluence instead of Google Docs.

---

## HR System Integration

**Priority:** Tier 2 (High-Value)

**Purpose:** Access org chart, reporting lines, role information.

### Features
- Query reporting relationships
- Get employee role and department
- Find team members by department
- Access tenure and background information

### APIs Required

**Workday:**
- [Workday REST API](https://community.workday.com/api)

**BambooHR:**
- [BambooHR API](https://documentation.bamboohr.com/docs)

**Rippling:**
- [Rippling API](https://developer.rippling.com/)

### Authentication
- API Key or OAuth 2.0 (varies by provider)

### Data Access
```typescript
interface Employee {
  id: string;
  name: string;
  email: string;
  role: string;
  department: string;
  reportsTo?: string;  // Manager's ID
  directReports: string[];
  startDate: Date;
}
```

### Implementation Notes
- Sync org chart daily or on webhook
- Cache in local database
- Handle organizational changes
- Privacy: Only expose necessary info

---

## Project Management Integration

**Priority:** Tier 2 (High-Value)

**Purpose:** Track project status, blockers, progress without meetings.

### Features
- Query project status
- List blockers and dependencies
- Get sprint/milestone progress
- Answer "What's the status of X?"

### APIs Required

**Linear:**
- [Linear API (GraphQL)](https://developers.linear.app/docs/graphql/working-with-the-graphql-api)

**Jira:**
- [Jira REST API](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

**Asana:**
- [Asana API](https://developers.asana.com/docs)

### Authentication

**Linear:**
- API Key or OAuth 2.0

**Jira:**
- API Token with email
- OAuth 2.0

**Asana:**
- Personal Access Token or OAuth 2.0

### Data Access
```typescript
interface Project {
  id: string;
  name: string;
  status: 'on-track' | 'at-risk' | 'off-track';
  progress: number;  // 0-100
  owner: User;
  team: User[];
  blockers: Issue[];
  milestones: Milestone[];
}

interface Issue {
  id: string;
  title: string;
  status: string;
  priority: 'low' | 'medium' | 'high' | 'urgent';
  assignee?: User;
  createdAt: Date;
  updatedAt: Date;
}
```

### Implementation Notes
- Query on-demand (don't sync all issues)
- Cache frequently accessed projects
- Subscribe to webhooks for status changes
- Provide rich status summaries

---

## CRM Integration (Salesforce)

**Priority:** Tier 2 (High-Value)

**Purpose:** Customer context, relationship history, deal status.

### Features
- Look up account information
- Get deal status and stage
- Find contact history
- Answer "What's the status with Company X?"

### APIs Required
- [Salesforce REST API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/)

### Authentication
- OAuth 2.0
- Connected App with scopes:
  - `api` - Full access
  - `refresh_token` - Offline access

### Data Access
```typescript
interface Account {
  id: string;
  name: string;
  industry: string;
  employees: number;
  annualRevenue: number;
  owner: User;
  status: string;
}

interface Opportunity {
  id: string;
  name: string;
  accountId: string;
  stage: string;
  amount: number;
  probability: number;
  closeDate: Date;
  owner: User;
}
```

### Implementation Notes
- Respect data privacy - filter by access level
- Cache account data
- Provide deal summaries
- Track executive's relationships with accounts

---

## GitHub/GitLab Integration

**Priority:** Tier 3 (Nice-to-Have)

**Purpose:** Engineering progress tracking for technical executives.

### Features
- Monitor repository activity
- Track pull request status
- Get deployment status
- Answer "What shipped this week?"

### APIs Required

**GitHub:**
- [GitHub REST API](https://docs.github.com/en/rest)
- [GitHub GraphQL API](https://docs.github.com/en/graphql)

**GitLab:**
- [GitLab REST API](https://docs.gitlab.com/ee/api/)

### Authentication

**GitHub:**
- Personal Access Token or GitHub App

**GitLab:**
- Personal Access Token or OAuth 2.0

### Data Access
```typescript
interface Repository {
  name: string;
  description: string;
  primaryLanguage: string;
  pullRequests: PullRequest[];
  deployments: Deployment[];
}

interface PullRequest {
  title: string;
  author: User;
  status: 'open' | 'merged' | 'closed';
  reviewers: User[];
  createdAt: Date;
}
```

### Implementation Notes
- Focus on high-level metrics, not code details
- Aggregate across repositories
- Weekly digests of engineering progress

---

## Common Integration Patterns

### Webhook Management
```typescript
interface WebhookConfig {
  platform: string;
  events: string[];
  endpoint: string;
  secret: string;
}

class WebhookManager {
  register(config: WebhookConfig): Promise<void>;
  verify(signature: string, body: string, secret: string): boolean;
  handleEvent(event: WebhookEvent): Promise<void>;
}
```

### Rate Limiting
```typescript
class RateLimiter {
  private limits: Map<string, RateLimit>;

  async throttle(platform: string, operation: string): Promise<void> {
    // Wait if rate limit exceeded
    // Use token bucket or sliding window algorithm
  }
}
```

### Error Handling
```typescript
class IntegrationError extends Error {
  constructor(
    public platform: string,
    public operation: string,
    public retryable: boolean,
    message: string
  ) {
    super(message);
  }
}

// Exponential backoff for retries
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  // Implement exponential backoff
}
```

### Health Checks
```typescript
interface IntegrationHealth {
  platform: string;
  status: 'healthy' | 'degraded' | 'down';
  lastCheck: Date;
  errorRate: number;
  responseTime: number;
}

class HealthMonitor {
  checkAll(): Promise<IntegrationHealth[]>;
  alert(platform: string, issue: string): void;
}
```

---

## Testing Integrations

### Unit Tests
- Mock external APIs
- Test data transformation
- Test error handling

### Integration Tests
- Use sandbox/test accounts
- Test full request/response cycle
- Verify webhooks

### Example Test Structure
```typescript
describe('SlackIntegration', () => {
  let integration: SlackIntegration;
  let mockClient: MockSlackClient;

  beforeEach(() => {
    mockClient = new MockSlackClient();
    integration = new SlackIntegration(mockClient);
  });

  it('should send message successfully', async () => {
    const message = { channel: 'C123', text: 'Hello' };
    await integration.sendMessage(message);

    expect(mockClient.sentMessages).toContainEqual(message);
  });

  it('should retry on rate limit', async () => {
    mockClient.simulateRateLimit(1); // Fail once
    await integration.sendMessage({ channel: 'C123', text: 'Hello' });

    expect(mockClient.sentMessages).toHaveLength(1);
  });
});
```

---

## Security Best Practices

1. **Credentials Management**
   - Store all secrets in environment variables
   - Use secret management service (AWS Secrets Manager, etc.)
   - Rotate credentials regularly
   - Never commit credentials to git

2. **API Access**
   - Use least-privilege scopes
   - Implement request signing
   - Validate webhook signatures
   - Use HTTPS for all communications

3. **Data Privacy**
   - Encrypt data in transit and at rest
   - Log access to sensitive data
   - Implement data retention policies
   - Respect user privacy settings

4. **Monitoring**
   - Alert on authentication failures
   - Monitor for unusual API usage
   - Track error rates by integration
   - Log all integration activities

---

## Future Integrations

Potential additions based on user needs:
- **Zoom/Teams**: Meeting transcripts, attendance
- **Slack Enterprise Grid**: Multi-workspace support
- **Datadog/Grafana**: System metrics for technical execs
- **Stripe**: Financial metrics for finance-focused execs
- **HubSpot**: Marketing automation context
- **Greenhouse**: Recruiting pipeline status
