# 🎯 CRITICAL: READ THIS FIRST

## Your Environment Structure

```
project-root/
├── .claude/
│   ├── settings.json          ← PERMISSIONS & HOOKS (enforced)
│   ├── CLAUDE.md              ← YOUR INSTRUCTIONS (read carefully)
│   ├── ONBOARDING.md          ← THIS FILE
│   ├── plans/                 ← Project plans (immutable once locked)
│   └── hooks/                 ← NOT PRESENT (hooks are in node_modules)
│
├── node_modules/
│   └── .gl-life-claude/
│       ├── scripts/           ← 17 WORKFLOW SCRIPTS (bundled, protected)
│       └── hooks/             ← 28 ENFORCEMENT HOOKS (bundled, protected)
│
├── .git/hooks/                ← GIT HOOKS (installed by setup:git-hooks)
│
├── Agent/                     ← YOUR WORKSPACE (you can modify)
│   ├── src/                   ← Source code
│   ├── tests/                 ← Test files
│   └── db/                    ← Database scripts (write only, human executes)
│
├── scripts/                   ← CUSTOM SCRIPTS (you can modify)
│
└── package.json               ← NPM scripts configured
```

## 🚨 HARD RULES - NEVER VIOLATED

### Rule 1: NO WORK WITHOUT ACTIVE PLAN + TASK
**BLOCKED BY:** `enforce-structured-development.js` hook (PreToolUse)

```bash
# ALWAYS start with a plan:
npm run plan:generate         # AI-assisted (recommended)
# OR
npm run plan:create           # Manual

# Then lock it:
npm run plan:init

# Then start a task:
npm run task:start TASK-001
```

**What gets blocked if no active task:**
- ❌ Edit()
- ❌ Write()
- ❌ TodoWrite()
- ❌ Read() on project files
- ❌ Grep() on project files
- ❌ Glob() on project files

**Only allowed without task:**
- ✅ npm run commands
- ✅ git status/log/diff
- ✅ Reading documentation

### Rule 2: WORK ONLY ON MAIN BRANCH
**BLOCKED BY:** Permission system in `settings.json`

```bash
# ✅ ALLOWED:
git checkout main
git add .
git commit -m "[TASK-001] Description"
git push origin main

# ❌ BLOCKED:
git checkout -b feature/anything    # DENIED
git checkout -b task/anything       # DENIED
git merge                           # DENIED
```

**All work happens directly on main branch. No branching.**

### Rule 3: RESTRICTED DIRECTORY ACCESS
**BLOCKED BY:** Permission system in `settings.json`

```bash
# ✅ YOU CAN MODIFY:
./Agent/**           # Your source code
./scripts/**         # Your custom scripts

# ❌ YOU CANNOT MODIFY:
./docs/**            # Documentation (DENIED)
./.claude/**         # Framework files (DENIED)
./node_modules/**    # Dependencies (DENIED)
**/*.md              # Markdown files (DENIED)
./.env*              # Environment files (DENIED)
```

### Rule 4: NO DANGEROUS OPERATIONS
**BLOCKED BY:** Permission system in `settings.json`

```bash
# ❌ BLOCKED COMMANDS:
git push --force
rm -rf
sudo
npm install           # Ask user first
npm publish
curl / wget
psql / mysql / mongosh
node ./scripts/**     # Write scripts, human executes
node ./Agent/db/**    # Write scripts, human executes
```

## 📋 Available Commands

### Planning Commands (ALWAYS START HERE)
```bash
npm run plan:generate    # AI-assisted plan generation
npm run plan:create      # Manual plan creation
npm run plan:init        # Lock plan, create task tracker
npm run plan:manager     # Switch between plans
npm run plan:amend       # Amend locked plan (audit trail)
npm run plan:help        # Planning help
```

### Task Commands (USE AFTER PLAN IS LOCKED)
```bash
npm run task:start TASK-001  # Start specific task
npm run task:next            # Auto-start next available task
npm run task:done TASK-001   # Complete current task
npm run task:status          # View project progress
```

