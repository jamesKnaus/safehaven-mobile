# SafeHaven React Native Rebuild Plan
## Following the PSB System Methodology

**Date:** 2026-01-19
**Goal:** Rebuild SafeHaven as a production-ready React Native/Expo app using proper PSB system
**Approach:** Start fresh with MVP, following structured planning and setup from ProjectSetUpGuide

---

## Project Overview

### Current State
- Existing iOS app (SwiftUI) with comprehensive features
- Well-designed Supabase backend (PostgreSQL) with tables: users, trusted_contacts, check_ins, escalation_events, medications, medication_logs
- Features: daily check-ins, medication reminders, trusted contacts, escalation system, Screen Time integration
- Complete documentation and database schema

### Target State
- React Native/Expo cross-platform app (iOS + Android)
- Production-ready, sustainable architecture
- Start with simple MVP, build additional features properly
- Keep existing Supabase backend and schema
- Follow PSB methodology throughout development

### Why React Native/Expo?
- Cross-platform (iOS + Android) from single codebase
- JavaScript/TypeScript ecosystem
- Supabase has excellent React Native SDK
- Faster iteration for startup
- Easier to find developers

### CRITICAL DECISION: iOS-First MVP Strategy
**After Opus architectural review, we're adopting a phased approach:**

**Phase 1 (MVP): iOS Only with Full Device Activity Monitoring**
- Build native module for iOS DeviceActivity framework
- Full implementation of device activity as primary signal
- Leverages existing Swift code as reference
- Validates product-market fit with correct architecture

**Phase 2 (Post-MVP): Android with Simplified Monitoring**
- Android version uses app-open + notification response signals
- UsageStatsManager API if feasible (requires special permission)
- OR: Simplified approach without true device monitoring

**Rationale:**
- iOS DeviceActivity framework is proven (exists in current app)
- Android has no equivalent API without major compromises
- Better to launch iOS-only with correct UX than compromise both platforms
- Elderly iPhone users are target demographic anyway

---

## Critical User Personas & Product Architecture (COMPLETE REDESIGN!)

### TWO DIFFERENT USER TYPES = TWO DIFFERENT APP EXPERIENCES

**This is NOT one app with different screens - it's TWO distinct app modes/experiences!**

---

### USER TYPE 1: "Monitored Person" (Elderly Parent)
**Who:** The person being watched for safety
**Age:** 65-85+
**Tech Skill:** Low - barely uses phone
**Living Situation:** Alone or at risk

**What They See:**
```
┌─────────────────────────────────────┐
│                                     │
│          SafeHaven                  │
│                                     │
│    ┌───────────────────────────┐   │
│    │                           │   │
│    │                           │   │
│    │    [HUGE BUTTON]          │   │
│    │    "Tap to Check In"      │   │
│    │                           │   │
│    │                           │   │
│    └───────────────────────────┘   │
│                                     │
│    ✓ Checked in today              │
│    Last check-in: 2 hours ago      │
│                                     │
└─────────────────────────────────────┘
```

**That's it. Nothing else.**

**Features:**
- ONE screen: Big check-in button
- Two button states:
  - "Tap to Check In" (gray/coral, needs action)
  - "✓ Checked In Today" (green, success state)
- Minimal text: Last check-in time
- No settings, no navigation, no complexity
- Device activity monitored in background (invisible to them)

---

### USER TYPE 2: "Monitor" (Adult Child/Caregiver)
**Who:** The person watching/monitoring elderly relatives
**Age:** 30-60
**Tech Skill:** High
**Motivation:** Peace of mind about parent's safety
**Key Insight:** They might monitor MULTIPLE people (both parents, aunt, uncle, etc.)

**What They See:**
```
┌─────────────────────────────────────┐
│  SafeHaven - Monitoring Dashboard   │
│                                     │
│  People You're Monitoring           │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ Mom (Jane Smith)            │   │
│  │ ✓ Safe - Active 2 hours ago │   │
│  │ Last check-in: Today 9:15am │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ Dad (John Smith)            │   │
│  │ ✓ Safe - Active 30 min ago  │   │
│  │ Last check-in: Today 8:45am │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ Aunt Sally                  │   │
│  │ ⚠️ No activity 18 hours      │   │
│  │ Last check-in: Yesterday    │   │
│  └─────────────────────────────┘   │
│                                     │
│  [+ Add Person to Monitor]          │
│                                     │
│  [Tabs: Home | Settings]            │
└─────────────────────────────────────┘
```

**Features:**
- Dashboard with status cards for each person they monitor
- Each card shows:
  - Person's name
  - Safety status (green ✓ safe, yellow ⚠️ warning, red 🚨 emergency)
  - Last device activity time
  - Last check-in time
- Add new people to monitor
- Receive push notifications when someone is at risk
- Basic settings

---

### THE RELATIONSHIP MODEL

**Key Insight:** One monitor (child) can watch MULTIPLE monitored people (parents, relatives)

```
Monitor (Adult Child)
  ↓ watches
Monitored Person (Mom)

Monitor (Adult Child)
  ↓ watches
Monitored Person (Dad)

Monitor (Adult Child)
  ↓ watches
Monitored Person (Aunt Sally)
```

**AND:** One monitored person can have MULTIPLE monitors (multiple children watching same parent)

```
Monitored Person (Mom)
  ↑ watched by
Monitor (Daughter)
Monitor (Son)
Monitor (Neighbor)
```

**This is a MANY-TO-MANY relationship!**

---

### ONBOARDING FLOWS

**Flow 1: Child Setting Up Parent's Account**

1. Child downloads app on their own phone
2. Child creates their own "Monitor" account
3. Child chooses "Add person to monitor"
4. Child either:
   - **Option A:** Sets up parent's account remotely
     - Enters parent's name, phone, email
     - App sends invitation to parent's phone
     - Parent clicks link, downloads app, logs in
   - **Option B:** Uses parent's phone in person
     - Child downloads app on parent's phone
     - Child creates "Monitored Person" account for parent
     - Child links their monitor account to parent's account
     - Child hands phone back to parent
5. Accounts are now linked - child can see parent's status

**Flow 2: Parent Using App (After Setup)**

1. Parent opens app
2. Sees big check-in button
3. Taps button
4. Button changes to "✓ Checked In Today"
5. Parent closes app
6. (Device activity continues being monitored in background)

---

### PRODUCT IMPLICATIONS

**Two Different Builds?**
- Could be same codebase with role-based UI switching
- OR two separate apps: "SafeHaven" (for monitored) and "SafeHaven Monitor" (for caregivers)
- Recommend: Single app with role detection on login

