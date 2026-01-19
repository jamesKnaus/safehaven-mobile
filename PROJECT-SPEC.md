# Project Specification: SafeHaven Mobile

> **Production-Ready Personal Safety App for Elderly Users**
> React Native/Expo rebuild following PSB System methodology

**Date Created:** 2026-01-19
**Last Updated:** 2026-01-19
**Status:** Setup Phase (Post-Planning, Pre-Implementation)

---

## Project Overview

### Purpose
SafeHaven is a personal safety monitoring app that provides peace of mind for families with elderly relatives living alone. The app passively monitors device activity and sends alerts to family members if their loved one hasn't used their phone in 24+ hours, indicating a potential emergency.

### Project Goal
**[X] Production Feature** - Building production-ready iOS app for real-world elderly safety monitoring

This is NOT a prototype. Lives depend on this system working correctly with minimal false alarms and no missed real emergencies.

### Success Criteria
- [ ] Elderly users can check in with one tap (optional habit building)
- [ ] Device activity monitoring works passively in background (primary safety signal)
- [ ] Adult children receive critical alerts when parent hasn't used phone in 24+ hours
- [ ] False alarm rate < 1% (vacation mode, proper edge case handling)
- [ ] Zero missed real emergencies in testing scenarios
- [ ] App Store approval with TestFlight distribution

---

## Milestones

### MVP (Version 1) - iOS Only
**Goal:** Launch functional iOS app with full device activity monitoring and two-user architecture

**Scope:**
- Two-user architecture: "Monitored Person" (elderly) and "Monitor" (adult children)
- One-tap check-in for monitored persons (ultra-simple UI)
- Dashboard for monitors showing status of multiple people they're watching
- iOS native module for DeviceActivity framework (primary safety signal)
- Push notifications (warning at 18hrs, critical at 24hrs no device activity)
- Many-to-many relationships (multiple monitors per person, multiple people per monitor)
- Basic escalation logic with vacation mode override
- In-person account setup flow

**Out of Scope for MVP:**
- Android version (Phase 2 post-MVP)
- SMS notifications (push only for MVP)
- Remote invitation system (in-person setup only)
- Medication tracking
- Check-in history beyond 7 days
- Advanced analytics/charts
- Payments/premium features

**Timeline:** 21-27 days of focused development

### Version 2 (Post-MVP)
**Goal:** Android support and enhanced monitoring features

**Scope:**
- Android app with simplified monitoring (no DeviceActivity equivalent)
- Remote invitation system for account setup
- SMS fallback for critical alerts
- Enhanced dashboard with charts and trends
- Custom escalation messages
- Check-in history view

**Timeline:** TBD based on MVP validation

### Version 3
**Goal:** Advanced features and premium tier

**Scope:**
- Medication reminders integration
- Location tracking (opt-in)
- Premium subscription model
- In-app messaging between monitor and monitored
- Advanced analytics for patterns

**Timeline:** TBD based on V2 success

---

# PART 1: PRODUCT REQUIREMENTS

## Who Is This For?

### Primary User Personas

**1. Monitored Person (Elderly Parent)**
- **Age:** 65-85+
- **Tech Skill:** Low - barely uses smartphone
- **Living Situation:** Lives alone or at elevated risk
- **Characteristics:**
  - May not remember to open apps regularly
  - Prefers simple, large-button interfaces
  - Uses phone mainly for calls/texts with family
  - Concerned about being a burden on children
- **Key Insight:** They are unlikely to reliably open SafeHaven app daily

**2. Monitor (Adult Child / Caregiver)**
- **Age:** 30-60
- **Tech Skill:** High - comfortable with apps
- **Motivation:** Peace of mind about parent's safety
- **Characteristics:**
  - Lives in different city/state from parent
  - Checks phone regularly throughout day
  - Wants reassurance parent is safe without constant calling
  - May be responsible for multiple elderly relatives (both parents, aunt, uncle)
  - Is the actual paying customer
- **Key Insight:** They pay for and set up the app, are the trusted contacts