### Utility Commands
```bash
npm run help                 # Show all commands
npm run setup:git-hooks      # Install git hooks
```

## 🔒 How Hooks Protect the Workflow

### PreToolUse Hooks (RUN BEFORE EVERY ACTION)
1. **enforce-hard-requirements.js** - Validates critical constraints
2. **prevent-read-bypass.js** - Prevents unauthorized file access
3. **enforce-structured-development.js** - Requires active task
4. **validate-git-workflow.js** - Validates git operations
5. **validate-task-completion.js** - Ensures task is active

### PostToolUse Hooks (RUN AFTER EVERY ACTION)
1. **auto-format.js** - Auto-formats code
2. **validate-test-quality.js** - Blocks fake tests
3. **validate-ui-integration.js** - Requires UI tests
4. **validate-database-changes.js** - Validates migrations
5. **enforce-test-pyramid.js** - Warns on missing tests
6. And 23 more hooks...

**ALL HOOKS USE `process.exit(1)` OR `process.exit(2)` TO HARD BLOCK VIOLATIONS**

## ✅ MANDATORY: Task Completion Steps

**EVERY SINGLE TASK MUST FOLLOW THESE STEPS - NO EXCEPTIONS**

### Step-by-Step Task Completion Checklist

```
□ 1. Read task file: .claude/plans/<plan-id>/tasks/TASK-XXX.json
     - Read description, completionCriteria, testCases
     - Understand what needs to be done

□ 2. Read related files mentioned in task
     - Check artifacts.code for files to modify
     - Read existing code before changing

□ 3. Make changes in ./Agent/ or ./scripts/ only
     - Edit/Write only in allowed directories
     - Follow task requirements exactly

□ 4. Write/update tests
     - Create test file if doesn't exist
     - Add test cases from task.testCases
     - Ensure tests have real assertions

□ 5. Run tests and verify they pass
     - npm test (or project-specific test command)
     - Fix any failures before continuing

□ 6. Commit changes with task ID
     - git add .
     - git commit -m "[TASK-XXX] Description"
     - Include task ID in EVERY commit

□ 7. Push to remote
     - git push origin main
     - Backup work regularly

□ 8. Verify all completion criteria met
     - Check task.completionCriteria array
     - Ensure EVERY criterion is satisfied

□ 9. Mark task complete
     - npm run task:done TASK-XXX
     - System validates all changes committed
```

**If ANY step fails, DO NOT proceed. Fix the issue first.**

## 🧪 TEST-DRIVEN DEVELOPMENT - HARD RULES

**ALL TESTING RULES ARE ENFORCED BY HOOKS - VIOLATIONS WILL BLOCK COMMITS**

### Rule 1: Tests MUST Have Real Assertions
**ENFORCED BY:** `validate-test-quality.js` (PostToolUse)

```javascript
// ❌ BLOCKED - Fake test
test('should work', () => {
  // empty test
});

// ❌ BLOCKED - No assertions
test('should calculate', () => {
  const result = add(2, 3);
  // no assertion
});

// ✅ ALLOWED - Real assertion
test('should add numbers', () => {
  const result = add(2, 3);
  expect(result).toBe(5);
});
```

**Hook blocks:**
- Empty test functions
- Tests without expect/assert/should
- Comment-only tests
- Console.log-only tests

### Rule 2: UI Components MUST Have Tests
**ENFORCED BY:** `validate-ui-integration.js` (PostToolUse)

When you modify UI components (`.tsx`, `.jsx`, `.vue`, `.svelte`):

```bash
# ❌ BLOCKED - No test file
src/components/Button.tsx           # modified
# Missing: src/components/Button.test.tsx

# ✅ ALLOWED - Test file exists
src/components/Button.tsx           # modified
src/components/Button.test.tsx      # exists with real tests
```

