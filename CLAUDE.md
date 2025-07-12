# AI AGENT DEVELOPMENT RULES

**⚠️ THESE ARE LAWS, NOT SUGGESTIONS ⚠️**
**VIOLATION = FAILURE**
**NO EXCEPTIONS, NO INTERPRETATIONS**

## DEFINITIONS & ENTITIES

### Core Entities
- **PBI (Product Backlog Item)**: A user story defining business value, identified by a unique immutable number (e.g., "7"). PBI IDs are permanent - they never change even when split or reordered
- **Task**: A unit of work within a PBI, identified as pbi-number (e.g., "7-1", "7-13") - NO dots or hierarchical numbering
- **Current Task Indicator**: A file named _current-task-{task-id} or _current-task-none (EMPTY file) in docs/delivery/
- **Backlog**: The master list at docs/delivery/backlog.md (ordered by priority, highest first)
- **User**: The human partner who defines requirements and approves changes
- **AI Agent**: The assistant executing tasks according to these rules

### Valid PBI Statuses
- **Proposed**: Suggested but not approved
- **Agreed**: Approved and ready for implementation
- **InProgress**: Being actively worked on
- **InReview**: Implementation complete, awaiting review
- **Done**: Completed and accepted
- **Rejected**: Rejected, requires rework

### Valid Task Statuses
- **Proposed**: Initial state of newly defined task
- **Agreed**: User approved task description and priority
- **InProgress**: AI Agent actively working on task
- **Review**: Work complete, awaits User validation
- **Done**: User reviewed and approved implementation
- **Blocked**: Cannot proceed due to external dependency