## Problems We're Solving

### Problem 1: Elderly Person Falls/Emergency with No One to Help
**Current State:** Elderly person living alone has medical emergency (fall, heart attack, stroke). No one discovers them for days. Traditional medical alert systems require wearing device and pressing button - often not accessible during actual emergency.

**Desired State:** Family is automatically notified within 24 hours if elderly person hasn't used their phone (any app, call, text) - indicating they may be incapacitated.

**Impact:** Life-or-death. Affects millions of elderly people living alone. Early detection = higher survival rate.

### Problem 2: Adult Children Constantly Worried About Elderly Parents
**Current State:** Adult children call parents daily to "check in" - feels intrusive, time-consuming, creates guilt for both parties. Parents feel like they're being a burden.

**Desired State:** Passive monitoring provides reassurance without daily phone calls. Alerts only when actually needed.

**Impact:** Reduces anxiety for adult children, reduces burden feeling for parents, maintains independence while providing safety net.

### Problem 3: Existing Solutions Require Too Much Engagement
**Current State:** Medical alert buttons require elderly person to remember to wear device and press button during emergency. Daily check-in apps require elderly person to reliably open app and check in - unrealistic for users with memory issues or who barely use smartphones.

**Desired State:** Passive device activity monitoring works without requiring elderly person to do anything specific. Any phone usage = they're alive and active.

**Impact:** Works for elderly users who can't/won't engage with traditional solutions.

## User Flows

### Flow 1: Adult Child Sets Up Monitoring for Parent (In-Person)

**Trigger:** Adult child visits parent with parent's iPhone

**Steps:**
1. Adult child downloads SafeHaven on their own phone
2. Adult child creates "Monitor" account (email, password, name)
3. Adult child taps "Add Person to Monitor"
4. Adult child downloads SafeHaven on parent's iPhone
5. Adult child creates "Monitored" account for parent (name, email, password)
6. App links accounts via monitoring_relationships table
7. Adult child requests DeviceActivity permission on parent's phone
8. Adult child tests check-in button with parent ("See? Just tap this big button")
9. Adult child hands phone back to parent
10. Adult child's dashboard now shows parent's status card

**Success Criteria:**
- Entire setup takes < 10 minutes
- Parent sees ultra-simple one-button interface
- Adult child sees parent on their dashboard immediately
- DeviceActivity monitoring is active

**Edge Cases:**
- Parent already has email? Ask to create new one or use existing
- Parent doesn't want to be monitored? Require explicit consent screen
- Multiple siblings want to monitor same parent? Each sets up their own monitor account, all link to same parent

### Flow 2: Monitored Person's Daily Experience

**Trigger:** Parent opens SafeHaven app (optional)

**Steps:**
1. Parent opens app (sees ultra-simple screen with one giant button)
2. Parent taps check-in button
3. Button changes to green with checkmark "✓ Checked In Today"
4. Parent closes app
5. (Background) Device activity continues being tracked
6. (Next day) Button resets to "Tap to Check In" state

**Success Criteria:**
- Check-in takes < 2 seconds (one tap)
- Visual feedback is immediate and clear
- Parent can close app and forget about it
- Device monitoring continues in background

**Edge Cases:**
- Parent forgets to check in? NO escalation if device shows activity
- Parent doesn't open app for weeks? Device activity still tracked, no issue
- Parent's phone dies overnight? Device activity shows gap, but < 24 hours

### Flow 3: Monitor Views Parent's Status

**Trigger:** Adult child opens SafeHaven app

**Steps:**
1. Adult child sees dashboard with status cards
2. Each card shows:
   - Parent's name
   - Green checkmark "Safe - Active 2 hours ago"
   - Last check-in time
3. Adult child taps card to see details
4. Detail screen shows:
   - Safety status with explanation
   - Check-in history (last 7 days)
   - Device activity timeline
   - Vacation mode toggle
5. Adult child closes app, reassured

