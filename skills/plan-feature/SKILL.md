---
name: plan-feature
description: Create feature planning documents in docs/features. Start with PRD, then break into implementation phases.
argument-hint: [feature-name]
---

# Plan Feature

Create feature documentation in `docs/features/[feature-name]/`.

## Directory Structure

```
docs/features/[feature-name]/
├── prd.md                      # Product requirements (non-technical)
├── 001-backend-database.md     # Phase 1: Database schema
├── 002-backend-api.md          # Phase 2: API endpoints
├── 003-frontend-components.md  # Phase 3: UI components
└── ...                         # Additional phases as needed
```

## Step 1: Product Requirements Document (PRD)

`docs/features/[feature-name]/prd.md`

```markdown
# [Feature Name]

## Overview

What is this feature and why are we building it?

## Problem Statement

What problem does this solve for users?

## Goals

- Goal 1
- Goal 2

## User Stories

### As a [user type]
- I want to [action]
- So that [benefit]

## Requirements

### Must Have
- Requirement 1
- Requirement 2

### Nice to Have
- Optional requirement 1

## User Flow

1. User does X
2. System responds with Y
3. User sees Z

## Success Metrics

- Metric 1
- Metric 2

## Out of Scope

- What we're NOT building
```

## Step 2: Break Into Implementation Phases

After PRD is approved, create implementation phases.

### Phase Naming

```
XXX-[backend|frontend]-[description].md
```

- `XXX` - Three-digit sequence (001, 002, 003)
- Backend phases come before frontend
- Each phase should be small and focused

### Backend Phase Template

```markdown
# [Feature] - Backend: [Phase Description]

## Overview

What this phase implements.

## Dependencies

- Depends on: (previous phases if any)
- Blocks: (phases that depend on this)

## Database Changes

### New Tables
\`\`\`typescript
// packages/db/src/schemas/[entity].ts
\`\`\`

### Migrations
- [ ] Generate migration
- [ ] Run migration

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | /path | Description |

## Implementation Checklist

### Database
- [ ] Create schema
- [ ] Create repository
- [ ] Export from index

### Service
- [ ] Create service module
- [ ] Add error classes
- [ ] Export from package.json

### Routes
- [ ] Create routes
- [ ] Register in index

## Verification

\`\`\`bash
# Run the project's check/test commands
\`\`\`
```

### Frontend Phase Template

```markdown
# [Feature] - Frontend: [Phase Description]

## Overview

What this phase implements.

## Dependencies

- Depends on: XXX-backend-api.md
- API endpoints required: GET /path, POST /path

## Components

| Component | Purpose |
|-----------|---------|
| ComponentName | What it does |

## Implementation Checklist

### API Client
- [ ] Add endpoints
- [ ] Add types

### Feature Structure
- [ ] Create feature directory
- [ ] Create mutation hooks
- [ ] Create components

### Pages
- [ ] Create page

## Verification

\`\`\`bash
# Run the project's check command
\`\`\`
```

## Example: Contact Import Feature

```
docs/features/contact-import/
├── prd.md
├── 001-backend-csv-parsing.md
├── 002-backend-import-api.md
├── 003-frontend-import-dialog.md
└── 004-frontend-progress-display.md
```

## Rules

1. **PRD first** - Always start with product requirements
2. **Non-technical PRD** - Focus on user needs, not implementation
3. **Small phases** - Each phase should be completable independently
4. **Backend before frontend** - API must exist before UI
5. **Clear dependencies** - State what each phase depends on
6. **Checkboxes** - Track implementation progress