**Required in UI tests:**
- Import and render the component
- Provide required props
- Simulate user events (clicks, inputs)
- Mock API calls if used
- Validate state changes
- Test form submission if applicable
- Test conditional rendering paths

### Rule 3: UI Components MUST Be in Integration Site
**ENFORCED BY:** `validate-integration-site.js` (PostToolUse)

```bash
# ❌ BLOCKED - Not in test-site
src/components/NewWidget.tsx        # created
# Missing: test-site/src/components/NewWidget.tsx

# ✅ ALLOWED - Added to integration site
src/components/NewWidget.tsx        # created
test-site/src/components/NewWidget.tsx  # added for stakeholder preview
```

**Purpose:** Integration site allows stakeholders to preview components with hot-reload.

### Rule 4: Tests MUST Pass Before Commit
**ENFORCED BY:** `validate-test-results.js` (PostToolUse)

```bash
# ❌ BLOCKED - Tests failing
npm test
# 5 passing, 2 failing
git commit -m "Add feature"
# BLOCKED: Tests must pass

# ✅ ALLOWED - All tests pass
npm test
# 7 passing
git commit -m "Add feature"
# Commit succeeds
```

**Hook checks npm test exit code. Non-zero = blocked.**

### Rule 5: Database Changes MUST Use Migrations
**ENFORCED BY:** `validate-database-changes.js` (PostToolUse)

```bash
# ❌ BLOCKED - Direct schema edit
Edit(./Agent/db/schema.sql)         # DENIED by permissions

# ✅ ALLOWED - Migration workflow
Write(./Agent/migrations/001_add_users_table.sql)
# Migration file created, human executes
```

**Rules:**
- NEVER edit existing migrations (immutable)
- ALWAYS create new migration files
- NEVER run database CLI directly (psql, mysql)
- Human executes migrations, Claude writes them

### Rule 6: Test Pyramid Must Be Balanced
**ENFORCED BY:** `enforce-test-pyramid.js` (PostToolUse)

```bash
# ⚠️  WARNING - Imbalanced tests
Unit tests: 50
Integration tests: 5
E2E tests: 0
# Warning: Need more integration and E2E tests

# ✅ GOOD - Balanced pyramid
Unit tests: 100
Integration tests: 30
E2E tests: 10
```

**Recommended ratio:** 70% unit, 20% integration, 10% e2e

## 📝 TASK DEVELOPMENT WORKFLOW (STEP-BY-STEP)

**Follow this EXACT workflow for EVERY task - hooks enforce it**

### Phase 1: Planning (Before Code)
```bash
□ 1. Read task file
     cat .claude/plans/<plan-id>/tasks/TASK-XXX.json

□ 2. Understand requirements
     - Read description
     - Check completionCriteria (what defines "done")
     - Check testCases (what tests are required)
     - Check artifacts.code (which files to modify)

□ 3. Read existing code
     Read(./Agent/src/target-file.ts)
     # NEVER modify code you haven't read first
```

### Phase 2: Write Tests First (TDD)
```bash
□ 4. Create test file
     Write(./Agent/tests/feature.test.ts)

□ 5. Write test cases from task
     - Use task.testCases as test names
     - Write failing tests with real assertions
     - Test expected behavior, edge cases, errors

□ 6. Run tests (should fail)
     npm test
     # Tests should fail - no implementation yet
```

### Phase 3: Implement Code
```bash
□ 7. Write minimum code to pass tests
     Edit(./Agent/src/feature.ts)
     # Only in ./Agent/** or ./scripts/**

□ 8. Run tests (should pass)
     npm test
     # Fix until all tests pass

□ 9. Refactor if needed
     # Improve code while tests stay green
```

### Phase 4: UI Components (If Applicable)
```bash
□ 10. Create UI test
      Write(./Agent/src/components/Widget.test.tsx)
      - Import and render component
      - Test props, events, state, API mocks

□ 11. Implement UI component
      Edit(./Agent/src/components/Widget.tsx)

□ 12. Add to integration site
      Write(./test-site/src/components/Widget.tsx)
      # For stakeholder preview

□ 13. Run UI tests
      npm test
      # Hooks validate test quality
```