**Success Criteria:**
- Status is accurate and real-time (< 5 min lag)
- Visual indicators are immediately clear (green = safe)
- Can monitor multiple people at a glance

**Edge Cases:**
- Parent hasn't checked in but device active? Show "Safe" status, de-emphasize check-in
- No data yet (just set up)? Show "Setting up monitoring..."
- Connection issues? Show last known status with timestamp

### Flow 4: Critical Escalation (No Device Activity 24+ Hours)

**Trigger:** Background task detects no device activity for 24+ hours

**Steps:**
1. (18 hours) Background task detects 18 hours no activity
2. System sends WARNING push notification to monitors
   - "Mom hasn't used her phone in 18 hours - might want to check on her"
3. (24 hours) Background task detects 24 hours no activity
4. System creates escalation_event in database
5. System sends CRITICAL push notification to ALL monitors
   - "ALERT: Mom hasn't used her phone in 24 hours"
6. Monitors receive notification, tap to open app
7. Dashboard shows parent's card in RED with alert icon
8. Monitors call parent / go check on them
9. Parent's phone comes back online (or parent checks in via app)
10. System resolves escalation, sends "All Clear" notification

**Success Criteria:**
- Escalation triggers within 24-26 hours of last device activity
- ALL monitors are notified
- Notifications use critical alert (bypass Do Not Disturb)
- De-escalation is automatic when device activity resumes

**Edge Cases:**
- Parent on vacation? Vacation mode prevents escalation
- Phone battery died, comes back at hour 25? Escalation already sent, de-escalates automatically
- Parent checks in during escalation processing? Escalation canceled, race condition handled
- Monitor has notifications disabled? Still logged, other monitors notified
- Parent in different timezone? Calculate 24 hours in parent's local time

### Flow 5: Setting Vacation Mode

**Trigger:** Parent going on vacation, won't have phone / limited usage

**Steps:**
1. Adult child (monitor) opens SafeHaven
2. Taps parent's status card
3. Toggles "Vacation Mode" to ON
4. Confirmation: "Monitoring paused. No alerts will be sent."
5. Parent's status card shows yellow "Vacation Mode" badge
6. (After vacation) Monitor toggles vacation mode OFF
7. Normal monitoring resumes

**Success Criteria:**
- No escalations trigger while vacation mode ON
- Clear visual indicator on monitor's dashboard
- Easy to toggle on/off

**Edge Cases:**
- Parent wants to set their own vacation mode? Not in MVP (monitor controls it)
- Multiple monitors disagree (one enables, one disables)? Last action wins, log all changes
- Parent's phone active during vacation mode? Still tracked but no escalations

---

## Feature Requirements

### Feature 1: Two-User Architecture with Role-Based UI

**Description:** App detects user role on login and routes to appropriate UI experience.

**User Stories:**
- As a monitored person, I want to see only a simple check-in button, so I'm not overwhelmed
- As a monitor, I want to see a dashboard of everyone I'm watching, so I can check on multiple relatives at once

**Detailed Requirements:**
- [ ] Database has `role` field on users table ('monitored' or 'monitor')
- [ ] On sign-up, user selects role or role is assigned during setup
- [ ] On login, app reads role and routes accordingly:
  - 'monitored' → MonitoredUserScreen (one button)
  - 'monitor' → MonitorDashboard (status cards)
- [ ] No navigation tabs or settings for monitored users
- [ ] Full navigation for monitors (Dashboard, Settings)

**Acceptance Criteria:**
- [ ] Given I'm a monitored user, when I log in, then I see only the check-in screen
- [ ] Given I'm a monitor, when I log in, then I see dashboard with status cards
- [ ] Given a new user, when they sign up, then they must select or be assigned a role

### Feature 2: Ultra-Simple Check-In UI for Monitored Person

**Description:** Dead simple one-screen interface with giant check-in button.

**User Stories:**
- As an elderly user with limited tech skills, I want a giant button I can tap, so I don't get confused by the app
- As an elderly user, I want immediate visual feedback when I check in, so I know it worked

