# System Architecture

## Overview

The Digital Chief of Staff is built as a **modular, event-driven system** that integrates with multiple platforms, processes information through AI, and provides intelligent responses aligned with executive context.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    External Platforms                        │
│  Slack │ Email │ Calendar │ Drive │ HR │ Jira │ CRM │ ...   │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                   Integration Layer                          │
│  Platform-specific connectors, auth, webhooks, polling      │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                      Core Engine                             │
│  • Message Router                                            │
│  • Context Builder                                           │
│  • Decision Engine                                           │
│  • Response Generator                                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    AI/LLM Layer                              │
│  • Prompt Engineering                                        │
│  • Model Selection                                           │
│  • Response Synthesis                                        │
│  • Learning & Adaptation                                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    Data Layer                                │
│  • User Context DB                                           │
│  • Conversation History                                      │
│  • Decision Log                                              │
│  • Knowledge Base                                            │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Integration Layer (`src/integrations/`)

Platform-specific connectors that handle authentication, webhooks, and API calls.

**Responsibilities:**
- Authenticate with external platforms
- Receive incoming messages/events
- Send outgoing responses
- Handle rate limiting and retries
- Normalize data into common format

**Key Integrations:**
```typescript
// Each integration implements a common interface
interface PlatformIntegration {
  authenticate(): Promise<void>;
  sendMessage(recipient: string, message: Message): Promise<void>;
  fetchContext(query: ContextQuery): Promise<Context>;
  subscribe(event: EventType, handler: EventHandler): void;
}
```

**Example Modules:**
- `slack/` - Slack Bot API, Events API, Interactive Components
- `email/` - Gmail/Outlook API, SMTP, IMAP
- `calendar/` - Google Calendar, Outlook Calendar
- `gdrive/` - Google Drive API for document search
- `hr/` - Workday, BambooHR, etc. for org chart

### 2. Core Engine (`src/core/`)

Business logic for routing, context building, and decision-making.

#### Message Router
```typescript
class MessageRouter {
  route(message: IncomingMessage): Promise<MessageRoute> {
    // 1. Identify sender and their relationship to exec
    // 2. Classify message type (question, request, FYI)
    // 3. Determine urgency and importance
    // 4. Route to appropriate handler
  }
}
```

#### Context Builder
```typescript
class ContextBuilder {
  async buildContext(message: Message): Promise<Context> {
    // Gather relevant context from multiple sources:
    // - Sender's role, reporting relationship, recent interactions
    // - Related documents, previous decisions
    // - Current strategic priorities
    // - Recent conversations on similar topics
    return context;
  }
}
```

#### Decision Engine
```typescript
class DecisionEngine {
  async decide(message: Message, context: Context): Promise<Decision> {
    // Determine action: auto-respond, draft for review, escalate, delegate
    // Based on: confidence, sensitivity, precedent, executive preferences
    return {
      action: 'draft_for_review',
      confidence: 0.85,
      reasoning: 'Novel strategic question requiring exec judgment'
    };
  }
}
```

#### Response Generator
```typescript
class ResponseGenerator {
  async generate(
    message: Message,
    context: Context,
    tone: CommunicationStyle
  ): Promise<Response> {
    // Generate response using AI with executive's voice
    // Apply tone, style, terminology from learned patterns
    return response;
  }
}
```

### 3. AI/LLM Layer (`src/ai/`)

Handles all interactions with language models.

**Key Components:**
- **Prompt Manager**: Constructs prompts with context, examples, guidelines
- **Model Router**: Selects appropriate model (fast/cheap vs. powerful)
- **Response Synthesizer**: Combines multiple sources into coherent response
- **Learning Engine**: Learns from executive feedback on drafts

**Example Flow:**
```typescript
// Prompt construction with executive context
const prompt = `
You are drafting a response on behalf of [Executive Name], [Title].

EXECUTIVE CONTEXT:
- Communication style: ${executiveStyle}
- Current priorities: ${currentPriorities}
- Related past decisions: ${pastDecisions}

INCOMING MESSAGE:
From: ${sender.name} (${sender.role})
Subject: ${message.subject}
Body: ${message.body}

TASK: Draft a response that:
1. Addresses the question directly
2. Maintains executive's authentic voice
3. References relevant context/decisions
4. Ends with appropriate next steps

RESPONSE:
`;

const response = await llm.complete(prompt);
```

### 4. Data Layer

**PostgreSQL Schema (simplified):**

```sql
-- User and org structure
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE,
  name VARCHAR(255),
  role VARCHAR(100),
  reports_to UUID REFERENCES users(id),
  access_level VARCHAR(50)
);

-- Conversation history
CREATE TABLE conversations (
  id UUID PRIMARY KEY,
  platform VARCHAR(50),  -- 'slack', 'email', etc.
  sender_id UUID REFERENCES users(id),
  message_text TEXT,
  response_text TEXT,
  confidence FLOAT,
  action_taken VARCHAR(50),  -- 'auto_sent', 'escalated', etc.
  created_at TIMESTAMP
);

-- Executive feedback for learning
CREATE TABLE feedback (
  id UUID PRIMARY KEY,
  conversation_id UUID REFERENCES conversations(id),
  original_draft TEXT,
  exec_edit TEXT,
  feedback_type VARCHAR(50),  -- 'tone', 'content', 'escalation'
  created_at TIMESTAMP
);

-- Institutional knowledge
CREATE TABLE decisions (
  id UUID PRIMARY KEY,
  topic VARCHAR(255),
  decision TEXT,
  context TEXT,
  made_by UUID REFERENCES users(id),
  made_at TIMESTAMP,
  tags TEXT[]
);

-- Strategic priorities (updated quarterly)
CREATE TABLE priorities (
  id UUID PRIMARY KEY,
  title VARCHAR(255),
  description TEXT,
  importance INT,  -- 1-10
  quarter VARCHAR(10),  -- '2024-Q1'
  created_at TIMESTAMP
);
```