### Phase 5: Commit & Push
```bash
□ 14. Verify completion criteria
      # Check task.completionCriteria - ALL must be met

□ 15. Run full test suite
      npm test
      # MUST pass - hooks will block otherwise

□ 16. Commit with task ID
      git add .
      git commit -m "[TASK-XXX] Implement feature with tests"
      # Hooks validate:
      # - Active task exists
      # - Tests pass
      # - Tests have assertions
      # - UI components have tests
      # - No fake/empty tests

□ 17. Push to remote
      git push origin main
      # Backup work regularly
```

### Phase 6: Complete Task
```bash
□ 18. Mark complete
      npm run task:done TASK-XXX
      # System validates:
      # - All changes committed
      # - No uncommitted files
      # - Task not already completed
```

## ⚠️ Common Test Violations (WILL BE BLOCKED)

### Violation 1: Empty Tests
```javascript
// ❌ BLOCKED by validate-test-quality.js
test('should work', () => {});
test('should do something', () => {
  // TODO: implement
});
```

### Violation 2: No Assertions
```javascript
// ❌ BLOCKED by validate-test-quality.js
test('calculates total', () => {
  const result = calculateTotal(items);
  console.log(result); // No assertion!
});
```

### Violation 3: UI Without Tests
```javascript
// ❌ BLOCKED by validate-ui-integration.js
// Created: src/components/Button.tsx
// Missing: src/components/Button.test.tsx
```

### Violation 4: Failing Tests
```bash
# ❌ BLOCKED by validate-test-results.js
$ npm test
FAIL src/feature.test.ts
  ✓ works in basic case
  ✗ handles edge case

$ git commit -m "Add feature"
# BLOCKED: Tests must pass
```

### Violation 5: Modifying Code Without Task
```bash
# ❌ BLOCKED by enforce-structured-development.js
$ Edit(./Agent/src/app.ts)
# ERROR: No active task
# Start task first: npm run task:start TASK-XXX
```

## ✅ Correct Test-Driven Workflow Example

```bash
# 1. Start task
npm run task:start TASK-005

# 2. Read task requirements
cat .claude/plans/plan-001/tasks/TASK-005.json

# 3. Read existing code
Read(./Agent/src/calculator.ts)

# 4. Write failing test
Write(./Agent/tests/calculator.test.ts)
"""
test('should multiply numbers', () => {
  const result = multiply(3, 4);
  expect(result).toBe(12);
});
"""

# 5. Run test (should fail)
npm test
# FAIL: multiply is not defined

# 6. Implement feature
Edit(./Agent/src/calculator.ts)
"""
export function multiply(a, b) {
  return a * b;
}
"""

# 7. Run test (should pass)
npm test
# PASS: 1 test passed

# 8. Commit
git add .
git commit -m "[TASK-005] Add multiply function with tests"
# Hooks validate:
# ✓ Active task exists
# ✓ Tests pass
# ✓ Tests have real assertions
# ✓ Code only in allowed directories

# 9. Push
git push origin main

# 10. Complete task
npm run task:done TASK-005
```

**If ANY step fails, DO NOT proceed. Fix the issue first.**

## 🎯 Correct Workflow (ALWAYS FOLLOW THIS)

### Step 1: Generate Plan
```bash
npm run plan:generate "Build a REST API with:
- User authentication
- CRUD operations for tasks
- PostgreSQL database
- Express + TypeScript"
```

This creates `.claude/plans/plan-001-generated/PROJECT-REQUIREMENTS.txt`

### Step 2: Ask Claude to Create Plan
In Claude Code chat:
```
"Please read .claude/plans/plan-001-generated/PROJECT-REQUIREMENTS.txt
and create a detailed PROJECT-PLAN.json with tasks, dependencies, and phases"
```

Claude will create structured plan with TASK-001, TASK-002, etc.