**Detailed Requirements:**
- [ ] Button is 200x200pt minimum
- [ ] Text is 24pt font minimum
- [ ] High contrast (white text on coral pink background)
- [ ] Two states: "Tap to Check In" (gray/coral) and "✓ Checked In Today" (green)
- [ ] Haptic feedback on tap
- [ ] Success animation (scale + fade)
- [ ] Show last check-in time below button
- [ ] Auto-reset at midnight
- [ ] No other UI elements (no tabs, no settings, no navigation)

**Acceptance Criteria:**
- [ ] Given I haven't checked in today, when I open the app, then I see "Tap to Check In" button
- [ ] Given I tap the button, when animation completes, then button shows "✓ Checked In Today"
- [ ] Given I've already checked in, when I open the app, then button is green with checkmark
- [ ] Given it's a new day, when I open the app, then button has reset to "Tap to Check In"

**UI/UX Notes:**
- Button must be tappable even with arthritic fingers (large target)
- Colors must be distinguishable for users with vision issues
- Animation must be clear but not distracting

### Feature 3: Monitor Dashboard with Status Cards

**Description:** Dashboard showing all people the monitor is watching, with clear safety status indicators.

**User Stories:**
- As a monitor, I want to see all my relatives' status at a glance, so I don't have to check multiple screens
- As a monitor, I want immediate visual indication if someone needs attention, so I can respond quickly

**Detailed Requirements:**
- [ ] Dashboard shows list of monitored persons as cards
- [ ] Each card displays:
  - Person's name
  - Safety status icon (✓ green, ⚠️ yellow, 🚨 red)
  - Last device activity: "Active 2 hours ago"
  - Last check-in: "Today 9:15am"
- [ ] Cards sorted by status (alerts first, then warnings, then safe)
- [ ] Pull-to-refresh updates all statuses
- [ ] Tap card to see detail screen
- [ ] "Add Person to Monitor" button at bottom
- [ ] Empty state: "No one being monitored yet - tap Add to get started"

**Acceptance Criteria:**
- [ ] Given I monitor 3 people, when I open dashboard, then I see 3 cards
- [ ] Given one person has no activity for 20 hours, when I view dashboard, then their card shows yellow warning
- [ ] Given person has activity in last 6 hours, when I view dashboard, then their card shows green safe
- [ ] Given I pull to refresh, when refresh completes, then all status times are updated

### Feature 4: iOS DeviceActivity Native Module (CRITICAL)

**Description:** Native iOS module wrapping DeviceActivity framework to track phone usage.

**User Stories:**
- As the system, I need to know when monitored person last used their phone (any app, call, text), so I can detect extended inactivity
- As a monitor, I want confidence that monitoring works even if parent never opens SafeHaven app, so I can trust the system

**Detailed Requirements:**
- [ ] Native Swift module using DeviceActivity framework
- [ ] Request FamilyControls authorization
- [ ] Monitor device activity in background
- [ ] Update last_device_activity timestamp in database every 15 minutes
- [ ] Expose JavaScript API:
  - `requestPermission(): Promise<boolean>`
  - `isAuthorized(): Promise<boolean>`
  - `getLastDeviceActivity(): Promise<Date | null>`
  - `startMonitoring(): Promise<void>`
  - `stopMonitoring(): Promise<void>`
- [ ] Handle app termination and restart (monitoring continues)
- [ ] Works with background app refresh

**Acceptance Criteria:**
- [ ] Given permission granted, when user uses phone (any app), then last_device_activity is updated within 15 minutes
- [ ] Given user hasn't used phone in 24 hours, when background task checks, then last_device_activity is > 24 hours old
- [ ] Given app is killed, when monitoring was active, then monitoring resumes on next app open
- [ ] Given app refresh runs, when device was recently active, then timestamp is updated in database