**Vector Database (for semantic search):**
- Use Pinecone, Weaviate, or pgvector
- Store embeddings of:
  - Past conversations
  - Company documents
  - Decision logs
  - Strategy memos

## Data Flow

### Example: Slack Question Flow

```
1. Slack message arrives
   ↓
2. Slack integration receives webhook
   ↓
3. Message Router classifies: "Question about vacation policy"
   ↓
4. Context Builder gathers:
   - Sender: Alice (Senior Engineer, reports to exec)
   - Related docs: HR policies, past vacation discussions
   - Confidence: High (common question with clear answer)
   ↓
5. Decision Engine determines: Auto-respond
   ↓
6. Response Generator drafts:
   "Our vacation policy is flexible PTO. Details: [link]
    Check with your manager for specific timing coordination."
   ↓
7. Slack integration sends response
   ↓
8. Log conversation for future learning
```

### Example: Complex Email Flow

```
1. Email arrives: "Should we pivot our Q3 strategy?"
   ↓
2. Email integration processes
   ↓
3. Message Router: Strategic question, high importance
   ↓
4. Context Builder:
   - Sender: VP Product (direct report)
   - Related: Q3 planning docs, recent board meeting notes
   - Confidence: Low (requires executive judgment)
   ↓
5. Decision Engine: Draft for review + escalate
   ↓
6. Response Generator drafts:
   "Important question. Let's discuss in our 1:1 tomorrow.
    In the meantime, can you send me the data that's prompting
    this question?"
   ↓
7. Send draft to exec for approval in Slack
   ↓
8. Exec approves or edits
   ↓
9. Learn from any edits made
   ↓
10. Send approved response via email
```

## Security Architecture

### Access Control

```typescript
enum AccessLevel {
  NONE = 0,           // External users
  LIMITED = 1,        // Cross-functional partners
  STANDARD = 2,       // Skip-level reports
  FULL = 3,           // Direct reports
  ADMIN = 4           // Executive + admins
}

class AccessControl {
  canQuery(user: User, topic: string): boolean {
    // Check if user has permission to ask about this topic
  }

  filterResponse(response: Response, userLevel: AccessLevel): Response {
    // Redact sensitive information based on access level
  }
}
```

### Data Privacy

- **Encryption at rest**: All data encrypted in PostgreSQL
- **Encryption in transit**: TLS for all external communications
- **Audit logging**: All queries and responses logged with timestamps
- **Data retention**: Configurable retention policies (default: 2 years)
- **GDPR compliance**: Data export and deletion capabilities

### Authentication

- **SSO integration**: Google Workspace, Okta, Azure AD
- **OAuth 2.0**: For platform integrations
- **API keys**: For programmatic access (with strict scoping)
- **MFA**: Required for admin access

## Scalability Considerations

### Current Phase (MVP)
- Single-tenant deployment
- Monolithic Node.js application
- PostgreSQL with connection pooling
- Handles ~1000 messages/day

### Future Scaling
- **Microservices**: Split integrations into separate services
- **Message queue**: Redis/RabbitMQ for async processing
- **Caching**: Redis for frequently accessed context
- **Load balancing**: Multiple API servers behind load balancer
- **Database sharding**: By organization or time period

## Technology Stack

**Backend:**
- Node.js 18+ with TypeScript
- Express.js for API endpoints
- PostgreSQL 14+ for relational data
- Redis for caching and queues
- Pinecone/pgvector for vector search

**AI/LLM:**
- OpenAI GPT-4 for complex reasoning
- Anthropic Claude for long context
- OpenAI embeddings for semantic search
- Custom fine-tuned models (future)

**Infrastructure:**
- Docker for containerization
- Kubernetes for orchestration (production)
- GitHub Actions for CI/CD
- AWS/GCP for cloud hosting
- Terraform for infrastructure as code

**Monitoring:**
- Prometheus for metrics
- Grafana for dashboards
- Sentry for error tracking
- DataDog for APM

## Development Principles

1. **Modular**: Each integration is self-contained
2. **Testable**: Mock external APIs, test core logic independently
3. **Observable**: Comprehensive logging and metrics
4. **Fail-safe**: Graceful degradation if services are down
5. **Privacy-first**: Minimal data retention, strict access control

## Future Enhancements

- **Real-time collaboration**: Multiple execs on same platform
- **Voice interface**: Integration with voice assistants
- **Mobile app**: Native iOS/Android apps
- **Advanced learning**: Fine-tuned models per executive
- **Proactive insights**: Surfacing trends before asked