**Database Schema Changes:**
- `users` table needs `role` field: "monitored" or "monitor"
- New `monitoring_relationships` table (many-to-many):
  - `monitor_id` (user who watches)
  - `monitored_id` (user being watched)
  - `relationship` (daughter, son, neighbor)
  - `active` (can be disabled)
- `trusted_contacts` table becomes obsolete - replaced by `monitoring_relationships`

**Notification Changes:**
- When monitored person has no device activity 24+ hours → push notification to ALL their monitors
- Monitors get push notifications, NOT SMS
- Monitored person gets gentle reminders to check in (optional)

**Authentication:**
- On login, check user role
- Route to appropriate UI:
  - "monitored" → Simple check-in screen
  - "monitor" → Dashboard with cards

This is essentially a **caregiver monitoring platform**, not just a personal safety app!

---

## PHASE 1: PLANNING (Following PSB System)

### Two Critical Questions

#### Question 1: What are we actually trying to do?
**Answer:** Building a production-ready app for real-world use

**Implications:**
- Need proper security and authentication
- Require error handling and edge cases
- Must handle production scenarios (poor network, device permissions, etc.)
- Need testing strategy
- Proper notification handling
- Production deployment setup

#### Question 2: What are the milestones?

**Version 1 (MVP)** - Core Safety Check-In System
- User authentication (email/password + Sign in with Apple)
- Daily check-in button with streak tracking
- Trusted contacts management (add/edit/delete up to 5 contacts)
- Check-in time configuration
- Basic escalation system (missed check-in → notify contacts)
- Push notifications (daily reminder)
- Vacation mode toggle

**Version 2** - Enhanced Safety Features
- Alert delay customization
- Reminder style options (gentle vs persistent)
- Custom escalation messages per contact
- Check-in history view
- Help/FAQ section
- Enhanced error handling and offline support

**Version 3** - Advanced Features
- Medication reminders (full tracking system)
- Contact photos and storage
- SMS escalation via Twilio
- Advanced analytics

**Note:** Payments/premium features deferred - not a priority for now

---

## MVP Scope Definition (REDESIGNED FOR TWO USER TYPES)

### In Scope for MVP

**CORE ARCHITECTURE**
- Two distinct user experiences in one app
- Role-based routing on login ("monitored" vs "monitor")
- Many-to-many relationship between monitors and monitored people
- Push notifications only (NO SMS for MVP)

---

### FEATURES FOR "MONITORED PERSON" (Elderly Parent)

**Their App Experience:**
- **ONE SCREEN: Check-in button**
- Two button states:
  - "Tap to Check In" (not checked in)
  - "✓ Checked In Today" (checked in)
- Show last check-in time below button
- NO navigation, NO tabs, NO settings
- NO complexity - just the button
- Large fonts, high contrast, huge touch target

**Background Features (Invisible to Them):**
- Device activity monitoring continuously
- Daily check-in reminder notification
- Track last_check_in timestamp
- Track last_device_activity timestamp
- Vacation mode (set by monitor, not by them)

---

### FEATURES FOR "MONITOR" (Adult Child/Caregiver)

**Their App Experience:**

**Dashboard Screen:**
- List of people they're monitoring (contact cards)
- Each card shows:
  - Person's name
  - Safety status icon (✓ green safe, ⚠️ yellow warning, 🚨 red emergency)
  - Last device activity: "Active 2 hours ago"
  - Last check-in: "Today 9:15am"
- Tap card to see more details
- "Add Person to Monitor" button

**Add Person Flow:**
- Enter person's name, phone, email, relationship
- Two options:
  - Send invitation (remote setup)
  - Create account now (in-person setup)
- Link accounts via monitoring_relationships

**Person Detail Screen:**
- Person's full info
- Safety status history
- Check-in history (last 7 days)
- Device activity log
- Enable/disable monitoring
- Remove person
- Set vacation mode for them

**Notifications Received:**
- Push notification when monitored person has no device activity 24+ hours
- Push notification when monitored person checks in after escalation
- Critical alerts for emergencies

**Settings Screen:**
- Account info (name, email)
- Notification preferences
- Logout

---

### SHARED FEATURES

**Authentication (Both Roles)**
- Email/password sign up and login
- Sign in with Apple
- Session management
- Password reset flow
- Profile creation with role selection

**Device Activity Monitoring (Primary Safety Feature)**
- Monitor device activity continuously - THIS IS THE MAIN FEATURE
- ANY phone usage (calls, texts, any apps) = user is safe
- Detect prolonged inactivity (24+ hours of no device use) = emergency
- This is PRIMARY signal because elderly users won't reliably open app
- Check-in is SECONDARY signal (nice bonus, not required)
- Privacy-focused: detect activity exists, not track what they're doing
- Works even if monitored person never opens SafeHaven app

**Escalation Logic (Background Service)**
- Detect no device activity for 24+ hours
- Trigger escalation event
- Send push notifications to ALL monitors of that person
- Log escalation event to database
- De-escalate if monitored person shows activity or checks in
- **IMPORTANT:** Device activity is PRIMARY trigger, not missed check-in