**Technical Notes:**
- Requires physical iOS device for testing (doesn't work in simulator)
- Requires FamilyControls entitlement in Info.plist
- May need Expo config plugin or bare workflow

### Feature 5: Escalation Logic with Multiple Failsafes

**Description:** Background system that detects prolonged inactivity and alerts monitors.

**User Stories:**
- As a monitor, I want to be alerted if parent hasn't used phone in 24 hours, so I can check on them
- As the system, I need to avoid false alarms by checking vacation mode and other signals, so monitors trust the alerts

**Detailed Requirements:**
- [ ] Background task checks device activity every hour
- [ ] At 18 hours no activity: Send WARNING push to monitors
- [ ] At 24 hours no activity: Create escalation_event, send CRITICAL push to ALL monitors
- [ ] Vacation mode = absolute override (never escalate)
- [ ] Race condition handling (check-in during escalation processing)
- [ ] Automatic de-escalation when device activity resumes
- [ ] Log all escalation attempts for debugging

**Escalation Decision Logic:**
```
✅ SAFE (No Action):
- Device activity in last 24 hours
→ No alerts sent

⚠️ WARNING (18 hours):
- No device activity for 18+ hours
- Vacation mode OFF
→ Send WARNING push: "Mom hasn't used phone in 18 hours"

🚨 CRITICAL (24 hours):
- No device activity for 24+ hours
- Vacation mode OFF
→ TRIGGER ESCALATION
→ Send CRITICAL push to ALL monitors: "ALERT: Mom hasn't used phone in 24 hours"

🛑 NEVER ESCALATE:
- Vacation mode is ON
```

**Acceptance Criteria:**
- [ ] Given device inactive 18 hours, when background check runs, then WARNING sent to monitors
- [ ] Given device inactive 24 hours, when background check runs, then CRITICAL escalation triggered
- [ ] Given vacation mode ON, when device inactive 48 hours, then NO escalation triggered
- [ ] Given device becomes active after escalation, when activity detected, then escalation auto-resolves

---

# PART 2: ENGINEERING REQUIREMENTS

## Tech Stack

### Frontend
- **Framework:** React Native + Expo SDK 50+
- **Language:** TypeScript (strict mode)
- **Navigation:** React Navigation 6.x (@react-navigation/native, @react-navigation/bottom-tabs)
- **State Management:** React Context + Hooks (no Redux for MVP)
- **Styling:** StyleSheet API with centralized theme
- **Key Libraries:**
  - expo-notifications (push notifications)
  - expo-device (device info)
  - expo-constants (app constants)
  - expo-background-fetch (background tasks)
  - expo-task-manager (task scheduling)
  - @react-native-async-storage/async-storage (local storage)
  - @react-native-picker/datetimepicker (time picker)
  - date-fns (date helpers)

### Backend
- **Database:** Supabase PostgreSQL (existing project)
- **Authentication:** Supabase Auth (email/password, Sign in with Apple)
- **Edge Functions:** Supabase Edge Functions (TypeScript/Deno)
- **Storage:** Supabase Storage (future - not MVP)
- **Realtime:** Supabase Realtime subscriptions (for dashboard live updates)

### Native Modules (iOS Only for MVP)
- **DeviceActivity Module:** Custom Swift module wrapping iOS DeviceActivity framework
- **FamilyControls:** iOS framework for Screen Time API access

### Third-Party Services
- **Expo Push Notifications:** Push notification delivery via Expo servers
- **GitHub:** Version control and CI/CD

### Development Tools
- **TypeScript:** Type safety
- **ESLint:** Code quality
- **EAS Build:** Expo Application Services for building iOS app
- **TestFlight:** iOS beta distribution

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    React Native App                      │
│  ┌────────────────┐         ┌─────────────────────┐    │
│  │ Monitored UI   │         │   Monitor UI        │    │
│  │ (One Button)   │         │   (Dashboard)       │    │
│  └────────────────┘         └─────────────────────┘    │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         React Navigation (Role-Based)            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │    Services (CheckIn, Monitoring, Escalation)    │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │       Repositories (Supabase Data Access)        │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │  iOS Native Module (DeviceActivity) - iOS Only   │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          ↓
              ┌───────────────────────┐
              │  Supabase Backend     │
              │  - PostgreSQL DB      │
              │  - Auth               │
              │  - Edge Functions     │
              │  - Realtime           │
              └───────────────────────┘
                          ↓
              ┌───────────────────────┐
              │  Expo Push Service    │
              └───────────────────────┘
```

### Three-Layer Architecture

**UI Layer** (Screens/Components)
- MonitoredUserScreen (check-in button)
- MonitorDashboard (status cards)
- MonitoredPersonCard component
- Settings screens
- Pure presentation logic only

**Service Layer** (Business Logic)
- AuthService: Authentication and session management
- CheckInService: Check-in creation and status
- MonitoringService: Managing monitoring relationships
- DeviceActivityService: Wrapper for native module
- EscalationService: Escalation detection and triggering
- NotificationService: Push notification handling

**Data Layer** (Repositories)
- UserRepository: User CRUD operations
- CheckInRepository: Check-in data access
- MonitoringRepository: Relationship data access
- EscalationRepository: Escalation event logging
- All Supabase queries contained here

### Database Schema (Extended from Existing)

**users** (extended)
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY REFERENCES auth.users(id),
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    role TEXT CHECK (role IN ('monitored', 'monitor')),
    last_check_in TIMESTAMPTZ,
    last_device_activity TIMESTAMPTZ,
    escalation_status TEXT DEFAULT 'none',
    vacation_mode BOOLEAN DEFAULT FALSE,
    check_in_time TIME DEFAULT '09:00:00',
    push_token TEXT,
    push_token_updated_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**monitoring_relationships** (new - many-to-many)
```sql
CREATE TABLE monitoring_relationships (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    monitor_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    monitored_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    relationship TEXT NOT NULL, -- 'daughter', 'son', etc.
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE(monitor_id, monitored_id)
);

CREATE INDEX idx_monitoring_monitor ON monitoring_relationships(monitor_id);
CREATE INDEX idx_monitoring_monitored ON monitoring_relationships(monitored_id);
```

**check_ins** (existing)
```sql
CREATE TABLE check_ins (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users(id),
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    method TEXT NOT NULL, -- 'manual', 'notification'
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**escalation_events** (extended)
```sql
CREATE TABLE escalation_events (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    monitored_id UUID NOT NULL REFERENCES users(id),
    triggered_at TIMESTAMPTZ NOT NULL,
    trigger_reason TEXT NOT NULL,
    confidence_level TEXT, -- 'high', 'medium', 'low'
    device_activity_detected BOOLEAN,
    resolved_at TIMESTAMPTZ,
    monitors_notified INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**notification_log** (new - per Opus recommendation)
```sql
CREATE TABLE notification_log (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id),
    type TEXT NOT NULL, -- 'escalation', 'warning', 'reminder'
    sent_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    delivered_at TIMESTAMPTZ,
    acknowledged_at TIMESTAMPTZ,
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**invitation_codes** (new - deferred to V2)
```sql
CREATE TABLE invitation_codes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    code TEXT UNIQUE NOT NULL, -- UUID v4
    monitor_id UUID NOT NULL REFERENCES users(id),
    monitored_email TEXT NOT NULL,
    monitored_name TEXT NOT NULL,
    relationship TEXT NOT NULL,
    used BOOLEAN DEFAULT FALSE,
    expires_at TIMESTAMPTZ NOT NULL, -- 48 hours
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### Row-Level Security Policies

**Users Table:**
```sql
-- Users can view their own profile
CREATE POLICY "Users can view own profile"
    ON users FOR SELECT
    USING (auth.uid() = id);

-- Monitors can view monitored persons' profiles
CREATE POLICY "Monitors can view monitored profiles"
    ON users FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM monitoring_relationships
            WHERE monitor_id = auth.uid()
            AND monitored_id = users.id
            AND active = true
        )
    );
