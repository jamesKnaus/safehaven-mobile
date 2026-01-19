# Project: SafeHaven Mobile

> **Part of the Claude Code Setup Kit** - Project guidelines for Claude Code
> For full requirements, see `PROJECT-SPEC.md`. For implementation plan, see `PLAN.md`

## Overview

**Project Description:** SafeHaven is a personal safety monitoring app for elderly users living alone. The app provides peace of mind to family members by monitoring device activity and daily check-ins. It features a two-user architecture: an ultra-simple check-in interface for elderly monitored persons, and a comprehensive dashboard for adult children/caregivers who monitor their safety.

**Project Goal:** Build production-ready MVP for iOS (lives depend on this working correctly)

**Current Milestone:** MVP - Core safety check-in and monitoring system with device activity tracking

**Last Updated:** 2026-01-19

---

## Tech Stack

### Mobile Frontend
- **Framework:** React Native + Expo SDK 50+
- **Language:** TypeScript (strict mode enabled)
- **Navigation:** React Navigation 6.x
  - Stack Navigator for auth flow
  - Tab Navigator for main app (monitor role only)
  - Role-based routing (monitored vs monitor)
- **State Management:** React Context API + Hooks
  - AuthContext for user session
  - CheckInContext for check-in state
  - No Redux - keep it simple for MVP
- **Styling:** React Native StyleSheet API + theme constants
  - Colors defined in `/src/config/theme.ts`
  - NO inline styles
  - Platform-specific styles handled via `Platform.select()`
- **UI Components:** Custom components only (no component library for MVP)
  - Large touch targets for elderly users (minimum 60x60pt)
  - High contrast colors for accessibility

### Backend
- **BaaS:** Supabase (existing project from iOS version)
- **Database:** PostgreSQL with Row-Level Security (RLS)
- **Authentication:** Supabase Auth
  - Email/password
  - Sign in with Apple (iOS only)
- **Real-time:** Supabase Realtime (for monitoring dashboard updates)
- **Edge Functions:** Supabase Edge Functions (for push notifications via Expo Push API)
- **Storage:** Supabase Storage (future - for contact photos)

### Key Services
- **Notifications:** expo-notifications
  - Daily check-in reminders
  - Critical safety alerts for monitors
  - Background notification scheduling
- **Background Tasks:** expo-background-fetch + expo-task-manager
  - Periodic device activity checks
  - Escalation detection (24-hour inactivity)
