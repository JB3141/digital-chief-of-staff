# Contributing to Digital Chief of Staff

Thank you for your interest in contributing! This document provides guidelines for collaborating effectively on this project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Branching Strategy](#branching-strategy)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Testing Requirements](#testing-requirements)
- [Avoiding Conflicts](#avoiding-conflicts)

## Code of Conduct

- Be respectful and constructive in all interactions
- Focus on what's best for the project and users
- Welcome newcomers and help them get started
- Give and receive feedback gracefully

## Getting Started

### Prerequisites

- Node.js 18 or higher
- PostgreSQL 14 or higher
- Git
- Your preferred code editor (VS Code recommended)

### Initial Setup

1. **Fork and clone the repository**
   ```bash
   git clone https://github.com/YOUR-USERNAME/digital-chief-of-staff.git
   cd digital-chief-of-staff
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your local configuration
   ```

4. **Run database migrations**
   ```bash
   npm run db:migrate
   ```

5. **Verify setup**
   ```bash
   npm test
   npm run dev
   ```

## Development Workflow

### Before Starting Work

1. **Check existing issues** - See if the feature/bug is already being worked on
2. **Create or claim an issue** - Describe what you plan to do
3. **Discuss approach** - For large changes, get feedback on your approach first
4. **Pull latest changes** - Always start from the latest `main` branch

### Daily Workflow

```bash
# Start your day by pulling latest changes
git checkout main
git pull origin main

# Create a feature branch
git checkout -b feature/your-feature-name

# Make changes, commit frequently with clear messages
git add .
git commit -m "Add Slack message parsing logic"

# Push to your fork
git push origin feature/your-feature-name

# Open a Pull Request on GitHub
```

## Branching Strategy

### Branch Naming Convention

- **Feature**: `feature/slack-integration`, `feature/meeting-intelligence`
- **Bug fix**: `fix/email-parsing-error`, `fix/auth-token-expiry`
- **Documentation**: `docs/api-endpoints`, `docs/setup-guide`
- **Refactor**: `refactor/extract-ai-service`, `refactor/consolidate-utils`

### Branch Lifecycle

1. **Create** from latest `main`
2. **Develop** with frequent commits
3. **Push** to your fork regularly
4. **Open PR** when ready for review
5. **Address feedback** with additional commits
6. **Merge** after approval (squash merge preferred)
7. **Delete** branch after merge

### Protected Branches

- **`main`** - Production-ready code
  - Cannot push directly
  - Requires PR with 1+ approval
  - All CI checks must pass
  - Linear history enforced

## Pull Request Process

### Before Opening a PR

- [ ] Code follows project style guidelines
- [ ] All tests pass locally (`npm test`)
- [ ] New tests added for new features
- [ ] Documentation updated if needed
- [ ] No merge conflicts with `main`
- [ ] Commit messages are clear and descriptive

### PR Description Template

```markdown
## Summary
Brief description of what this PR does.

## Changes
- Added Slack bot message parsing
- Implemented response generation using OpenAI
- Added unit tests for message handlers

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manually tested in dev environment

## Related Issues
Closes #123
Related to #456

## Screenshots (if applicable)
[Add screenshots for UI changes]
```

### Review Process

1. **Automated checks** run on every PR (linting, tests, build)
2. **Code review** by at least one maintainer
3. **Address feedback** - Make requested changes
4. **Approval** - PR approved by reviewer
5. **Merge** - Squash and merge to `main`

### Getting Your PR Reviewed Quickly

- Keep PRs **small and focused** (< 400 lines changed)
- Write **clear descriptions** of what and why
- Add **tests** for new functionality
- Respond to feedback **promptly**
- Tag relevant reviewers using `@username`

## Coding Standards

### TypeScript/JavaScript

- Use **TypeScript** for all new code
- Follow **ESLint** configuration
- Use **async/await** over raw promises
- Prefer **functional** programming patterns where appropriate

### Code Style

```typescript
// Good: Clear function names, typed parameters
async function fetchUserContext(userId: string): Promise<UserContext> {
  const user = await db.users.findById(userId);
  return {
    name: user.name,
    role: user.role,
    permissions: await getUserPermissions(userId)
  };
}

// Bad: Unclear naming, no types
async function get(id) {
  const u = await db.users.findById(id);
  return { n: u.name, r: u.role };
}
```

### File Organization

```typescript
// 1. Imports (grouped: external, internal, types)
import express from 'express';
import { SlackClient } from '../integrations/slack';
import type { Message } from '../types';

// 2. Types and interfaces
interface HandlerOptions {
  timeout: number;
}

// 3. Constants
const DEFAULT_TIMEOUT = 5000;

// 4. Main logic
export class MessageHandler {
  // ...
}

// 5. Helpers (if small, otherwise separate file)
function sanitizeInput(text: string): string {
  // ...
}
```

### Documentation

- **JSDoc comments** for public APIs
- **Inline comments** for complex logic
- **README** in each major directory explaining purpose

```typescript
/**
 * Processes incoming Slack messages and generates appropriate responses.
 *
 * @param message - The incoming Slack message
 * @param context - User and organizational context
 * @returns Promise resolving to the generated response
 * @throws {AuthError} If user lacks required permissions
 */
async function processMessage(
  message: SlackMessage,
  context: Context
): Promise<Response> {
  // ...
}
```

## Testing Requirements

### Test Coverage

- **Minimum 80%** code coverage for new code
- **Unit tests** for business logic
- **Integration tests** for API endpoints
- **E2E tests** for critical user flows

### Test Structure

```typescript
describe('MessageHandler', () => {
  describe('processMessage', () => {
    it('should handle simple questions', async () => {
      const message = createMockMessage('What is our vacation policy?');
      const response = await handler.processMessage(message);

      expect(response.text).toContain('vacation policy');
      expect(response.confidence).toBeGreaterThan(0.8);
    });

    it('should escalate complex questions', async () => {
      const message = createMockMessage('Should we acquire Company X?');
      const response = await handler.processMessage(message);

      expect(response.shouldEscalate).toBe(true);
    });
  });
});
```

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test -- path/to/test.spec.ts
```

## Avoiding Conflicts

### Communication

- **Announce your work** - Comment on the issue you're working on
- **Coordinate on features** - Discuss who's working on what
- **Use draft PRs** - Show work-in-progress early
- **Daily updates** - Brief async updates in Discussions or Slack

### Module Ownership

To minimize conflicts, we use a **module ownership** model:

| Module | Owner | Description |
|--------|-------|-------------|
| `src/integrations/slack/` | TBD | Slack bot and API integration |
| `src/integrations/email/` | TBD | Email processing and sending |
| `src/integrations/calendar/` | TBD | Calendar integration |
| `src/core/` | Shared | Core business logic |
| `src/ai/` | TBD | LLM integration and prompts |
| `src/api/` | TBD | REST/GraphQL API |

**Ownership means:**
- You review PRs touching your module
- Others should coordinate with you before major changes
- You're responsible for that module's tests and documentation

### Preventing Merge Conflicts

1. **Pull frequently** - `git pull origin main` at least daily
2. **Small PRs** - Easier to review, less conflict potential
3. **Separate files** - Work on different files when possible
4. **Communicate** - "I'm refactoring the AI module this week"

### Resolving Conflicts

If you encounter merge conflicts:

```bash
# Update your branch with latest main
git checkout main
git pull origin main
git checkout your-feature-branch
git rebase main

# Resolve conflicts in your editor
# After resolving:
git add .
git rebase --continue

# Force push (your branch only!)
git push --force-with-lease origin your-feature-branch
```

## Project Conventions

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```bash
feat: add Slack message threading support
fix: resolve email parsing for HTML messages
docs: update API documentation for /query endpoint
test: add integration tests for calendar sync
refactor: extract response generation into separate service
```

### Issue Labels

- `bug` - Something isn't working
- `feature` - New functionality
- `documentation` - Documentation improvements
- `good first issue` - Good for newcomers
- `help wanted` - Extra attention needed
- `integration` - Related to external integrations
- `ai` - AI/LLM related work

## Need Help?

- **Questions?** - Open a [Discussion](https://github.com/yourusername/digital-chief-of-staff/discussions)
- **Bug?** - Create an [Issue](https://github.com/yourusername/digital-chief-of-staff/issues)
- **Stuck?** - Tag maintainers in your PR or issue

Thank you for contributing to Digital Chief of Staff!