```

**Monitoring Relationships:**
```sql
-- Monitors can view their monitoring relationships
CREATE POLICY "View own monitoring relationships"
    ON monitoring_relationships FOR SELECT
    USING (monitor_id = auth.uid() OR monitored_id = auth.uid());

-- Monitors can create relationships
CREATE POLICY "Create monitoring relationships"
    ON monitoring_relationships FOR INSERT
    WITH CHECK (monitor_id = auth.uid());
```

**Check-Ins:**
```sql
-- Users can view own check-ins
CREATE POLICY "View own check-ins"
    ON check_ins FOR SELECT
    USING (user_id = auth.uid());

-- Monitors can view monitored persons' check-ins
CREATE POLICY "Monitors can view monitored check-ins"
    ON check_ins FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM monitoring_relationships
            WHERE monitor_id = auth.uid()
            AND monitored_id = check_ins.user_id
            AND active = true
        )
    );
```

## Key Technical Decisions

### Decision 1: iOS-First MVP with Native DeviceActivity

**Context:** DeviceActivity framework is iOS-only. Android has no equivalent without significant compromises.

**Options Considered:**
1. Cross-platform simplified approach (app-open detection only)
2. Build both platforms with different capabilities
3. iOS-first with full DeviceActivity, Android later

**Decision:** Option 3 - iOS-first MVP

**Rationale:**
- DeviceActivity is proven (exists in current Swift app)
- Correct UX for elderly users (passive monitoring)
- Can validate product-market fit with iOS
- iPhone users more likely to be target demographic
- Android can launch later with learned lessons

**Trade-offs:**
- Smaller addressable market initially
- iOS-specific development effort
- May need to redesign for Android

### Decision 2: React Native/Expo vs Native iOS

**Context:** Existing app is Swift/SwiftUI. Could continue native or switch to cross-platform.

**Decision:** React Native + Expo

**Rationale:**
- Faster iteration (hot reload, declarative UI)
- Easier to find developers (JavaScript/TypeScript vs Swift)
- Eventual cross-platform support
- Supabase has excellent React Native SDK
- Can still use native modules for DeviceActivity

**Trade-offs:**
- Need to port Swift DeviceActivity code to native module
- Slightly larger app size
- Some iOS features harder to access

### Decision 3: Two-User Architecture with Role-Based Routing

**Context:** Original app was single-user. Could continue that model or split into two experiences.

**Decision:** Two distinct user experiences based on role

**Rationale:**
- Elderly users need ultra-simple UI (one button)
- Monitors need dashboard with multiple people
- Reflects real-world usage (child sets up for parent)
- Allows optimizing each experience independently

**Trade-offs:**
- More complex navigation logic
- Two code paths to maintain
- Database schema more complex (many-to-many relationships)

### Decision 4: Device Activity as Primary Signal (Not Check-Ins)

**Context:** Original app emphasized daily check-ins. Could continue that approach.

**Decision:** Device activity is primary, check-ins are secondary/optional

**Rationale:**
- Elderly users won't reliably check in via app
- Any phone usage (calls, texts, other apps) indicates safety
- Check-ins are good habit building but not required
- Passive monitoring is more realistic for target users

**Trade-offs:**
- Requires native module development
- More complex to implement
- Harder to test (24-hour waits)

### Decision 5: Push Notifications Only (No SMS for MVP)

**Context:** Existing app uses Twilio for SMS. Could continue that approach.

**Decision:** Push notifications only for MVP

**Rationale:**
- Simpler implementation (no Twilio integration)
- Lower cost (no per-SMS charges)
- Push notifications support critical alerts
- Can add SMS fallback in V2 if needed

**Trade-offs:**
- Less reliable than SMS
- Monitors must have app installed and notifications enabled
- No backup if push fails

## Testing Strategy

### Unit Tests
**Coverage Target:** 70%+ for business logic

**What to Test:**
- All service layer functions
- Escalation decision logic
- Timestamp calculations
- Data transformations
- Validation functions

### Integration Tests
**Key Flows to Test:**
- Account creation with role selection
- Linking monitor to monitored person
- Check-in flow (button tap → database update)
- Device activity update flow
- Escalation trigger conditions

### Manual Testing (Physical iOS Devices Required)
**Critical Scenarios:**
1. **Normal Operation:** Parent uses phone regularly, no alerts sent
2. **18-Hour Warning:** Simulate 18 hours inactivity, verify warning sent
3. **24-Hour Escalation:** Simulate 24 hours inactivity, verify critical alert sent
4. **Vacation Mode:** Enable vacation mode, simulate 48 hours inactivity, verify NO alert
5. **De-Escalation:** Escalation triggered, parent's phone comes back, verify auto-resolve
6. **Multiple Monitors:** One parent with 3 monitors, verify all receive alerts
7. **Multiple Monitored:** One monitor watching 5 people, verify dashboard shows all

### Testing Tools
- **TypeScript:** Compile-time type checking
- **Jest:** Unit testing
- **React Native Testing Library:** Component testing
- **Manual Testing:** Physical iOS devices (iPhone 12+, iOS 16+)
- **TestFlight:** Beta distribution for real-world testing

---

## Implementation Plan

See `PLAN.md` for detailed milestone breakdown (9 milestones, 21-27 days).

**Key Milestones:**
1. Foundation, Auth & Role System (3 days)
2. Monitored Person UI (2 days)
3. Monitor Dashboard UI (3 days)
4. Settings (1 day)
5. Push Notifications (2-3 days)
6. iOS DeviceActivity Native Module (2-3 days) ← CRITICAL
7. Escalation System (3-4 days) ← CRITICAL
8. Onboarding & Polish (2-3 days)
9. Testing & Deployment (3 days)

---

## Risks & Mitigation

### Risk 1: DeviceActivity Native Module Complexity
**Probability:** High
**Impact:** Very High (blocks entire MVP)
**Mitigation:**
- Reference existing Swift code from old app
- Allocate extra time (2-3 days vs 1 day)
- Have fallback to app-open detection if native module fails
- Test on physical devices early

### Risk 2: Push Notification Delivery Reliability
**Probability:** Medium
**Impact:** High (missed alerts = safety risk)
**Mitigation:**
- Use critical alerts (bypass Do Not Disturb)
- Log all notification attempts in notification_log table
- Implement retry logic
- Add SMS fallback in V2

### Risk 3: Background Task Throttling
**Probability:** High (iOS throttles background tasks)
**Impact:** Medium (delays in escalation)
**Mitigation:**
- Use server-side scheduled checks (Supabase Edge Function + pg_cron)
- Client background fetch as backup only
- Document tolerance: "Escalation between 24-26 hours"

### Risk 4: False Positive Escalations
**Probability:** Medium
**Impact:** High (erodes trust)
**Mitigation:**
- Vacation mode as absolute override
- 18-hour warning before 24-hour critical
- Extensive edge case testing
- Log all escalation decisions for analysis

### Risk 5: RLS Policy Complexity
**Probability:** Medium
**Impact:** High (data leakage or blocked access)
**Mitigation:**
- Test RLS policies thoroughly before launch
- Write integration tests for each policy
- Review with Opus before deployment

---

## Success Metrics (Post-Launch)

### Safety Metrics
- **False Positive Rate:** < 1% (escalations when person was actually safe)
- **False Negative Rate:** 0% (missed real emergencies)
- **Average Escalation Response Time:** < 2 hours from trigger to monitor action

### Usage Metrics
- **Daily Active Monitors:** > 70% (monitors check app daily)
- **Check-In Rate (Monitored):** 30-50% (optional, so low % is acceptable)
- **Device Activity Tracking Uptime:** > 99%

### Product Metrics
- **Monitor Retention:** > 80% after 30 days
- **NPS Score:** > 50 (would recommend to others)
- **App Store Rating:** > 4.5 stars

---

*Last updated: 2026-01-19*
*Following PSB System by Afar*
*Opus architectural review completed*