### Key Concepts
- **Hook**: Automated code quality check that runs after file modifications
- **Exit Code 2**: Hook failure requiring immediate fix
- **DRY (Don't Repeat Yourself)**: Define information once, reference elsewhere
- **CoS (Conditions of Satisfaction)**: Acceptance criteria for a PBI
- **PRD (Product Requirements Document)**: Detailed requirements for a product/feature

## CORE RULES

### LAWS (VIOLATE = IMMEDIATE STOP)
- Never change code without a task ❌ STOP if violated
- Never create a task without a PBI ❌ STOP if violated
- Never create files outside defined structures ❌ STOP if violated
- The User decides all scope ❌ STOP if violated
- No duplication - follow DRY principle ❌ STOP if violated
- Test changes MUST be in the same commit as code changes ❌ STOP if violated
- ALL hook issues are BLOCKING - EVERYTHING must be ✅ GREEN ❌ STOP if violated
- Zero tolerance: No errors, formatting issues, or linting problems ❌ STOP if violated
- PBI IDs are immutable - they NEVER change ❌ STOP if violated

## WORKFLOWS

### PBI Management

When creating a PBI:
- Set status to Proposed
- Must have unique ID and clear title
- Log creation with timestamp and user

When User approves PBI:
- If has requirements AND aligns with PRD
- Then set status to Agreed
- And create docs/delivery/{pbi}/prd.md
- And notify stakeholders
- And log approval with timestamp

When starting a PBI:
- If no other PBI is InProgress for same component
- Then set status to InProgress
- And create tasks
- And assign initial tasks
- And log start

When submitting PBI for review:
- If all tasks complete AND acceptance criteria met
- Then set status to InReview
- And update documentation
- And notify reviewers
- And log submission

When approving PBI:
- From InReview to Done
- If all acceptance criteria verified AND tests pass
- Then update completion date
- And archive related tasks
- And log approval
- And notify stakeholders

When rejecting PBI:
- From InReview to Rejected
- Document rejection reasons
- Identify required changes
- Update with review feedback
- Log rejection
- Notify team

When reopening PBI:
- From Rejected to InProgress
- If feedback addressed
- Then update PBI with changes
- And log reopening
- And notify team

When deprioritizing PBI:
- From Agreed or InProgress to Proposed
- Document reason
- Pause any work
- Update priority
- Log deprioritization
- Notify stakeholders

When splitting PBI:
- Original PBI gets "Split" event in history
- Create new PBIs with next available unique IDs
- New PBIs reference original in creation history
- Move original PBI's folder to first new PBI's ID
- Create folder for second new PBI
- Update all internal references to use new PBI numbers
- Original PBI is effectively replaced, not kept
- Backlog maintains priority order (not ID order)

### Task Management

When User approves a task:
- Create file {pbi}-{task}.md in docs/delivery/{pbi}/
- Link it in tasks.md as [{name}](./{pbi}-{task}.md)
- Analyze requirements and document in task file
- Design implementation approach
- Create test plan proportional to complexity
- Set status to Agreed

When starting a task:
- If no sibling tasks are InProgress
- Then run: git checkout -b task-{task}
- And rename _current-task-* to _current-task-{task}
- And set status to InProgress
- And log start time

When submitting task for review:
- If requirements met AND tests pass
- Then update documentation
- And create pull request
- And notify User
- And set status to Review
- And log submission

When User approves task review:
- Verify all acceptance criteria met
- Merge to main branch
- Run: git acp "{task} {description}"
- Ask User if next tasks still relevant
- Update logbook if took long OR had challenges OR deviated
- Archive task documentation
- Set status to Done
- Log completion

When User rejects task review:
- Document rejection reasons
- Identify required changes
- Update task with feedback
- Set status to InProgress
- Log rejection
- Create new tasks if scope expanded

When significant update during review:
- Document nature of changes
- Update requirements/implementation/test plans
- Set status to InProgress
- Log update reason
- Resume development

When marking task blocked:
- Document blockers in detail
- Identify dependencies
- Notify stakeholders
- Consider creating tasks for blockers
- Set status to Blocked
- Log blocking reason

When unblocking task:
- Document resolution
- Update task documentation
- Resume work
- Set status to InProgress
- Log unblocking

When handling bugs:
- If bug relates to current task: fix it now
- If bug relates to current PBI: create new task
- If bug unrelated: create new PBI with user-focused title
- Always follow User direction

### REALITY CHECKPOINTS & VALIDATION

Stop and validate at:
- After complete feature
- Before new major component
- When something feels wrong
- Before declaring "done"
- WHEN HOOKS FAIL WITH ERRORS ❌

Run: `make fmt && make test && make lint`

Why: Prevents cascading failures

### Hook Failure Protocol
When hooks report ANY issues (exit code 2):
1. STOP IMMEDIATELY - Do not continue
2. FIX ALL ISSUES - Address every ❌ until ✅ GREEN
3. VERIFY THE FIX - Re-run failed command
4. CONTINUE ORIGINAL TASK - Return to interrupted work
5. NEVER IGNORE - No warnings, only requirements

Includes: formatting, linting, forbidden patterns, ALL checks
Code must be 100% clean - no exceptions

Recovery: Maintain awareness of original task, use task tracking

## MANDATORY STANDARDS

### Change Management
- ANY code change conversation MUST start by identifying the task
- If User requests change without task reference, STOP
- Ask if it relates to existing task or needs new PBI+task
- No gold plating - stick to task scope exactly
- Sense-check all data for consistency
- Any scope creep must be rolled back
- NEVER implement "fallbacks" as part of building a solution. They obscure bugs, waste resources and confuse everything. 

### TYPESCRIPT-SPECIFIC RULES
- Use `bun` and `bunx` for package management (never npm)
- NEVER use timeouts to try and resolve timing issues. The golden rule: if there's a race condition, you just haven't found the right event to listen to. 

### PYTHON-SPECIFIC RULES
- Use `uv` and `uvx` for package management (never pip)
- Always create a virtual environment `uv venv -p <appropriate version of python>`
- use `uv add` for adding packages (not uv pip install). 

### GO-SPECIFIC RULES

#### FORBIDDEN - NEVER DO THESE
- NO interface{} or any{} - use concrete types!
- NO time.Sleep() or busy waits - use channels for synchronization!
- NO keeping old and new code together
- NO migration functions or compatibility layers
- NO versioned function names (processV2, handleNew)
- NO custom error struct hierarchies
- NO TODOs in final code

AUTOMATED ENFORCEMENT: smart-lint hook BLOCKS commits violating these
When see `❌ FORBIDDEN PATTERN`, MUST fix immediately!

#### REQUIRED STANDARDS
- Delete old code when replacing it. We have version control
- never create duplciate copies of files with things like "_fixed"
- Meaningful names: userID not id
- Early returns to reduce nesting
- Concrete types from constructors: func NewServer() *Server
- Simple errors: return fmt.Errorf("context: %w", err)
- Table-driven tests for complex logic
- Channels for synchronization: signal readiness, not sleep
- Select for timeouts: use select with timeout channels, not sleep loops

### General Development Standards

Tenacity: if an agreed technology approach doesn't work the first attempt, you must persist:
- Try at least 3x to use the chosen approach. None of this "that didn't work let's try a simpler approach". 
- If there is still a problem, alert the user saying "I know the task said to use technology x but I'm persistently encountering issues and would like to discuss alternatives"

Tasks must be as small as practicable:
- Single cohesive unit of work
- Independently testable
- Clear boundaries
- If task seems large, break it down

Every code change requires tests:
- Tests MUST be in the same commit
- No code without tests
- Tests prove the change works
- Update existing tests if behavior changes

Technical documentation is mandatory:
- Every API needs usage examples
- Every interface needs contracts documented
- Every service needs integration guide
- Location: docs/technical/ or inline
- Link from PBI detail document

Constants for ALL repeated values:
```
// BAD: Magic numbers
for (let i = 0; i < 10; i++) { processWebsite(i); }
if (retries > 3) { fail(); }

// GOOD: Named constants
const MAX_WEBSITES = 10;
const MAX_RETRIES = 3;
for (let i = 0; i < MAX_WEBSITES; i++) { processWebsite(i); }
if (retries > MAX_RETRIES) { fail(); }
```

DRY Principle enforcement:
- Task details only in task files, reference from index
- PBI details only in PBI files, reference from backlog
- Exception: titles/names in lists
- Duplication causes inconsistency bugs

## TESTING

### Testing Framework
- Risk-based approach (complex = more tests)
- Follow test pyramid (unit → integration → E2E)
- Test file locations:
  - Unit tests: test/unit/ (mirror source structure)
  - Integration tests: test/integration/ or test/{module}/
- Naming: clear, descriptive test names
- TypeScript / Vite: use vitest

### Integration Test Mocking Strategy
- Mock external third-party services (Firecrawl, Gemini, external APIs)
- Use REAL instances for internal infrastructure:
  - Database (configured for test environment)
  - Message queues (pg-boss, etc.)
  - Redis/cache layers
- This ensures tests validate actual behavior, not mocked assumptions

### Testing Strategy
- Complex business logic → Write tests first
- Simple CRUD → Write tests after
- Hot paths → Add benchmarks
- Skip tests for main() and simple CLI parsing
- Each PBI: dedicated E2E CoS test task

### Code-Change Test Checklist (MANDATORY):
□ Schema changes have migration tests
□ API status codes are tested (200, 400, 404, 500)
□ Error messages match expected format
□ Validation paths covered (valid + invalid inputs)
□ Edge cases handled (null, empty, overflow)
□ Authentication/authorization verified
□ Performance within acceptable bounds
□ Rollback scenarios tested

## PROJECT STRUCTURE

### Go Project Structure
```
cmd/        # Application entrypoints
internal/   # Private code (majority)
pkg/        # Public libraries (if truly reusable)
```

### Documentation Structure
```
docs/
  delivery/
    backlog.md
    {pbi}/
      prd.md
      tasks.md
      {pbi}-{task}.md
  technical/
  research/
  logbook.md
pocs/       # Proof of concepts and experiments
```

### Experiments and POCs
Any experiments must be stored in separate folders inside `pocs/`. Each experiment should have:
- Its own folder with descriptive name
- README.md explaining what's being tested
- Self-contained code that doesn't affect the main project
- Clear results/conclusions documented

### Test Structure
```
test/
  unit/       # Mirrors source structure
  integration/# Or test/{module}/
```

## PROCEDURES

### Task ID Assignment
1. Check existing tasks.md for the PBI
2. Find all used numbers
3. Pick any unused integer
4. NEVER use: 7-1.1, 7-1a, 7-1-sub
5. Task order in list determines priority, not ID

### Starting New Task
1. Check all previous tasks are Done
2. If not, ask User whether to:
   - Complete unfinished tasks first
   - Skip them and proceed
   - Mark as no longer needed
3. Document decision in task history
4. Only proceed after explicit direction

### Task Removal
- NO "Cancelled" or "Removed" status exists
- If task becomes redundant:
  1. Remove entry from tasks.md immediately
  2. Keep .md file for history
  3. Document in related task why removed
  4. Never use invalid status values

### Package Research
1. Search package documentation online
2. Create {task}-{package}-guide.md
3. Include: 
   - Date stamp
   - Link to official docs
   - API usage examples
   - Version information
4. Example: tasks/2-1-pg-boss-guide.md

### Status Updates
1. Update task file status history table
2. Update tasks.md status column
3. Both MUST be updated IN THE SAME COMMIT
4. If mismatch found, fix immediately in one commit
5. Format: | 2025-05-19 15:02:00 | Event Type | From | To | Details | User |

### Current Task Tracking
- File: _current-task-{pbi}-{task} or _current-task-none
- Location: docs/delivery/
- Rules:
  - File MUST be empty (presence alone indicates state)
  - Only ONE _current-task-* file may exist
  - Rename existing file when changing tasks
  - When no tasks InProgress, rename to _current-task-none
  - Add to .gitignore: docs/delivery/_current-task-*
- Workflow:
  - Task→InProgress: rename to _current-task-{task}
  - Task→Done/Blocked: rename to _current-task-none

### Git Workflow
- Branch: task-{id}
- Commit: "{task-id} {description}"
- PR Title: "[{task-id}] {description}"
- PR must link to task
- Merge only after approval

Git ACP Command:
- `git acp "{message}"` is an alias that:
  1. Stages all changes (git add -A)
  2. Creates signed commit (git commit -S -m)
  3. Pushes to current branch (git push)
- Use when marking task as Done
- Example: `git acp "7-1 Add retry logic"`

### Pre-Implementation Checks
- Verify task exists and is Agreed
- Document task ID in all changes
- List all files to be modified
- Get explicit approval before proceeding

### Error Prevention
- If can't access files, stop and report
- For protected files, provide changes for manual application
- Verify task status before ANY work
- Document all status checks

## WORKING MEMORY MANAGEMENT

When context gets long:
- Re-read CLAUDE.md
- Summarize progress in PROGRESS.md or logbook.md
- Document current state before major changes
- If 30+ minutes since reading this file - RE-READ IT

### Project Logbook
- Location: docs/logbook.md
- Update when:
  - After resolving significant technical challenges
  - When making architectural decisions
  - After discovering important API changes or limitations
  - When implementation deviates significantly from original plan
  - After post-mortems of issues or delays
  - When completing any task that involved unexpected challenges or learnings

## PROBLEM-SOLVING

When stuck:
1. Stop - don't spiral into complex solutions
2. Delegate - spawn agents for parallel investigation
3. Ultrathink - "I need to ultrathink through this challenge" for deeper reasoning
4. Step back - re-read requirements
5. Simplify - simple solution usually correct
6. Ask - "I see [A] vs [B]. Which do you prefer?"

## PERFORMANCE & SECURITY

### Measure First
- No premature optimization
- Benchmark before claiming faster
- Use pprof for real bottlenecks

### Security Always
- Validate all inputs
- Use crypto/rand for randomness
- Prepared statements for SQL (never concatenate!)

## COMMUNICATION

### Task Status Updates
```
| 2025-05-19 15:02:00 | Status Update | Agreed | InProgress | Started work | AI |
```

### Progress Updates
```
✓ Implemented authentication (tests passing)
✓ Added rate limiting
✗ Found issue with token expiration - investigating
```

### Honest Progress Reporting
If the task has been completed but has not been tested, the language must be clear that's the case. Only if some capability has had passing tests must it be communicated that "it does xyz".

**Examples:**
- ❌ BAD: "The app now shows file picker when clicking File → Open"
- ✅ GOOD: "I've implemented code that should show file picker when clicking File → Open"
- ❌ BAD: "Drag-and-drop accepts video files"
- ✅ GOOD: "Added drag-and-drop handler that checks for video extensions"
- ✅ BEST: "Tested: File picker appears and filters for video files (manually verified)"

Never claim functionality works unless you have evidence (test results, console output, or explicit user confirmation).

### Suggesting Improvements
"Current approach works, but I notice [observation]. Would you like me to [improvement]?"

## EXAMPLES

### Task IDs
```
✅ GOOD: 7-1, 7-2, 7-13, 7-99
❌ BAD: 7-1.1, 7-1a, 7-1-subtask, 7.1
```

### Status History
```
| 2025-05-19 15:02:00 | Created       | N/A        | Proposed   | Task created      | Julian |
| 2025-05-19 16:15:00 | Status Update | Proposed   | Agreed     | Approved by PO    | Julian |
| 2025-05-19 16:45:00 | Status Update | Agreed     | InProgress | Started work      | AI     |
| 2025-05-20 10:30:00 | Status Update | InProgress | Review     | Ready for review  | AI     |
```

### Commit Messages
```
✅ "7-1 Add database connection retry logic"
✅ "12-3 Fix CSV parsing for special characters"
❌ "Updated stuff"
❌ "7-1.1 Small fix" (bad task ID)
❌ "Added retry logic" (missing task ID)
```

### Task Index Entry
```
| 7-1 | [Add retry logic](./7-1.md) | InProgress | Adds exponential backoff for DB connections |
```

### PBI Entry
```
| 7 | Developer | As a developer, I want resilient connections so that temporary failures don't crash the system | Agreed | System reconnects automatically within 30s |
```

### DRY Example
```markdown
# BAD - Duplicating task details in index:
| 7-1 | [Add retry logic](./7-1.md) | InProgress | This task implements exponential backoff with jitter for database connections, supporting up to 5 retries with configurable delays |

# GOOD - Brief description, details in task file:
| 7-1 | [Add retry logic](./7-1.md) | InProgress | Adds exponential backoff for DB connections |
```

## REQUIRED STRUCTURES

### Backlog (docs/delivery/backlog.md)
```markdown
# Product Backlog
| ID | Actor | User Story | Status | Conditions of Satisfaction (CoS) |
| -- | ----- | ---------- | ------ | -------------------------------- |
| 1  | Dev   | As a...    | Agreed | System does X, Y, Z              |
```
Rules:
- Ordered by priority (highest first) - NOT by ID number
- PBI IDs are unique and immutable (never reused or changed)
- When splitting: original PBI replaced by new PBIs with new IDs
- PBIs written as user stories
- Must describe business value, not technical tasks
- Single source of truth for all PBIs

### PBI History Log (in backlog.md)
After the main table:
```markdown
## History
| Timestamp           | PBI_ID | Event_Type | Details                | User   |
| ------------------- | ------ | ---------- | ---------------------- | ------ |
| 2025-01-20 10:30:00 | 7      | Created    | Initial PBI creation   | Julian |
| 2025-01-20 14:15:00 | 7      | Approved   | Moved to backlog       | Julian |
```
All status transitions MUST be logged with these 5 fields.

### PBI Detail (docs/delivery/{pbi}/prd.md)
```markdown
# PBI-{id}: {title}
## Overview
## Problem Statement
## User Stories
## Technical Approach
## UX/UI Considerations
## Acceptance Criteria
## Dependencies
## Open Questions
## Related Tasks
[View in Backlog](../backlog.md#user-content-{id})
```

### Task File ({pbi}-{task}.md)
```markdown
# {task-id} {task-name}
## Description
## Status History
| Timestamp | Event Type | From Status | To Status | Details | User |
| --------- | ---------- | ----------- | --------- | ------- | ---- |
## Requirements
## Implementation Plan
## Test Plan
### Objectives
### Test Scope  
### Key Scenarios
### Success Criteria
## Verification
## Files Modified
[Back to task list](./tasks.md)
```

### Tasks Index (tasks.md)
```markdown
# Tasks for PBI {id}: {title}
This document lists all tasks associated with PBI {id}.
**Parent PBI**: [PBI {id}: {title}](./prd.md)
## Task Summary
| Task ID | Name | Status | Description |
| :------ | :--- | :----- | :---------- |
| {id}    | [name](./{id}.md) | Status | Brief desc |
```
CRITICAL: This table must contain ONLY these 4 columns. No additional columns or content unless User explicitly approves (parsers depend on this structure).

### Test Plan Requirements
For simple tasks (constants, interfaces, config):
- "TypeScript compilation passes"
- "Integrates with existing system"

For complex tasks (multi-service, business logic):
- Objectives: what tests verify
- Test Scope: components covered
- Environment Setup: test configuration
- Mocking Strategy: what to mock and why
- Key Scenarios: success, failure, edge cases
- Success Criteria: how to determine pass/fail

## VALIDATION RULES

### Before Implementation
- Task exists and status is Agreed
- No sibling tasks InProgress (unless approved)
- Previous tasks completed or explicitly skipped
- Package guides created for ALL dependencies
- Task file has complete implementation plan
- Test plan documented and proportional to complexity

### During Implementation
- All changes reference task ID
- Constants defined for repeated values
- Tests written alongside code
- API documentation created with code
- DRY principle followed

### Task Completion
- All requirements met
- Tests defined in plan are passing
- Tests committed with code changes
- Documentation updated (including technical docs)
- Status synced in both locations
- Logbook updated if applicable
- No code without tests in commit

### File Creation
- Only in defined locations
- Following naming patterns
- With ALL required sections
- Linked appropriately
- No duplication of content

## RATIONALE

### Why One Task at a Time
Prevents context switching, maintains focus, ensures quality, makes progress visible, reduces merge conflicts.

### Why No Hierarchical Task IDs
Simplicity, prevents confusion, easier parsing, cleaner git history, forces proper task breakdown.

### Why Controlled File Creation
Prevents sprawl, maintains organization, ensures discoverability, simplifies cleanup, enables automation.

### Why Research Packages First
Prevents hallucination, documents assumptions, provides reference, speeds development, ensures correct usage.

### Why Test-Driven Changes
Proves code works, prevents regressions, documents behavior, enables refactoring, maintains quality.

### Why DRY Principle
Prevents inconsistency, reduces maintenance, single source of truth, catches errors early.

### Why Technical Documentation
Enables integration, reduces support burden, speeds onboarding, prevents misuse, documents contracts.

### Why Research → Plan → Implement
Prevents wasted effort, ensures understanding, catches issues early, enables better solutions.

### Why Multiple Agents
Parallel processing, specialized expertise, faster exploration, better coverage.

### Why Hook Enforcement
Maintains code quality, prevents technical debt, ensures consistency, catches issues immediately.

## WORKING PRINCIPLES
- Only ONE task per PBI can be InProgress (unless User approves exception)
- Task IDs must be unique integers within PBI
- Status must match in both task file and index (IN SAME COMMIT)
- Research packages online before using them
- Constants for ALL repeated values
- Feature branch - no backwards compatibility needed
- Clarity over cleverness
- Simple, obvious solutions preferred
- This is a partnership - guidance helps stay focused
- Code must be 100% clean - no exceptions

## ENFORCEMENT
- Before EVERY action, check: "Does this violate ANY rule?"
- If unsure, ASK THE USER
- Rules override ANY other consideration
- "I thought..." is NOT an excuse
- "It seemed like..." is NOT an excuse
- READ THE RULE LITERALLY - no interpretation