### Step 3: Lock the Plan
```bash
npm run plan:init
```

This creates:
- `TASK-TRACKER.json` (tracks progress)
- `tasks/TASK-001.json`, `tasks/TASK-002.json`, etc.
- `.plan-locked` (makes plan immutable)

### Step 4: Start Working
```bash
npm run task:start TASK-001
```

Now you can:
- Edit files in `./Agent/`
- Write tests
- Commit changes: `git commit -m "[TASK-001] Implemented feature"`
- Push: `git push origin main`

### Step 5: Complete Task
```bash
npm run task:done TASK-001
```

### Step 6: Continue
```bash
npm run task:next    # Automatically starts TASK-002
```

## ⚠️ Common Mistakes (DON'T DO THIS)

### ❌ WRONG: Working without a plan
```bash
# This will FAIL - no active task:
Edit(./Agent/src/app.ts)
# ❌ BLOCKED: No active task
```

### ✅ CORRECT: Start with plan
```bash
npm run plan:generate "My project requirements..."
# Claude creates plan
npm run plan:init
npm run task:start TASK-001
# Now Edit() works
```

### ❌ WRONG: Creating branches
```bash
git checkout -b feature/my-feature
# ❌ BLOCKED: Permission denied
```

### ✅ CORRECT: Work on main
```bash
git checkout main
# Work here, commit, push
```

### ❌ WRONG: Editing framework files
```bash
Edit(./.claude/settings.json)
# ❌ BLOCKED: Permission denied
```

### ✅ CORRECT: Only edit your code
```bash
Edit(./Agent/src/myfile.ts)    # ✅ Allowed
Edit(./scripts/myutil.js)      # ✅ Allowed
```

### ❌ WRONG: Running scripts directly
```bash
Bash(node ./scripts/setup.js)
# ❌ BLOCKED: Permission denied
```

### ✅ CORRECT: Write scripts, human executes
```bash
Write(./scripts/setup.js)      # ✅ Allowed
# Tell user: "Run: node ./scripts/setup.js"
```

## 🔍 Debugging Blocked Actions

If you get blocked, check:

1. **Is there an active plan?**
   ```bash
   cat .claude/ACTIVE-PLAN
   ```

2. **Is the plan locked?**
   ```bash
   ls .claude/plans/plan-*/. plan-locked
   ```

3. **Is there an active task?**
   ```bash
   npm run task:status
   ```

4. **Are hooks installed?**
   ```bash
   ls .git/hooks/pre-commit
   ```

5. **What are my permissions?**
   ```bash
   cat .claude/settings.json
   ```

## 📖 Where to Find More Info

- **Workflow rules:** `.claude/CLAUDE.md`
- **Permissions:** `.claude/settings.json`
- **Active plan:** `.claude/ACTIVE-PLAN`
- **Task progress:** `.claude/plans/<plan-id>/TASK-TRACKER.json`
- **Help commands:** `npm run help`

## 🚀 Quick Start for Claude

```bash
# 1. Check if plan exists
npm run task:status

# 2a. If no plan, create one:
npm run plan:generate

# 2b. If plan exists but not locked:
npm run plan:init

# 3. Start working:
npm run task:next

# 4. Work on files in ./Agent/ directory
# 5. Commit frequently with task ID
# 6. Complete task when done:
npm run task:done TASK-XXX
```

## 🎓 Remember

1. **NEVER** work without an active task
2. **NEVER** create branches (work on main)
3. **NEVER** modify framework files (`.claude/`, `node_modules/`)
4. **NEVER** run dangerous commands (force push, rm -rf, sudo)
5. **ALWAYS** use npm run commands for workflow
6. **ALWAYS** commit with task ID: `[TASK-XXX] Description`
7. **ALWAYS** check `npm run task:status` if blocked

---

**This framework enforces professional development practices. The hooks and permissions exist to prevent mistakes, not to annoy you. Follow the workflow and you'll be productive.**