**Notifications**
- **For Monitored Person:**
  - Daily gentle reminder to check in (optional, not critical)
  - NOT escalation notifications (they don't need to see these)

- **For Monitors:**
  - Critical alert when monitored person has 24+ hours no activity
  - Warning notification at 18 hours no activity
  - "All clear" notification when issue resolves

---

### Database Schema for MVP

**users**
- id (uuid, PK)
- email (text, unique)
- name (text)
- **role (text) - 'monitored' or 'monitor'** ← NEW
- last_check_in (timestamptz)
- last_device_activity (timestamptz)
- escalation_status (text) - 'none', 'warning', 'critical', 'escalated'
- vacation_mode (boolean)
- check_in_time (time) - reminder time
- created_at, updated_at

**monitoring_relationships** ← NEW TABLE (replaces trusted_contacts)
- id (uuid, PK)
- monitor_id (uuid, FK to users) - the person watching
- monitored_id (uuid, FK to users) - the person being watched
- relationship (text) - 'daughter', 'son', 'neighbor', etc.
- active (boolean)
- created_at (timestamptz)
- Unique constraint: (monitor_id, monitored_id)

**check_ins**
- id (uuid, PK)
- user_id (uuid, FK to users) - the monitored person
- timestamp (timestamptz)
- method (text) - 'manual', 'notification'
- created_at

**escalation_events**
- id (uuid, PK)
- monitored_id (uuid, FK to users) - the person at risk
- triggered_at (timestamptz)
- trigger_reason (text) - 'device_inactivity', 'no_checkin_and_inactivity'
- confidence_level (text) - 'high', 'medium', 'low'
- device_activity_detected (boolean)
- resolved_at (timestamptz, nullable)
- monitors_notified (integer) - count
- created_at

**invitation_codes** ← NEW TABLE (for remote setup)
- id (uuid, PK)
- code (text, unique) - invitation code (UUID v4, 32+ chars)
- monitor_id (uuid, FK to users) - who sent invitation
- monitored_email (text) - parent's email
- monitored_name (text) - parent's name
- relationship (text)
- used (boolean)
- expires_at (timestamptz) - 48 hour expiration
- created_at

**notification_log** ← NEW TABLE (for debugging, per Opus recommendation)
- id (uuid, PK)
- user_id (uuid, FK to users)
- type (text) - 'escalation', 'warning', 'reminder'
- sent_at (timestamptz)
- delivered_at (timestamptz, nullable)
- acknowledged_at (timestamptz, nullable)
- payload (jsonb)
- created_at

**Additional fields for users table:**
- push_token (text) - Expo push token
- push_token_updated_at (timestamptz)

---

### Out of Scope for MVP

- Medication tracking
- Contact photos
- SMS notifications - push notifications only
- Check-in history view (beyond last 7 days)
- Advanced analytics/charts
- Payments/premium features
- Custom escalation messages
- Alert delay customization (use 24 hour default)
- Help/FAQ section
- In-app messaging between monitor and monitored person
- Location tracking

---

## PHASE 2: PROJECT SETUP (7-Step Checklist)

### Step 1: Initialize Git & GitHub
```bash
# New directory for React Native project
mkdir safehaven-mobile
cd safehaven-mobile
git init
gh repo create safehaven-mobile --public --source=. --remote=origin
```

### Step 2: Initialize React Native/Expo Project
```bash
npx create-expo-app@latest . --template blank-typescript

# Core dependencies
npm install @supabase/supabase-js @react-native-async-storage/async-storage
npm install react-native-url-polyfill

# Navigation
npm install @react-navigation/native @react-navigation/bottom-tabs
npm install react-native-safe-area-context react-native-screens

# Notifications & Background Tasks
npm install expo-notifications expo-device expo-constants
npm install expo-background-fetch expo-task-manager

# Device Activity Monitoring
npm install expo-device-activity || # May need native modules or custom implementation

# UI Components
npm install @react-native-picker/datetimepicker
npm install date-fns # For date helpers

# Development
npm install -D @types/react @types/react-native
```

### Step 3: Create PROJECT-SPEC.md
Using `project-spec-template.md` as base, create comprehensive spec document with:
- Product requirements (who, what, why)
- Engineering requirements (tech stack, architecture)
- User flows for MVP features
- Feature requirements with acceptance criteria
- Data models and API design
- Testing strategy

### Step 4: Create CLAUDE.md
Using `CLAUDE-template.md` as base, customize for SafeHaven with:
- Project overview and goals
- Tech stack (React Native, Expo, TypeScript, Supabase)
- Coding standards (TypeScript strict mode, functional components, async/await)
- Project structure conventions
- Development guidelines
- Common tasks (run dev, build, test)
- SafeHaven-specific patterns (check-in logic, escalation system)

### Step 5: Create .clinerules
```
# Code Quality Rules
- No unused imports or variables
- No console.logs in production code
- All async functions must have error handling
- Maximum function length: 50 lines
- All components must use TypeScript types

# TypeScript Rules
- Enable strict mode
- No 'any' types without justification
- All props must have explicit types
- Use interfaces for component props

# React Native Rules
- Use functional components with hooks
- Use StyleSheet.create for styles
- Handle platform differences (iOS/Android)
- Always check permissions before using device features

# Testing Rules
- Critical user flows must have tests
- Mock Supabase calls in tests
- Test error scenarios
```

### Step 6: Configure Supabase
- Use existing Supabase project from iOS version
- Update RLS policies if needed for mobile access
- Configure environment variables in .env:
  - EXPO_PUBLIC_SUPABASE_URL
  - EXPO_PUBLIC_SUPABASE_ANON_KEY

### Step 7: Setup Project Structure
```
/safehaven-mobile
  /src
    /components       # Reusable UI components
      /common         # Button, Card, Input, etc.
      /checkin        # CheckInButton, StreakBadge, etc.
      /contacts       # ContactCard, ContactForm, etc.
    /screens          # Screen components
      /auth           # LoginScreen, SignUpScreen, etc.
      /home           # HomeScreen (main check-in)
      /contacts       # ContactsScreen, AddContactScreen
      /settings       # SettingsScreen
      /onboarding     # OnboardingScreen
    /navigation       # Navigation setup
      AppNavigator.tsx
      TabNavigator.tsx
    /services         # Business logic
      AuthService.ts
      CheckInService.ts
      ContactService.ts
      NotificationService.ts
      EscalationService.ts
      DeviceActivityService.ts   # NEW: Monitor device activity
      AppActivityService.ts      # NEW: Track app opens
      BackgroundTaskService.ts   # NEW: Background check scheduling
    /repositories     # Data access layer
      UserRepository.ts
      ContactRepository.ts
      CheckInRepository.ts
    /hooks            # Custom React hooks
      useAuth.ts
      useCheckIn.ts
      useContacts.ts
    /contexts         # React Context providers
      AuthContext.tsx
      CheckInContext.tsx
    /types            # TypeScript type definitions
      models.ts
      api.ts
    /utils            # Helper functions
      constants.ts
      dateHelpers.ts
      validators.ts
    /config           # Configuration
      supabase.ts
      theme.ts
  /assets             # Images, fonts, etc.
  App.tsx             # Root component
  app.json            # Expo configuration
  .env                # Environment variables (gitignored)
  tsconfig.json       # TypeScript configuration
```

---

## PHASE 3: IMPLEMENTATION PLAN

### Implementation Order (REDESIGNED FOR TWO-USER ARCHITECTURE)

#### Milestone 1: Foundation, Auth & Role System
**Duration: 3 days**

1. **Project Setup**
   - Initialize Expo project
   - Install dependencies (including push notifications)
   - Configure Supabase client
   - Setup TypeScript and linting
   - Create project structure for two-user system

2. **Database Setup**
   - Create new migration for MVP schema:
     - Add `role` field to users ('monitored' or 'monitor')
     - Create `monitoring_relationships` table
     - Create `invitation_codes` table
     - Update escalation_events for new schema
   - Run migrations on Supabase
   - Test RLS policies for both roles

3. **Authentication with Role Selection**
   - AuthContext with Supabase Auth
   - AuthService (sign up, login, logout, session management)
   - Login screen UI (email/password)
   - Sign up screen UI with role selection:
     - "I need monitoring" (creates monitored account)
     - "I'm setting this up for someone" (creates monitor account)
   - Sign in with Apple (iOS)
   - Password reset screen

4. **Role-Based Routing**
   - RootNavigator detects user role on login
   - Routes to appropriate UI:
     - "monitored" → MonitoredUserScreen (just check-in button)
     - "monitor" → MonitorDashboard (status cards)
   - Auth flow vs Main app flow
   - Safe area handling

#### Milestone 2: Monitored Person UI (Simple Check-In Screen)
**Duration: 2 days**

**IMPORTANT:** This is the ULTRA-SIMPLE view for elderly users. Just one screen with a button.

1. **Data Models & Types**
   - User model with role field
   - CheckIn model
   - MonitoringRelationship model
   - TypeScript interfaces

2. **Check-In Service Layer**
   - CheckInService (createCheckIn, getLastCheckIn)
   - CheckInRepository (database access)
   - useCheckIn hook

3. **MonitoredUserScreen (ONE SCREEN - ENTIRE APP FOR THEM)**

   **Layout:**
   ```
   ┌─────────────────────────────────────┐
   │          SafeHaven                  │
   │                                     │
   │                                     │
   │    ┌───────────────────────────┐   │
   │    │                           │   │
   │    │                           │   │
   │    │   [HUGE BUTTON 200x200]   │   │
   │    │   "Tap to Check In"       │   │
   │    │                           │   │
   │    │                           │   │
   │    └───────────────────────────┘   │
   │                                     │
   │    Last check-in: Today 9:15am     │
   │                                     │
   └─────────────────────────────────────┘
   ```

   **Two Button States:**
   - **Not Checked In:** Coral/gray button, text "Tap to Check In"
   - **Checked In:** Green button with checkmark, text "✓ Checked In Today"

   **Design Requirements:**
   - Button: 200x200pt minimum, coral pink
   - Text: 24pt font minimum
   - High contrast (white text on coral)
   - Haptic feedback on tap
   - Success animation (scale + fade)
   - Auto-reset at midnight

   **That's it. No other screens, no navigation, no settings.**

4. **Check-In Logic**
   - Create check-in record in database
   - Update user's last_check_in timestamp
   - Show success state
   - Store method: 'manual'

#### Milestone 3: Monitor Dashboard UI (Status Cards)
**Duration: 3 days**

1. **MonitoringRelationships Service**
   - MonitoringService (create, list, delete relationships)
   - Get all monitored people for a monitor
   - Get all monitors for a monitored person
   - MonitoringRepository

2. **Monitor Dashboard Screen**

   **Layout:**
   ```
   ┌─────────────────────────────────────┐
   │  SafeHaven Dashboard                │
   │                                     │
   │  People You're Monitoring           │
   │                                     │
   │  [Status Card: Mom]                 │
   │  [Status Card: Dad]                 │
   │  [Status Card: Aunt Sally]          │
   │                                     │
   │  [+ Add Person to Monitor]          │
   │                                     │
   │  [Bottom Tabs: Dashboard|Settings]  │
   └─────────────────────────────────────┘
   ```

   **Components:**
   - MonitoredPersonCard component:
     - Person's name
     - Safety status icon (✓ ⚠️ 🚨)
     - Last device activity relative time
     - Last check-in time
     - Tap to see details
   - Empty state: "No one being monitored yet"
   - Pull-to-refresh for status updates

3. **Person Detail Screen**
   - Full person info
   - Safety status with details
   - Check-in history (last 7 days)
   - Device activity timeline
   - Vacation mode toggle
   - Remove person button
   - Disable monitoring toggle

4. **Add Person Flow**
   - "Add Person to Monitor" button on dashboard
   - Form: name, email, phone, relationship
   - Two options:
     - "Set up in person" → creates account immediately, generates temp password
     - "Send invitation" → creates invitation code, sends email
   - Link accounts via monitoring_relationships table

5. **Invitation System**
   - Generate unique invitation codes
   - Send email with app download link + code
   - Invitation redemption flow
   - Link accounts when invitation accepted

#### Milestone 4: Settings (Both Roles)
**Duration: 1 day**

1. **Monitored Person Settings**
   - NONE - they don't need settings
   - Everything configured by their monitors

2. **Monitor Settings Screen**
   - Account info (name, email)
   - Notification preferences
   - Logout button
   - App version
   - Support/help link

#### Milestone 5: Push Notifications (Two Types)
**Duration: 2-3 days**

1. **Notification Service**
   - Request push notification permissions
   - Register device token with Supabase
   - Store device tokens in users table (push_token field)
   - NotificationService (send, schedule, cancel)

2. **Notifications for Monitored Person**
   - Daily gentle reminder to check in
     - Scheduled at check_in_time
     - "Tap to check in with SafeHaven"
     - Opens app when tapped
     - Can be snoozed/dismissed
   - NO escalation notifications (they don't need to see these)

3. **Notifications for Monitors**
   - **Critical Alert:** "Mom hasn't used her phone in 24 hours"
     - Sent when monitored person hits 24 hours no device activity
     - High priority/critical alert
     - Tap opens app to person detail screen
   - **Warning Alert:** "Mom hasn't used her phone in 18 hours"
     - Sent as early warning
     - Medium priority
   - **All Clear:** "Mom checked in - she's safe"
     - Sent when issue resolves
     - Low priority

4. **Notification Delivery**
   - Use expo-notifications for local notifications
   - Use Supabase Edge Function to send push notifications via Expo Push API
   - Queue notifications in database for reliability
   - Handle notification taps (deep linking to appropriate screen)

#### Milestone 6: iOS Native Module for Device Activity (CRITICAL)
**Duration: 2-3 days**

**IMPORTANT:** This is the foundation of the safety system. Must be completed before escalation logic.

**Based on existing Swift code at:**
- `/Users/jamesknaus/Development/safehavenx/safehaven/Core/Services/ScreenTimeService.swift`
- `/Users/jamesknaus/Development/safehavenx/safehaven/Core/Services/InactivityMonitorService.swift`

**Implementation:**

1. **Create Expo Config Plugin for DeviceActivity**
   - Use `expo-config-plugin` to add native iOS module
   - OR: Eject to bare workflow if needed
   - Add FamilyControls entitlement to Info.plist

2. **Port Swift ScreenTimeService to Native Module**
   - Request DeviceActivity authorization
   - Monitor device activity in background
   - Track last_device_activity timestamp
   - Expose JavaScript API: `DeviceActivityModule.requestPermission()`, `DeviceActivityModule.getLastActivity()`

3. **Native Module API Design**
   ```typescript
   interface DeviceActivityModule {
     requestPermission(): Promise<boolean>;
     isAuthorized(): Promise<boolean>;
     getLastDeviceActivity(): Promise<Date | null>;
     startMonitoring(): Promise<void>;
     stopMonitoring(): Promise<void>;
   }
   ```

4. **Update last_device_activity in Supabase**
   - Periodically (every 15 minutes) update database
   - Use background fetch to keep running
   - Handle app termination and restart

5. **Testing on Real iOS Device**
   - Cannot test in simulator (DeviceActivity requires physical device)
   - Test: Phone used normally → activity tracked
   - Test: Phone idle 24 hours → no activity recorded
   - Test: Background app refresh → still tracking

**Deliverable:** Working iOS native module that tracks device activity reliably.

#### Milestone 7: Escalation System with Multiple Failsafes (CRITICAL)
**Duration: 3-4 days**

**IMPORTANT:** This is a life-safety feature. We need multiple verification methods to avoid both false alarms AND missed real emergencies.

1. **Multi-Layered Detection System (REORDERED FOR ELDERLY USERS)**

   **Primary Detection: Device Activity Monitoring (MOST IMPORTANT)**
   - Monitor device activity/screen time continuously
   - ANY phone usage (calls, texts, ANY apps) = user is alive and active
   - Track last_device_activity timestamp
   - No device activity for 24+ hours = CRITICAL emergency signal
   - This is the MAIN safety mechanism since elderly users won't reliably open app

   **Secondary Detection: Check-In Status (Nice to Have)**
   - User can check in via app once per day
   - Builds habit and gives extra confidence
   - Track last_check_in timestamp in database
   - BUT: Missing check-in while device is active = NOT an emergency
   - Use for gentle reminders, not escalation

   **Tertiary Detection: App Open Events (Bonus Data)**
   - Track when user opens app (even without checking in)
   - Recent app opens show engagement with SafeHaven
   - Helps identify if user is trying to check in but having issues
   - Provides data for improving UX

2. **Escalation Decision Logic (SIMPLIFIED - Device Activity is PRIMARY)**

   **Core Principle:** Device activity = safe. No device activity = emergency.

   ```
   ESCALATION LOGIC:

   ✅ SAFE (No Action):
   - Device activity in last 24 hours
   - ANY phone usage = they're alive
   - Check-in is nice but NOT required
   → No escalation, all is well

   ⚠️ WARNING (18 hours):
   - No device activity for 18+ hours
   - Vacation mode OFF
   → Send WARNING push notification to monitors
   → "Mom hasn't used her phone in 18 hours - checking on her?"
   → Do NOT escalate yet

   🚨 CRITICAL (24 hours):
   - No device activity for 24+ hours
   - Vacation mode OFF
   → TRIGGER ESCALATION
   → Send CRITICAL push notification to ALL monitors
   → "ALERT: Mom hasn't used her phone in 24 hours"
   → Create escalation_event in database
   → Set escalation_status = 'escalated'

   🛑 NEVER ESCALATE:
   - Vacation mode is ON (absolute override)
   - Device activity detected (even without check-in)
   ```

   **Key Insight:** We only care about device activity. Check-ins are optional bonus data for building habits.

3. **False Alarm Prevention**
   - Check vacation mode status (never escalate if on)
   - Verify user is not in middle of checking in (race condition)
   - Check for recent network connectivity issues
   - Log all detection attempts for debugging
   - Send warning notifications before escalation

4. **Escalation Service Implementation**
   - EscalationService with multi-factor decision logic
   - DeviceActivityService for screen time monitoring
   - AppActivityService to track app opens
   - EscalationRepository for event logging
   - Escalation state machine (Warning → Critical → Escalated → Resolved)

5. **Device Activity Monitoring**
   - Use Expo background fetch for periodic checks
   - Store last_device_activity timestamp
   - Use expo-device-activity or native modules
   - Respect privacy: only detect activity, not what user is doing
   - Handle permissions gracefully

6. **Escalation Execution**
   - Log escalation event to database
   - Mark all active trusted contacts for notification
   - For MVP: database logging only (no actual SMS/push to contacts)
   - Track: contacts_notified, trigger_reason, confidence_level
   - Allow manual de-escalation

7. **De-Escalation Flow**
   - User checks in → immediately resolve escalation
   - Update escalation event with resolved_at timestamp
   - Send "all clear" notification to user
   - Log resolution for audit trail

8. **Testing Critical Scenarios**
   - Test: User misses check-in, device inactive → escalation triggers
   - Test: User misses check-in, device active → warning sent, no escalation
   - Test: User in vacation mode → no escalation
   - Test: False alarm scenarios (network issues, app crashes)
   - Test: De-escalation when user checks in late
   - Test: Multiple missed days handling

#### Milestone 8: Onboarding & Polish
**Duration: 2-3 days**

1. **Onboarding Flow (REDESIGNED FOR ADULT CHILD SETTING UP FOR PARENT)**

   **Context:** Adult child is setting up app on parent's phone

   **Screen 1: Who is this for?**
   - "Are you setting this up for yourself or someone else?"
   - Option A: "For myself" (simple flow)
   - Option B: "For my parent/family member" (assisted setup)
   - Explanation of how SafeHaven works

   **Screen 2: How It Works**
   - "SafeHaven monitors your parent's phone activity to ensure they're safe"
   - Explains device monitoring (PRIMARY feature)
   - Explains optional check-ins (SECONDARY feature)
   - "Your parent doesn't need to open the app daily - just use their phone normally"
   - "You'll be notified only if there's 24+ hours of no phone activity"

   **Screen 3: Create Parent's Account**
   - Enter parent's name
   - Enter parent's email (or create one for them)
   - Set up password
   - "Your parent can use this to log in if needed"

   **Screen 4: Add Yourself as Trusted Contact**
   - "You'll be notified if we detect a safety concern"
   - Enter your name
   - Enter your phone number
   - Relationship: "Son", "Daughter", "Sibling", etc.
   - Option to add more contacts (siblings, neighbors)

   **Screen 5: Configure Settings**
   - Set check-in reminder time (optional)
   - Enable device activity monitoring (required)
   - "Recommend daily check-ins, but phone activity is what we'll monitor"

   **Screen 6: Permissions**
   - Request notification permissions
   - Request device activity/usage permissions
   - Explain why each is needed
   - "These let SafeHaven monitor your parent's safety passively"

   **Screen 7: Test & Handoff**
   - "Let's test it! Tap the check-in button"
   - Parent taps button, sees success
   - "Great! You're all set. Your parent can close the app now."
   - "SafeHaven will monitor in the background. You'll only hear from us if needed."

   **Alternative: Self-Setup Flow** (if user selected "For myself")
   - Simplified 3-screen flow
   - Same core features but different copy
   - "SafeHaven monitors your phone activity for safety"

2. **Error Handling**
   - Network error handling
   - Form validation messages
   - Error boundary component
   - User-friendly error messages

3. **Loading States**
   - Skeleton screens
   - Loading spinners
   - Pull-to-refresh

4. **Polish**
   - Animations and transitions
   - Haptic feedback
   - Color scheme consistency
   - Responsive design

#### Milestone 9: Testing & Deployment Prep
**Duration: 3 days** (increased per Opus recommendation)

1. **Testing** (iOS-Only for MVP)
   - Manual testing of all flows on physical iOS device
   - Test notification handling (including critical alerts)
   - Test background tasks and DeviceActivity monitoring
   - Test escalation detection with real 24-hour waits
   - Edge case testing (phone dying, airplane mode, etc.)
   - Multi-device testing (different iPhone models, iOS versions)

2. **Deployment Setup**
   - Configure app.json for production
   - Set up EAS Build
   - Configure app icons and splash screens
   - Prepare for App Store/Play Store
   - Production environment variables

---

## Critical Files to Work With

### Existing Files (Reference)
- `/Users/jamesknaus/Development/safehavenx/supabase/migrations/` - Database schema
- `/Users/jamesknaus/Development/safehavenx/Documentation/README.md` - Feature specifications
- `/Users/jamesknaus/Development/safehavenx/safehaven/Core/Models/` - Data model references

### New Files to Create
- `safehaven-mobile/PROJECT-SPEC.md` - Comprehensive project specification
- `safehaven-mobile/CLAUDE.md` - Project guidelines for Claude
- `safehaven-mobile/.clinerules` - Code quality rules
- `safehaven-mobile/src/config/supabase.ts` - Supabase client configuration
- `safehaven-mobile/src/services/AuthService.ts` - Authentication logic
- `safehaven-mobile/src/services/CheckInService.ts` - Check-in logic
- `safehaven-mobile/src/services/ContactService.ts` - Contact management
- `safehaven-mobile/src/services/NotificationService.ts` - Notification handling
- `safehaven-mobile/src/services/EscalationService.ts` - Escalation logic

---

## Technical Architecture

### Tech Stack
- **Framework:** React Native + Expo (SDK 50+)
- **Language:** TypeScript (strict mode)
- **Backend:** Supabase (existing project)
  - Database: PostgreSQL with RLS
  - Auth: Supabase Auth
  - Storage: Supabase Storage (future)
- **Navigation:** React Navigation 6.x
- **State Management:** React Context + Hooks
- **Styling:** StyleSheet API + theme constants
- **Notifications:** expo-notifications
- **Date/Time:** DateTimePicker, date-fns
- **Storage:** AsyncStorage (local preferences)

### Key Architectural Decisions

**1. Three-Layer Architecture**
- **UI Layer** (Screens/Components) → Display and user interaction
- **Service Layer** (Services/Hooks) → Business logic
- **Data Layer** (Repositories) → Database access

**2. Separation of Concerns**
- Keep Supabase queries in Repository classes
- Business logic in Service classes
- UI components remain pure and testable
- Use Context for global state (auth, check-in status)

**3. Error Handling Strategy**
- All async functions wrapped in try-catch
- User-friendly error messages
- Network error detection and handling
- Graceful degradation when offline

**4. Notification Strategy**
- Local notifications only for MVP
- Schedule daily reminder at user's configured time
- Use notification actions for quick check-in
- Critical alerts for missed check-ins

**5. Security Considerations**
- Environment variables for sensitive data (never committed)
- Supabase RLS for data access control
- Validate all user input
- Secure token storage (handled by Supabase SDK)

---

## Database Schema (Existing - Reuse)

### Tables to Use

**users**
- id (uuid, FK to auth.users)
- name (text)
- email (text)
- check_in_time (time) - Daily check-in reminder time
- streak (integer) - Current check-in streak
- vacation_mode (boolean) - Pause check-ins
- last_check_in (timestamptz) - NEED TO ADD: Track last successful check-in
- last_device_activity (timestamptz) - NEED TO ADD: Track last device activity
- last_app_open (timestamptz) - NEED TO ADD: Track last app open
- escalation_status (text) - NEED TO ADD: 'none', 'warning', 'critical', 'escalated'
- RLS: Users can only access their own row

**trusted_contacts**
- id (uuid, PK)
- user_id (uuid, FK to users)
- name (text)
- phone (text) - E.164 format
- relationship (text)
- active (boolean) - Contact enabled/disabled
- RLS: Users can only access their own contacts

**check_ins**
- id (uuid, PK)
- user_id (uuid, FK to users)
- timestamp (timestamptz)
- method (text) - 'manual', 'notification', 'screenTime'
- streak (integer) - Streak at time of check-in
- RLS: Users can only access their own check-ins

**escalation_events**
- id (uuid, PK)
- user_id (uuid, FK to users)
- triggered_at (timestamptz)
- trigger_reason (text) - 'missed_checkin', 'inactivity', 'missed_checkin_and_inactivity'
- confidence_level (text) - NEED TO ADD: 'high', 'medium', 'low'
- device_activity_detected (boolean) - NEED TO ADD: Was device active?
- app_activity_detected (boolean) - NEED TO ADD: Was app opened recently?
- resolved_at (timestamptz, nullable)
- contacts_notified (integer) - Count of contacts notified
- RLS: Users can only access their own escalations

### Database Migrations
- Use existing migrations from iOS project as base
- **NEED TO CREATE:** New migration for multi-layered detection fields:
  - Add to users: last_check_in, last_device_activity, last_app_open, escalation_status
  - Add to escalation_events: confidence_level, device_activity_detected, app_activity_detected
- Already includes RLS policies (verify they work for new fields)

---

## Verification & Testing Plan

### End-to-End Testing Scenarios

**Scenario 1: New User Sign Up & First Check-In**
1. Open app → See onboarding screens
2. Complete onboarding
3. Sign up with email/password
4. Grant notification permission
5. See home screen with check-in button
6. Tap check-in button → See success state
7. Verify streak shows "1"
8. Verify check-in recorded in database

**Scenario 2: Add Trusted Contacts**
1. Navigate to Contacts tab
2. Tap "Add Contact" button
3. Fill in name, phone, relationship
4. Save contact
5. Verify contact appears in list
6. Add 4 more contacts (total 5)
7. Verify "Add Contact" is disabled after 5
8. Delete one contact
9. Verify can add another contact

**Scenario 3: Daily Check-In Flow**
1. User checks in today → streak = 1
2. Advance time to next day
3. Receive notification at configured time
4. Tap notification → app opens
5. Check in via notification action
6. Verify streak = 2
7. Verify check-in method = "notification"

**Scenario 4: Missed Check-In Escalation**
1. User has checked in (streak = 5)
2. User misses check-in for 24 hours
3. Background task detects missed check-in
4. Escalation event created
5. Contacts marked for notification (logged)
6. User checks in late
7. Escalation resolved
8. Verify escalation_event has resolved_at timestamp

**Scenario 5: Vacation Mode**
1. User enables vacation mode
2. Check-in button shows "vacation mode" state
3. Check-in button is disabled
4. No notifications scheduled
5. User disables vacation mode
6. Check-in button active again
7. Notifications rescheduled

**Scenario 6: Settings Configuration**
1. Navigate to Settings tab
2. Change check-in time to 9:00 AM
3. Verify notification rescheduled
4. Toggle notifications off
5. Verify no scheduled notifications
6. Toggle notifications on
7. Update user profile name
8. Logout
9. Verify returned to login screen

**Scenario 7: Elderly User Behavior (CRITICAL TEST)**
1. Adult child sets up app on parent's phone
2. Parent uses phone normally (calls, texts) but NEVER opens SafeHaven
3. Verify device activity is tracked
4. Verify home screen shows "You're safe - phone active X minutes ago"
5. Verify NO escalation triggered (even without check-in)
6. After 7 days of no app opens but regular phone use:
   - Verify parent is NOT marked as at-risk
   - Verify NO alerts sent to family
7. Simulate parent's phone dying (no device activity 24+ hours)
8. Verify escalation IS triggered
9. Verify family is notified (via database log in MVP)

**Scenario 8: Check-In Optional Test**
1. Parent checks in via app → streak increments, nice!
2. Next day, parent forgets to check in
3. But parent makes phone call at 2pm
4. Verify device activity detected
5. Verify home screen shows "You're safe"
6. Verify NO escalation
7. Verify only a gentle reminder notification (optional)

### Manual Testing Checklist

**Authentication**
- [ ] Sign up with email/password works
- [ ] Login with existing account works
- [ ] Password reset flow works
- [ ] Sign in with Apple works (iOS)
- [ ] Session persists across app restarts
- [ ] Logout clears session

**Check-In**
- [ ] Check-in button tap increments streak
- [ ] Streak displays correctly
- [ ] Can only check in once per day
- [ ] Check-in resets at configured time
- [ ] Vacation mode disables check-in
- [ ] Check-in status persists across app restarts

**Contacts**
- [ ] Can add up to 5 contacts
- [ ] Phone number validation works
- [ ] Can edit existing contacts
- [ ] Can delete contacts
- [ ] Cannot add more than 5 contacts
- [ ] Empty state shows when no contacts

**Settings**
- [ ] Check-in time picker works
- [ ] Notification toggle works
- [ ] Profile updates save
- [ ] Logout works

**Notifications**
- [ ] Daily reminder fires at configured time
- [ ] Notification action check-in works
- [ ] Notifications can be disabled
- [ ] Changing time reschedules notifications

**Escalation**
- [ ] Missed check-in detected after 24 hours
- [ ] Escalation event created
- [ ] Contacts marked for notification
- [ ] Late check-in resolves escalation
- [ ] Vacation mode prevents escalation

**Cross-Platform**
- [ ] App works on iOS
- [ ] App works on Android
- [ ] UI adapts to different screen sizes
- [ ] Platform-specific features work (Sign in with Apple on iOS only)

### Performance Testing
- [ ] App launches in < 3 seconds
- [ ] Check-in button responds instantly
- [ ] List scrolling is smooth (contacts)
- [ ] No memory leaks during navigation
- [ ] Background tasks don't drain battery excessively

---

## Risk Assessment

### Technical Risks

**Risk 1: Background Notification Scheduling**
- **Impact:** High - Core feature
- **Mitigation:** Use expo-notifications which handles background scheduling. Test thoroughly on both platforms. Have fallback detection logic.

**Risk 2: Missed Check-In Detection Reliability**
- **Impact:** High - Core feature
- **Mitigation:** Use multiple detection methods (background fetch, app open). Log all attempts. Test extensively with different scenarios.

**Risk 3: Expo vs Bare React Native**
- **Impact:** Medium - Future limitations
- **Mitigation:** Expo covers all MVP needs. Can eject to bare workflow later if needed. Document any Expo limitations discovered.

**Risk 4: Cross-Platform Differences**
- **Impact:** Medium - User experience
- **Mitigation:** Test on both iOS and Android throughout development. Use Platform.select() for platform-specific code. Document differences.

**Risk 5: Supabase RLS Policies**
- **Impact:** Medium - Security
- **Mitigation:** Test RLS policies thoroughly. Verify users can only access their own data. Review policies before production.

---

## Development Workflow

### Using PSB Workflow 1 (General Development)

1. **Start in Plan Mode** for each feature
   ```bash
   claude --permission-mode plan
   ```

2. **Give Clear, Specific Prompts**
   - "Implement authentication flow with email/password and Sign in with Apple. Use Supabase Auth. Create AuthContext, AuthService, login screen, and signup screen. Follow React Native best practices."

3. **Review Plan Before Implementation**
   - Check Claude understands requirements
   - Verify approach matches architecture
   - Request clarifications if needed

4. **Switch to Implementation**
   - Toggle to auto-accept mode (Shift+Tab)
   - Claude implements feature

5. **Test Thoroughly**
   - Manual testing on iOS and Android
   - Fix any issues
   - Verify edge cases

6. **Update Documentation**
   - Use `#` command for regression prevention
   - Update CLAUDE.md with new patterns learned
   - Commit with good message

### Using Custom Slash Commands

Create these in `.claude/config.json`:

```json
{
  "commands": {
    "/test-app": "Start the Expo development server and open in iOS simulator",
    "/lint": "Run ESLint and fix auto-fixable issues",
    "/commit-feature": "git add . && git commit -m 'feat: $1' && git push",
    "/new-screen": "Create a new screen component in src/screens/ named $1Screen.tsx with navigation setup",
    "/new-service": "Create a new service in src/services/ named $1Service.ts with TypeScript types",
    "/update-docs": "Update CLAUDE.md and PROJECT-SPEC.md to reflect recent changes"
  }
}
```

---

## Success Metrics

### Definition of Done for MVP

**Functionality**
- [ ] Users can sign up and log in
- [ ] Users can check in daily
- [ ] Streak counter works correctly
- [ ] Users can add/edit/delete trusted contacts (max 5)
- [ ] Daily notifications work
- [ ] Missed check-in detection works
- [ ] Basic escalation triggers after 24 hours
- [ ] Vacation mode works
- [ ] Settings can be configured
- [ ] App works on iOS and Android

**Quality**
- [ ] No critical bugs
- [ ] TypeScript strict mode passes
- [ ] All async functions have error handling
- [ ] User-friendly error messages
- [ ] Smooth animations and transitions
- [ ] Responsive on different screen sizes
- [ ] Proper loading states

**Documentation**
- [ ] PROJECT-SPEC.md complete
- [ ] CLAUDE.md updated with patterns
- [ ] .clinerules followed
- [ ] Code comments for complex logic
- [ ] README with setup instructions

**Production Readiness**
- [ ] Environment variables configured
- [ ] App icons and splash screens set
- [ ] EAS Build configured
- [ ] Ready for TestFlight/Internal Testing

---

## Next Steps After Plan Approval

1. **Create New Project Directory**
   - Initialize React Native/Expo project
   - Set up git and GitHub

2. **Complete Phase 2 Setup**
   - Create PROJECT-SPEC.md
   - Create CLAUDE.md
   - Create .clinerules
   - Install dependencies
   - Configure Supabase

3. **Begin Phase 3 Implementation**
   - Start with Milestone 1 (Foundation & Authentication)
   - Use Workflow 1 (General Development)
   - Test each milestone before moving to next
   - Update documentation as we go

4. **Regular Check-Ins**
   - Review progress after each milestone
   - Update plan if requirements change
   - Use `#` command for regression prevention
   - Keep CLAUDE.md current

---

## Estimated Timeline (Updated After Opus Review)

- **Phase 1 (Planning):** Complete ✅ (with Opus architectural review)
- **Phase 2 (Setup):** 0.5-1 day
- **Phase 3 (Implementation - iOS Only for MVP):**
  - Milestone 1 (Foundation, Auth, Role System): 3 days
  - Milestone 2 (Monitored Person UI): 2 days
  - Milestone 3 (Monitor Dashboard UI): 3 days
  - Milestone 4 (Settings): 1 day
  - Milestone 5 (Push Notifications): 2-3 days
  - **Milestone 6 (iOS Native Module for DeviceActivity): 2-3 days** ← NEW, CRITICAL
  - Milestone 7 (Escalation System): 3-4 days (CRITICAL - Safety feature)
  - Milestone 8 (Onboarding & Polish): 2-3 days
  - Milestone 9 (Testing & Deployment): 3 days

**Total Estimated Time: 21-27 days of focused development**

**Critical Notes:**
- Milestone 6 (Native Module) must be completed before Milestone 7 (Escalation)
- Milestone 7 requires extensive real-world testing (24-hour waits)
- Testing requires physical iOS devices (DeviceActivity doesn't work in simulator)
- Per Opus: This is more realistic than original 15-21 day estimate

**Assumptions:**
- Focused work following PSB methodology
- Access to physical iOS devices for testing
- iOS-only launch (Android deferred to Phase 2 post-MVP)

---

## Critical Design Notes

### User-First Design Philosophy

**Who We're Building For:**

1. **Primary User: Elderly parent (65-85+)**
   - Not tech-savvy, barely uses phone
   - May not remember to open app
   - Needs passive monitoring, not active participation
   - Lives alone, at risk

2. **Secondary User: Adult child (30-60)**
   - Sets up app on parent's phone
   - Pays for service
   - Wants peace of mind
   - Is the trusted contact
   - Tech-savvy

**Product Design Implications:**

- **Onboarding:** Designed for adult child setting up parent's account
- **Home Screen:** Extremely simple, show device activity status prominently
- **Check-In:** Optional habit building, NOT required for safety
- **Monitoring:** Device activity is PRIMARY signal, not backup
- **Notifications:** Only send to family if 24+ hours of NO phone activity

### Check-In Detection System Philosophy (UPDATED)

**Core Principle:** Device activity is the REAL safety signal. Elderly users won't reliably check in via app.

**Reordered Priority:**
1. **Primary Signal:** Device activity (ANY phone usage = they're alive)
2. **Secondary Signal:** Check-in status (nice bonus, habit building)
3. **Tertiary Signal:** App opens (engagement data)

**Decision Tree (SIMPLIFIED):**
- Device active in last 24 hours → User is SAFE (even if no check-in)
- No device activity 24+ hours → CRITICAL EMERGENCY (escalate immediately)
- No check-in but device active → Send gentle reminder (do NOT escalate)

**Key Insight:**
An 80-year-old using their phone to call their friend, text their grandkid, or check the weather is SAFE, even if they forgot about SafeHaven. That's what we monitor.

**False Alarm Prevention:**
- Vacation mode is absolute override (never escalate)
- Device activity detection must be reliable
- Network issues should not trigger escalation
- Race conditions (user checking in while detection runs) must be handled
- Log everything for debugging and audit trail

**Testing is Critical:**
- Simulate elderly user behavior (infrequent app use, device charging, etc.)
- Test: Parent uses phone for calls but never opens app (should NOT escalate)
- Test: Parent's phone dies completely (should escalate after 24 hours)
- Test: Parent goes on vacation, enables vacation mode (no escalation)
- Test on real devices with real background task limitations
- Measure false positive rate and tune accordingly
- Get feedback from actual elderly users and their families

**MVP Simplification:**
- For MVP, log escalations to database but don't send SMS/push to contacts
- Focus on proving the device activity detection works correctly
- Focus on proving the escalation logic avoids false alarms
- Add actual contact notification in Version 2 after detection is proven reliable
- This lets us tune the algorithm safely without bothering real families

---

---

## Summary of Major Design Decisions

### 1. TWO USER TYPES = TWO APP EXPERIENCES
- **"Monitored" users** (elderly): Ultra-simple one-screen app with just check-in button
- **"Monitor" users** (adult children): Dashboard showing status cards of people they watch
- Single codebase, role-based routing on login

### 2. MANY-TO-MANY RELATIONSHIP
- One monitor can watch multiple people (mom, dad, aunt)
- One monitored person can have multiple monitors (daughter, son, neighbor)
- Database: `monitoring_relationships` table (replaces `trusted_contacts`)

### 3. DEVICE ACTIVITY IS PRIMARY SAFETY SIGNAL
- ANY phone usage (calls, texts, other apps) = person is safe
- Check-in via app is SECONDARY (nice habit, not required for safety)
- Escalate ONLY when no device activity for 24+ hours
- Elderly users won't reliably open app - that's okay!

### 4. PUSH NOTIFICATIONS ONLY (NO SMS)
- Monitored person: Gentle daily check-in reminders
- Monitors: Critical alerts when someone at risk (18hr warning, 24hr escalation)
- All notifications via Expo Push API

### 5. ONBOARDING FOR ADULT CHILD
- Child sets up account on parent's phone
- Child links their monitor account to parent's monitored account
- Parent sees simple button, child sees dashboard
- Two setup methods: in-person or remote invitation

### 6. SIMPLIFIED ESCALATION LOGIC
- ✅ Device active in 24 hours = safe
- ⚠️ No activity 18 hours = warning to monitors
- 🚨 No activity 24 hours = critical escalation
- 🛑 Vacation mode = never escalate

---

*This plan follows the PSB (Plan → Setup → Build) System by Afar*
*Plan created: 2026-01-19*
*Last major update: 2026-01-19 - Complete redesign for two-user architecture*
