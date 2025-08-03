# PRD Agent Usage Guide & Examples

## Getting Started

### Installation
```bash
# Install Claude Code CLI if not already installed
npm install -g claude-code

# Save the PRD agent prompt
claude-code agent create prd-writer --file prd-agent-prompt.md
```

### Basic Usage
```bash
# Start a new PRD
claude-code chat -a prd-writer "Create a PRD for [feature description]"

# Continue working on existing PRD
claude-code chat -a prd-writer --continue
```

## Effective Input Strategies

### 1. Minimal Input (Let Agent Lead)
Best for: When you want the agent to structure everything

```
Create a PRD for a user notification system that sends email and in-app notifications based on user preferences.
```

### 2. Structured Input (Guided Creation)
Best for: When you have specific requirements

```
Create a PRD for a notification system with these requirements:
- Users can opt in/out of email and in-app notifications separately
- Notifications should be batched (max 1 email per hour)
- Must integrate with existing UserPreferences service
- Need real-time in-app notifications via WebSocket
- Support notification templates with variables
- Track open/click rates
```

### 3. Context-Rich Input (Optimal Results)
Best for: Complex features needing specific architectural alignment

```
Create a PRD for a notification system.

Context:
- We're using Node.js with TypeScript
- Current architecture uses event-driven microservices
- We have an existing EventBus using RabbitMQ
- User service exposes REST APIs at api.example.com/users
- Frontend is React with Redux

Requirements:
- Email and in-app notifications with user preferences
- Template system for consistent messaging
- Analytics tracking for engagement
- Must handle 10k notifications/minute at peak

Constraints:
- Cannot modify existing User service
- Must use existing authentication (JWT)
- 99.9% uptime requirement
```

## Example Interactions

### Example 1: E-commerce Feature
```
Human: Create a PRD for a product recommendation engine that shows personalized recommendations on the homepage and product pages.

Agent: I'll create a comprehensive PRD for your product recommendation engine. Let me structure this to optimize both for human collaboration and Taskmaster AI processing.

[Agent generates full PRD with sections for ML model integration, data pipeline, API design, and includes collaboration markers for business rules]
```

### Example 2: Iterative Refinement
```
Human: The notification system PRD looks good, but we need to add support for SMS notifications and the ability to schedule notifications for future delivery.

Agent: I'll update the PRD to include SMS notifications and scheduled delivery. Let me add these features while maintaining compatibility with the existing structure.

[Agent updates relevant sections, adds new feature blocks for Taskmaster, and marks changes with version numbers]
```

### Example 3: Technical Constraint Focus
```
Human: We need to follow our company's hexagonal architecture pattern. How should I phrase my request?

Agent: Here's an optimal way to request a PRD with specific architectural constraints:

"Create a PRD for [feature]. Our codebase follows hexagonal architecture with these conventions:
- Domain logic in core/ directory
- Ports defined as interfaces in ports/
- Adapters in adapters/ (separate for inbound/outbound)
- Use dependency injection with InversifyJS
- All business logic must be framework-agnostic"
```

## Best Practices for Product Leads

### 1. Provide Business Context
```
Good: "Create a PRD for a referral system. Our goal is to increase user acquisition by 30% in Q2. Average customer lifetime value is $500, and we can spend up to $50 per acquisition."

Less Effective: "Create a PRD for a referral system."
```

### 2. Specify Integration Points
```
Good: "The feature needs to integrate with:
- Stripe for payment processing
- SendGrid for emails  
- Segment for analytics
- Our existing user-service at port 3001"

Less Effective: "It should work with our existing systems."
```

### 3. Include Non-Functional Requirements Early
```
Good: "Must support 1000 concurrent users, page load under 2s, WCAG 2.1 AA compliance, and work offline after initial load."

Less Effective: "Should be fast and accessible."
```

## Collaboration Workflow

### Step 1: Initial PRD Generation
```bash
claude-code chat -a prd-writer "Create PRD for [feature]" > feature-prd-v1.md
```

### Step 2: Review and Mark Questions
Open the PRD and look for:
- 🤔 DECISION NEEDED: markers
- <!-- COLLABORATION ZONE --> sections
- [PLACEHOLDER] tags

### Step 3: Refine with Answers
```bash
claude-code chat -a prd-writer --file feature-prd-v1.md "
Here are my answers to the open questions:
- Question 1: Go with Option A because...
- Question 2: The budget is $50k
- Question 3: We need to support iOS 14+"
```

### Step 4: Prepare for Taskmaster
```bash
claude-code chat -a prd-writer "Review this PRD and ensure all sections are complete for Taskmaster processing. Add any missing test scenarios or technical specifications."
```

### Step 5: Generate Tasks
```bash
# Feed to Taskmaster
taskmaster generate --prd feature-prd-final.md --output-format jira
```

## Common Pitfalls to Avoid

### 1. Vague Requirements
❌ "Make it user-friendly"
✅ "Users should be able to complete the workflow in under 3 clicks with clear visual feedback at each step"

### 2. Missing Technical Context
❌ "Use best practices"
✅ "Follow our REST API conventions from api-guidelines.md, use our standard error format, implement retry logic with exponential backoff"

### 3. Assuming Knowledge
❌ "Integrate with the payment system"
✅ "Integrate with Stripe Connect API v2023-10-16, using our existing StripeService wrapper in services/payment/"

### 4. Incomplete Acceptance Criteria
❌ "Notifications should work"
✅ "Notifications should: 1) Deliver within 30s, 2) Retry 3 times on failure, 3) Log delivery status, 4) Fall back to email if push fails"

## Advanced Tips

### Using Templates
Create domain-specific templates:
```bash
# Save a template
claude-code chat -a prd-writer "Create a PRD template for payment features" > templates/payment-prd.md

# Use template
claude-code chat -a prd-writer "Using the payment PRD template, create a PRD for subscription management"
```

### Bulk Processing
For multiple related features:
```bash
# Create a batch file
echo "Create PRDs for these related features:
1. User authentication with OAuth
2. User profile management  
3. User preferences API
Ensure they share common data models and follow the same patterns." > batch-request.txt

claude-code chat -a prd-writer --file batch-request.txt
```

### Integration with CI/CD
```yaml
# .github/workflows/prd-review.yml
name: PRD Validation
on:
  pull_request:
    paths:
      - 'docs/prds/*.md'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Validate PRD Structure
        run: |
          claude-code chat -a prd-writer "Validate this PRD has all required sections for Taskmaster" --file ${{ github.event.pull_request.files }}
```

## Measuring Success

Good PRDs created with this agent should:
1. Reduce back-and-forth clarifications by 80%
2. Enable Taskmaster to generate complete task sets without missing features
3. Include specific test scenarios for 100% of acceptance criteria
4. Reference existing code patterns to maintain consistency
5. Have zero ambiguous requirements after collaboration

Remember: The goal is to create PRDs that are simultaneously human-friendly for collaboration and machine-optimized for task generation.