- **Device Activity Monitoring:** Custom iOS Native Module
  - Uses iOS DeviceActivity framework
  - Tracks device usage as PRIMARY safety signal
  - Requires physical iOS device for testing (doesn't work in simulator)
- **Date/Time:** date-fns, @react-native-picker/datetimepicker

### Infrastructure
- **Version Control:** GitHub (https://github.com/jamesKnaus/safehaven-mobile)
- **CI/CD:** GitHub Actions (future)
- **Build System:** EAS Build (Expo Application Services)
- **App Distribution:** TestFlight (iOS) initially, then App Store
- **Monitoring:** Sentry (future - post-MVP)

### Development Tools
- **Testing:** Jest + React Native Testing Library (future)
- **Linting:** ESLint + Prettier
- **Type Checking:** TypeScript strict mode
- **Git Workflow:** Feature branches → PR → main
- **Device Testing:** Physical iOS devices required (DeviceActivity framework)

---

## Project Structure

```
/safehaven-mobile
  /src
    /components       # Reusable UI components
      /common         # Button, Card, Input, LoadingSpinner, etc.
      /checkin        # CheckInButton, CheckInStatus, StreakBadge
      /monitoring     # MonitoredPersonCard, SafetyStatusBadge
      /auth           # LoginForm, SignUpForm
    /screens          # Screen components (one per route)
      /auth           # LoginScreen, SignUpScreen, OnboardingScreen
      /monitored      # MonitoredUserScreen (simple check-in screen)
      /monitor        # MonitorDashboardScreen, PersonDetailScreen, AddPersonScreen
      /settings       # SettingsScreen
    /navigation       # Navigation setup
      RootNavigator.tsx       # Main navigator with role-based routing
      AuthNavigator.tsx       # Auth flow (login, signup)
      MonitoredNavigator.tsx  # Just check-in screen (no tabs)
      MonitorTabNavigator.tsx # Dashboard + Settings tabs
    /services         # Business logic layer
      AuthService.ts          # Authentication logic
      CheckInService.ts       # Check-in creation and validation
      MonitoringService.ts    # Monitoring relationships CRUD
      NotificationService.ts  # Push notification handling
      EscalationService.ts    # Escalation detection and triggering
      DeviceActivityService.ts # Native module wrapper (iOS)
      BackgroundTaskService.ts # Background fetch setup
    /repositories     # Data access layer (Supabase queries)
      UserRepository.ts
      CheckInRepository.ts
      MonitoringRepository.ts
      EscalationRepository.ts
    /contexts         # React Context providers
      AuthContext.tsx
      CheckInContext.tsx
    /hooks            # Custom React hooks
      useAuth.ts
      useCheckIn.ts
      useMonitoring.ts
      useNotifications.ts
    /types            # TypeScript type definitions
      models.ts       # User, CheckIn, MonitoringRelationship, etc.
      navigation.ts   # Navigation param types
      api.ts          # API response types
    /utils            # Helper functions
      constants.ts    # App constants (colors, sizes, etc.)
      dateHelpers.ts  # Date formatting and validation
      validators.ts   # Input validation
    /config           # Configuration
      supabase.ts     # Supabase client setup
      theme.ts        # Colors, fonts, spacing
    /native           # iOS native modules
      DeviceActivityModule/ # iOS DeviceActivity framework wrapper
  /assets             # Images, fonts, splash screens
  /ios                # iOS native code (after ejecting if needed)
  App.tsx             # Root component
  app.json            # Expo configuration
  .env                # Environment variables (NEVER commit)
  .env.example        # Template for environment variables
  tsconfig.json       # TypeScript configuration
  PROJECT-SPEC.md     # Product and engineering requirements
  PLAN.md             # Implementation plan
  CLAUDE.md           # This file
  .clinerules         # Code quality rules
```

---

## Coding Standards

### General Principles
- **Safety First** - This is a life-safety application. Error handling is critical.
- **Simple Over Clever** - Elderly users need reliability, not complexity
- **Test Critical Paths** - Device activity detection and escalation MUST work correctly
- **Explicit Over Implicit** - Clear, readable code over "clever" solutions
- **Separation of Concerns** - UI Layer → Service Layer → Data Layer

### TypeScript Guidelines
- ✅ **Always use explicit types** - Never rely on inference for function signatures
- ✅ **Use interfaces for objects** - `interface User {}`, not `type User = {}`
- ✅ **Enable strict mode** - All TypeScript strict flags enabled
- ❌ **Avoid `any` type** - Use `unknown` if type is truly unknown
- ✅ **Use const assertions** - For literal types: `as const`
- ✅ **Generic types for reusability** - Especially for repository methods

```typescript
// ✅ Good - Explicit types
interface User {
  id: string;
  name: string;
  email: string;
  role: 'monitored' | 'monitor';
  lastCheckIn: Date | null;
  lastDeviceActivity: Date | null;
}

async function fetchUser(id: string): Promise<User | null> {
  // implementation
}

// ❌ Bad - Implicit any
function fetchUser(id) {
  // implementation
}
```

### React Native Guidelines
- ✅ **Functional components only** - No class components
- ✅ **Use hooks** - useState, useEffect, useContext, custom hooks
- ✅ **Props destructuring** - Destructure props in function signature
- ✅ **Named exports** - Use named exports, not default exports
- ✅ **StyleSheet.create** - Define all styles with StyleSheet.create
- ❌ **No inline styles** - Always use StyleSheet
- ✅ **Platform-specific code** - Use Platform.select() or Platform.OS
- ✅ **Large touch targets** - Minimum 60x60pt for elderly users
- ✅ **Handle keyboard** - Use KeyboardAvoidingView, dismissKeyboard

```typescript
// ✅ Good - React Native component
interface CheckInButtonProps {
  onCheckIn: () => Promise<void>;
  isCheckedIn: boolean;
  isLoading: boolean;
}

export function CheckInButton({ onCheckIn, isCheckedIn, isLoading }: CheckInButtonProps) {
  return (
    <TouchableOpacity
      style={styles.button}
      onPress={onCheckIn}
      disabled={isCheckedIn || isLoading}
      activeOpacity={0.7}
    >
      <Text style={styles.buttonText}>
        {isCheckedIn ? '✓ Checked In Today' : 'Tap to Check In'}
      </Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  button: {
    width: 200,
    height: 200,
    backgroundColor: colors.coral,
    borderRadius: 16,
    justifyContent: 'center',
    alignItems: 'center',
  },
  buttonText: {
    fontSize: 24,
    fontWeight: 'bold',
    color: colors.white,
  },
});

// ❌ Bad - Inline styles, default export
export default function CheckInButton(props: any) {
  return (
    <View style={{ padding: 20 }}>
      <Text>{props.text}</Text>
    </View>
  );
}
```

### Service Layer Guidelines
- ✅ **Three-layer architecture** - UI → Service → Repository
- ✅ **Services contain business logic** - Not UI, not database queries
- ✅ **Repositories contain database access** - Supabase queries only
- ✅ **Async/await over promises** - Use async/await, not .then()
- ✅ **Try-catch for ALL async** - Every async operation wrapped in try-catch
- ✅ **Return structured responses** - `{ success: boolean; data?: T; error?: string }`

```typescript
// ✅ Good - Service layer with error handling
export class CheckInService {
  constructor(
    private checkInRepo: CheckInRepository,
    private userRepo: UserRepository
  ) {}

  async createCheckIn(userId: string): Promise<ServiceResponse<CheckIn>> {
    try {
      // Validation
      const user = await this.userRepo.findById(userId);
      if (!user) {
        return { success: false, error: 'User not found' };
      }

      // Check if already checked in today
      const hasCheckedInToday = await this.checkInRepo.hasCheckedInToday(userId);
      if (hasCheckedInToday) {
        return { success: false, error: 'Already checked in today' };
      }

      // Business logic
      const checkIn = await this.checkInRepo.create({
        userId,
        timestamp: new Date(),
        method: 'manual',
      });

      // Update user's last check-in
      await this.userRepo.updateLastCheckIn(userId, checkIn.timestamp);

      return { success: true, data: checkIn };
    } catch (error) {
      console.error('CheckInService.createCheckIn error:', error);
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Unknown error'
      };
    }
  }
}
```

### Database/Repository Guidelines
- ✅ **Row-Level Security (RLS)** - All tables use RLS policies
- ✅ **Validate RLS in tests** - Ensure users can't access others' data
- ✅ **Handle null values** - Database fields can be null, handle gracefully
- ✅ **Use transactions** - For multi-step operations (escalation, etc.)
- ✅ **Specific error messages** - Don't just return generic "database error"

```typescript
// ✅ Good - Repository with RLS consideration
export class MonitoringRepository {
  constructor(private supabase: SupabaseClient) {}

  async getMonitoredPeople(monitorId: string): Promise<User[]> {
    const { data, error } = await this.supabase
      .from('monitoring_relationships')
      .select(`
        monitored:monitored_id (
          id,
          name,
          email,
          last_check_in,
          last_device_activity,
          escalation_status,
          vacation_mode
        )
      `)
      .eq('monitor_id', monitorId)
      .eq('active', true);

    if (error) {
      throw new Error(`Failed to fetch monitored people: ${error.message}`);
    }

    return data?.map(row => row.monitored) || [];
  }
}
```

### Naming Conventions
- **Files:** kebab-case (`check-in-service.ts`, `monitored-user-screen.tsx`)
- **Components:** PascalCase (`CheckInButton`, `MonitoredPersonCard`)
- **Functions:** camelCase (`createCheckIn`, `hasCheckedInToday`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_CONTACTS`, `ESCALATION_THRESHOLD_HOURS`)
- **Types/Interfaces:** PascalCase (`User`, `CheckIn`, `ServiceResponse`)
- **React Native styles:** camelCase (`buttonPrimary`, `textLarge`)

### Error Handling Guidelines (CRITICAL)
- ✅ **Always handle errors** - Never ignore catch blocks
- ✅ **User-friendly messages** - Show clear, actionable error messages
- ✅ **Log errors for debugging** - console.error with context
- ✅ **Network error detection** - Handle offline/timeout scenarios
- ✅ **Permission errors** - Handle denied permissions gracefully
- ✅ **Graceful degradation** - App should function without permissions if possible

```typescript
// ✅ Good - Comprehensive error handling
async function requestDeviceActivityPermission(): Promise<boolean> {
  try {
    const granted = await DeviceActivityModule.requestPermission();

    if (!granted) {
      Alert.alert(
        'Permission Required',
        'SafeHaven needs permission to monitor device activity for your safety. This is the core feature of the app.',
        [
          { text: 'Cancel', style: 'cancel' },
          { text: 'Open Settings', onPress: () => Linking.openSettings() }
        ]
      );
      return false;
    }

    return true;
  } catch (error) {
    console.error('DeviceActivityPermission error:', error);

    Alert.alert(
      'Permission Error',
      'Unable to request device activity permission. Please try again or contact support.',
      [{ text: 'OK' }]
    );

    return false;
  }
}
```

---

## Development Guidelines

### Before Starting Any Task

1. **Read PROJECT-SPEC.md** - Understand the requirements
2. **Read PLAN.md** - Understand the implementation approach
3. **Start in Plan Mode** - Enter plan mode before implementing
4. **Ask clarifying questions** - If anything is unclear, ask before implementing
5. **Consider safety implications** - This is a life-safety app
6. **Think about elderly users** - Large buttons, simple UI, high contrast

### During Implementation

- **Test on physical device** - DeviceActivity requires physical iOS device
- **Handle errors explicitly** - Never ignore errors in safety-critical code
- **Large touch targets** - Minimum 60x60pt buttons for elderly users
- **High contrast UI** - Ensure text is readable for elderly users
- **Run TypeScript checks** - `npx tsc --noEmit` before committing
- **Commit early and often** - Small, focused commits
- **Update CLAUDE.md** - Use `#` command for learned patterns

### After Completing a Feature

- [ ] All TypeScript checks pass (`npx tsc --noEmit`)
- [ ] No console errors or warnings
- [ ] Tested on physical iOS device
- [ ] Error handling for all async operations
- [ ] User-friendly error messages
- [ ] Code follows style guidelines
- [ ] CLAUDE.md updated if needed
- [ ] Commits are clean and descriptive

### Git Commit Messages

Follow conventional commits format:

```
type(scope): subject

feat(auth): add role selection during sign up
feat(monitoring): add person detail screen with safety status
fix(checkin): resolve double check-in on quick tap
fix(escalation): prevent false alarms during vacation mode
docs(readme): update setup instructions
refactor(services): simplify error handling in CheckInService
test(checkin): add unit tests for check-in validation
chore(deps): update expo to SDK 51
```

**Types:** feat, fix, docs, style, refactor, test, chore

---

## Common Tasks

### Starting Development Server

```bash
# Start Expo development server
npm start

# Start on iOS simulator
npm run ios

# Start on Android emulator
npm run android

# Start with specific device
npm run ios -- --device "Your Device Name"
```

### Running Tests

```bash
# Unit tests (future)
npm test

# Watch mode
npm test -- --watch

# Type checking
npx tsc --noEmit

# Linting
npm run lint

# Auto-fix linting issues
npm run lint -- --fix
```

### Building for Testing

```bash
# Build iOS app for physical device testing
eas build --profile development --platform ios

# Build iOS app for TestFlight
eas build --profile preview --platform ios

# Build production iOS app
eas build --profile production --platform ios
```

### Database Operations

```bash
# Connect to Supabase project
# (Use Supabase dashboard or SQL editor)

# Run migrations (create new migration file)
# Edit in supabase/migrations/ directory
# Then run via Supabase dashboard

# View database
# Use Supabase dashboard → Table Editor
```

---

## SafeHaven-Specific Guidelines

### Two-User Architecture

**CRITICAL:** This app has TWO completely different user experiences based on role.

**Monitored Person (Elderly Parent):**
- Ultra-simple UI - ONE screen with check-in button
- Large touch targets (200x200pt button)
- High contrast colors
- Minimal text
- NO navigation, NO tabs, NO settings
- They barely interact with app - device activity is PRIMARY signal

**Monitor (Adult Child/Caregiver):**
- Dashboard with status cards
- Can monitor multiple people
- Receives push notifications for safety alerts
- Can add/remove monitored people
- Settings screen

**Implementation:**
- Detect user role on login
- Route to appropriate UI via RootNavigator
- Completely different navigation structures
- Different notification types for each role

### Device Activity Monitoring (PRIMARY FEATURE)

**Core Principle:** Device activity = person is safe. No device activity = emergency.

- Device activity is PRIMARY safety signal (not check-ins)
- ANY phone usage (calls, texts, other apps) = person is alive
- Check-in is SECONDARY (nice habit, not required)
- Elderly users won't reliably open app - that's okay!
- Monitor device activity continuously in background
- Update last_device_activity timestamp every 15 minutes

**Implementation:**
- iOS native module wrapping DeviceActivity framework
- Background fetch to check activity periodically
- Update Supabase with last_device_activity timestamp
- Respect privacy: detect activity exists, not track what they're doing

### Escalation Logic (LIFE-SAFETY CRITICAL)

**Escalation Decision Tree:**

```
✅ SAFE (No Action):
- Device activity in last 24 hours
- Vacation mode ON (override)

⚠️ WARNING (18 hours):
- No device activity for 18+ hours
- Vacation mode OFF
→ Send WARNING push to monitors

🚨 CRITICAL (24 hours):
- No device activity for 24+ hours
- Vacation mode OFF
→ TRIGGER ESCALATION
→ Send CRITICAL push to ALL monitors
→ Create escalation_event in database

✅ RESOLVED:
- Device activity detected
- OR user checks in
→ Resolve escalation
→ Send "all clear" notification
```

**Implementation Guidelines:**
- Check vacation mode FIRST (absolute override)
- Verify no race conditions (user checking in while detection runs)
- Log every escalation decision for debugging
- Use database transactions for escalation events
- Test extensively (simulate 24-hour waits)
- Handle false alarm scenarios gracefully

### Notification Guidelines

**For Monitored Person:**
- Daily gentle reminder to check in
- "Tap to check in with SafeHaven"
- Can be snoozed/dismissed
- NOT escalation notifications

**For Monitors:**
- Warning at 18 hours: "Mom hasn't used her phone in 18 hours"
- Critical at 24 hours: "ALERT: Mom hasn't used her phone in 24 hours"
- All clear: "Mom checked in - she's safe"
- Use critical alert flag for emergencies
- Notification tap opens app to person detail screen

### Background Task Guidelines

**Critical Requirements:**
- Use expo-background-fetch for periodic checks
- Check device activity every 15 minutes minimum
- Update last_device_activity in Supabase
- iOS throttles background tasks - need server-side backup
- Test background fetch on physical device
- Handle app termination and restart

### Onboarding Flow (Adult Child Setting Up Parent)

**Context:** Adult child sets up app on parent's phone

**Flow:**
1. "Who is this for?" (parent or self)
2. "How it works" (device monitoring, not app checking)
3. Create parent's account
4. Add child as monitor
5. Configure settings
6. Request permissions
7. Test check-in
8. Handoff to parent

**Guidelines:**
- Clear, reassuring copy
- Emphasize passive monitoring
- "You'll only hear from us if there's a problem"
- Large fonts, simple language
- Test button at end

### Testing Guidelines (iOS-Only MVP)

**CRITICAL:** DeviceActivity framework doesn't work in simulator. Must test on physical iOS devices.

**Test Scenarios:**
- Parent uses phone normally (calls, texts) but never opens SafeHaven → Should NOT escalate
- Parent's phone dies (24+ hours no activity) → Should escalate
- Parent checks in daily → Streak increments
- Parent misses check-in but uses phone → No escalation, just reminder
- Vacation mode ON → Never escalate
- Multiple monitors watching same person → All get notifications
- Monitor adds second person to watch → Dashboard shows both

**Test Devices:**
- iPhone 11+ (iOS 15+)
- Test on multiple iOS versions
- Test with different network conditions
- Test with airplane mode
- Test with battery saver mode

---

## Regression Prevention

When Claude makes a mistake, use the `#` command to add an instruction to this file automatically.

**Example:**
```
# always-validate-device-activity-timestamp-before-escalation
# check-vacation-mode-before-any-escalation-logic
# use-high-priority-for-critical-safety-alerts
```

### Current Regression Preventions

[Instructions added via `#` command will appear here]

---

## Troubleshooting

### Common Issues

**DeviceActivity not working in simulator**
```
ERROR: DeviceActivity framework requires physical iOS device
```
- DeviceActivity only works on real devices
- Must test on physical iPhone
- Use device debugging via USB

**Background tasks not running**
```
Background fetch not executing
```
- iOS throttles background tasks aggressively
- Test on device, not simulator
- Check Background App Refresh is enabled in iOS Settings
- Need server-side backup for reliability

**Supabase RLS blocking access**
```
ERROR: new row violates row-level security policy
```
- Check RLS policies in Supabase dashboard
- Ensure user has permission for operation
- Verify JWT token is valid
- Check if user_id matches authenticated user

**Notifications not appearing**
```
Push notification not received
```
- Check notification permissions granted
- Verify push token registered with Expo
- Check Expo push notification dashboard for errors
- Test with Expo push notification tool
- Critical alerts require special entitlement

**TypeScript errors after setup**
```bash
# Regenerate types
rm -rf node_modules
npm install
npx tsc --noEmit
```

---

## Resources

### Documentation
- [React Native Docs](https://reactnative.dev/docs/getting-started)
- [Expo Docs](https://docs.expo.dev/)
- [React Navigation Docs](https://reactnavigation.org/docs/getting-started)
- [Supabase Docs](https://supabase.com/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

### Internal Resources
- **PROJECT-SPEC.md:** Product and engineering requirements
- **PLAN.md:** Implementation plan reviewed by Opus
- **iOS Reference:** `/Users/jamesknaus/Development/safehavenx/` (original Swift app)

---

## Notes for Claude

### When Implementing Features

1. Always start by reading PROJECT-SPEC.md for requirements
2. Review PLAN.md for implementation approach
3. Review this CLAUDE.md for coding standards
4. Check .clinerules for code quality rules
5. Ask clarifying questions if requirements are ambiguous
6. Propose an implementation plan before writing code
7. Consider safety implications (lives depend on this working)
8. Think about elderly users (simple, large, high-contrast)
9. Test on physical iOS device (DeviceActivity requires it)
10. Update documentation when adding new patterns

### Context You Have Access To

- **PROJECT-SPEC.md** - Product and engineering requirements
- **PLAN.md** - Implementation plan (iOS-first MVP strategy)
- **CLAUDE.md** - This file with guidelines
- **.clinerules** - Code quality rules
- **Codebase** - All source files in /src
- **iOS Reference** - Original SwiftUI app for DeviceActivity implementation

### Critical Reminders for SafeHaven

- 🚨 **This is life-safety software** - Error handling is NOT optional
- 👵 **Elderly users** - Simple UI, large buttons, high contrast, clear text
- 📱 **Device activity is PRIMARY** - Not check-ins. Phone usage = safe.
- ⏰ **24-hour inactivity = emergency** - 18 hours = warning. Test thoroughly.
- 🏖️ **Vacation mode overrides everything** - Never escalate if vacation mode ON
- 👨‍👩‍👧 **Two user types** - Completely different UIs for monitored vs monitor
- 🔔 **Critical alerts only when needed** - Don't cry wolf with false alarms
- 📲 **iOS-first MVP** - Android deferred to Phase 2 post-MVP
- 🧪 **Physical device required** - DeviceActivity doesn't work in simulator
- 📝 **Log everything** - Escalation decisions must be auditable

---

*Last updated: 2026-01-19*
*Template version: 1.0 (SafeHaven customized)*
