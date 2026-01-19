# SafeHaven Slash Commands - Quick Reference

> Custom commands configured in `.claude/config.json` for workflow automation

---

## 🚀 Getting Started

### `/onboard` - Quick Onboarding (Start Here!)
**Use this at the beginning of every new Claude Code session**

Reads essential files and provides context:
1. `CHANGELOG.md` - Current status and next tasks
2. `PLAN.md` - Implementation strategy
3. `CLAUDE.md` - Coding standards

Then summarizes:
- Current phase and milestone
- What was last completed
- What's next
- Any blockers

**Example:**
```
/onboard
```

---

## 📊 Status & Context Commands

### `/status` - Show Next Tasks
Shows only the "NEXT UP" section from CHANGELOG.md

**Example:**
```
/status
```

### `/full-context` - Complete Documentation Load
Reads ALL documentation files (use when you need complete context):
- CHANGELOG.md
- PLAN.md
- PROJECT-SPEC.md
- CLAUDE.md
- .clinerules

**Example:**
```
/full-context
```

### `/standards` - Show Coding Standards
Reads CLAUDE.md and .clinerules - key patterns and safety rules

**Example:**
```
/standards
```

### `/plan` - Show Milestone Details
Shows specific milestone from PLAN.md

**Example:**
```
/plan 1          # Show Milestone 1: Foundation & Auth
/plan 6          # Show Milestone 6: iOS DeviceActivity Module
/plan            # Show current milestone
```

### `/next` - Auto-Continue
Looks at CHANGELOG "NEXT UP", reads relevant context, and starts working on highest priority task

**Example:**
```
/next
```

---

## 📝 Documentation Commands

### `/update-changelog` - Update Progress
Updates CHANGELOG.md after completing work:
- Moves completed items from "NEXT UP" to new entry
- Asks what was accomplished, decisions made, files created
- Updates "NEXT UP" with upcoming tasks

**Example:**
```
/update-changelog
```

---

## 🔧 Development Commands

### `/test-ios` - Start Development Server
Starts Expo development server for iOS

**Example:**
```
/test-ios
```

### `/check-quality` - Run Quality Checks
Runs all quality checks:
1. TypeScript type check (`npx tsc --noEmit`)
2. ESLint (`npm run lint`)
3. Check for console.logs in src/

**Example:**
```
/check-quality
```

### `/db-schema` - Show Database Schema
Reads PROJECT-SPEC.md and displays complete database schema with all tables, fields, and relationships

**Example:**
```
/db-schema
```

### `/safety-check` - Review Safety Rules
Reviews code changes against safety-critical rules from .clinerules:
- Error handling for all async operations
- Vacation mode checks in escalation logic
- User-friendly error messages
- Device activity validation

**Example:**
```
/safety-check
```

---

## 🏗️ Code Generation Commands

### `/new-service` - Create New Service
Creates service + repository with proper structure:
- TypeScript interfaces
- Try-catch error handling
- Structured response types
- JSDoc comments

**Example:**
```
/new-service Monitoring     # Creates MonitoringService.ts and MonitoringRepository.ts
```

### `/new-screen` - Create New Screen
Creates screen component with:
- TypeScript props interface
- StyleSheet styles (no inline)
- Error handling and loading states
- Navigation setup

**Example:**
```
/new-screen PersonDetail    # Creates PersonDetailScreen.tsx
```

### `/new-component` - Create New Component
Creates component with:
- TypeScript props interface
- StyleSheet styles
- 60pt minimum touch targets
- High contrast colors
- JSDoc comments

**Example:**
```
/new-component SafetyStatusBadge    # Creates SafetyStatusBadge.tsx
```

---

## 🎯 Git Commands

### `/commit-docs` - Commit Documentation
Stages and commits all documentation files with "docs:" prefix

**Example:**
```
/commit-docs
```

### `/commit-feature` - Commit Feature
Commits with format: `feat(scope): description`

**Example:**
```
/commit-feature auth add role selection to sign up
# Result: feat(auth): add role selection to sign up
```

### `/commit-fix` - Commit Bug Fix
Commits with format: `fix(scope): description`

**Example:**
```
/commit-fix escalation prevent false alarms during vacation mode
# Result: fix(escalation): prevent false alarms during vacation mode
```

---

## 💡 Workflow Examples

### Starting a New Session
```
/onboard                    # Get up to speed
/status                     # See what's next
/next                       # Auto-continue from last session
```

### Working on a Feature
```
/plan 3                     # Read Milestone 3 details
/standards                  # Review coding standards
# ... implement feature ...
/check-quality              # Run quality checks
/safety-check               # Verify safety rules
/commit-feature monitoring add person detail screen
/update-changelog           # Update progress
```

### Debugging an Issue
```
/full-context               # Get complete context
/db-schema                  # Review database schema
/safety-check               # Check safety rules
# ... fix issue ...
/commit-fix escalation handle race condition
```

### Creating New Code
```
/new-service CheckIn        # Create service layer
/new-screen MonitoredUser   # Create UI screen
/new-component CheckInButton # Create component
/check-quality              # Verify code quality
```

---

## 📌 Best Practices

**Always Start With:**
```
/onboard
```

**Before Implementing:**
```
/standards      # Review patterns
/plan X         # Read milestone details
```

**After Completing Work:**
```
/check-quality
/safety-check
/commit-feature (or /commit-fix)
/update-changelog
```

**End of Session:**
```
/status         # Verify "NEXT UP" is current
/commit-docs    # Commit any doc updates
```

---

## 🎓 Tips

1. **Use `/onboard` at start of EVERY session** - It's fast and ensures Claude has context
2. **Use `/next` for continuity** - Auto-continues from where you left off
3. **Use `/safety-check` before commits** - Catches safety-critical issues early
4. **Use `/update-changelog` frequently** - Keep progress tracking current
5. **Use `/milestone X` when switching tasks** - Get full context for that milestone

---

## 🔧 Customization

All commands are defined in `.claude/config.json`

To add or modify commands:
1. Edit `.claude/config.json`
2. Add new command with format: `"/command-name": "instruction"`
3. Restart Claude Code session
4. Update this file with new command documentation

---

*Last Updated: 2026-01-19*
*17 commands configured*
