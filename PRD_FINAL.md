# Product Requirements Document: LPU Campaign Deployment with Automated Retries

**Version:** 1.0  
**Date:** April 28, 2026    
**Prototype:** https://lpu-campaign-deploynew.vercel.app/

---

## 1. OBJECTIVE

Enable campaign managers to deploy outbound calling campaigns with intelligent retry strategies (both scheduled and immediate) to increase lead connection rates. The system provides real-time campaign insights, detailed lead-level journey visibility, and automated retry management through the Mio AI Voice agent platform.

---

## 2. PROBLEM STATEMENT

- **Single-attempt calling results in low connection rates:** Current average connection rate across clients is ~30%, missing significant revenue opportunities
- **Manual retry management is burdensome:** Campaign managers must manually configure and track retries, creating operational overhead
- **Lack of visibility into retry progression:** No structured view of lead journey, call outcomes, and retry effectiveness across the campaign
- **Inconsistent retry automation:** Retry logic not deeply integrated with calling system, causing missed retries and manual workarounds
- **Billing inconsistencies:** METS billing logic unclear during retries, causing confusion and potential revenue leaks

---

## 3. GOAL

**Primary Goal:** Increase lead connection rates by 15-20% through intelligent automated retry strategies

**Secondary Goals:**
- Enable campaign managers to easily configure and deploy retry strategies without technical overhead
- Provide complete visibility into lead lifecycle and call outcomes
- Implement transparent METS-based billing that reflects actual retry behavior
- Scale retry automation through the Automation/Mio Voice Node integration

**Business Impact:**  
More connections → Higher conversion rates → Increased METS consumption → Direct revenue growth

---

## 4. FUNCTIONAL REQUIREMENTS

### 4.1 Campaign Setup & Configuration

| ID | Requirement | Details |
|---|---|---|
| FR1 | Agent Selection | Users must select an active calling agent before campaign creation (Mandatory) |
| FR2 | Campaign Naming | Campaign must have a unique, descriptive name (Mandatory) |
| FR3 | Campaign Type | Support Outbound, Inbound, and Blended campaign types (Mandatory) |
| FR4 | Retry Toggle | Users can enable/disable retries (default: ON for immideate retry) |
| FR5 | Default Retry Type | Default configured as Immediate Retries (2 attempts) |
| FR6 | Retry Type Selection | Support two mutually exclusive retry types: Scheduled and Immediate |
| FR7 | Retry Count | Allow 1-5 retry attempts for both retry types |
| FR8 | Scheduled Retry Timing | Allow configuration of retry times (hours-based or day+time) by default set to 2, first retry after 30 minutes of initial call and second retry next day at 1 PM |
| FR9 | Immediate Retry Config | No timing configuration needed; back-to-back calling |
| FR10 | Allowed Calling Window | Validate retry times against agent's allowed calling hours for day time selection, no need to validate for hours based retry scheduling |
| FR11 | Retry Rescheduling | If retry falls outside allowed window, auto-reschedule to next earliest available and valid slot, in most cases it will be 9 AM |
| FR12 | Test Call | Ability to trigger test call or skip before campaign launch |
| FR13 | Campaign Launch | Start campaign with one click; redirect to Running Campaign UI |

### 4.2 Running Campaign UI & Monitoring

| ID | Requirement | Details |
|---|---|---|
| FR14 | Real-time Metrics | Display 8 key metrics with real-time updates: Total Contacts, Contacts in Queue, Connected, Not Connected, Goal Achieved, Avg Call Duration, Avg Attempts/Contact, Retried |
| FR15 | Clickable Metrics | "Not Connected" and "Retried" cards are clickable, opening detailed modals |
| FR16 | Not Connected Modal | Show breakdown with color-coded items: Not Answered, Busy, Call Failed or any other possible status received from Vendor |
| FR17 | Retry Modal | Show retry status (Exhausted/Scheduled) and call attempts breakdown - Initial, Retry 1, Retry 2, Retry 3 |
| FR18 | Lead Table | Display all leads with columns: Attempts, Last Status, Last Contact Time, Outcome, Inbound Indicator, Retry Attempts Left |
| FR19 | Lead Expandable Rows | Expand lead row to show attempt-level details (Initial Call, Retry 1, Retry 2...Retry N) |
| FR20 | Attempt-Level Details | Per attempt show: Status, Timestamp, Duration |
| FR21 | Call Details Access | Each attempt has link to "View Transcription" modal, change the hyperlink that shows "View Transcription to View Call Details" and open the existing modal which is being used currently in the Timeline under the leads Manager |
| FR22 | Timeline View | Show all call attempts, retry events, and final outcome |

### 4.3 Transcription, recording, summary & Call Analysis

| ID | Requirement | Details |
|---|---|---|
| FR23 | Recording Playback | Current supported UI |
| FR24 | Full Transcript | Current Supported UI |
| FR25 | AI Summary | Show AI-generated call summary for connected calls |

### 4.4 Completed Campaign UI

| ID | Requirement | Details |
|---|---|---|
| FR29 | Final Metrics | Display same 8 metrics as running campaign (adjusted for final state) |
| FR30 | Final Lead Table | Show all leads with final status and outcomes |
| FR32 | Lead Timeline Access | Expandable leads with full attempt history |
| FR33 | Data Export | CSV/Excel export of campaign results |

### 4.5 Automation Node Integration

| ID | Requirement | Details |
|---|---|---|
| FR34 | Node Configuration | Configure block name, agent name, campaign name, campaign type |
| FR35 | Retry Config in Node | Same retry configuration as manual campaigns (Scheduled or Immediate) |
| FR36 | Node Publishing | Publish button to start automation execution |
| FR37 | Node Summary Tooltip | Info icon shows summary of call statuses and lead stages |

### 4.6 Retry Engine

| ID | Requirement | Details |
|---|---|---|
| FR38 | Retry Triggering | Automatically trigger retries for non-connected calls - EXCEPT Voicemails and Calls that were marked failed by the cron due to no pingback receival. |
| FR39 | Retry Stopping | Stop retries immediately upon connection (outbound or inbound) |
| FR40 | Retry Limits | Respect configured retry attempt limits (1-5) |
| FR41 | Inbound Handling | If lead calls back inbound until the campaign is completed, mark as connected and cancel remaining retries, if the lead made an inbound call after all the campaign has ended then there is no need to mark it as connected contact because campaign has already ended, but for the existing job id that handles the inbound calls for that account it will still be going there, no change to that. |
| FR42 | Pingback Failure | If no pingback for >3 days or the day configured in the cron, mark as failed and do NOT retry |

### 4.7 METS Billing

| ID | Requirement | Details |
|---|---|---|
| FR43 | Deduction on Initial | METS deducted once on initial call (Pending state) |
| FR44 | No Retry Deduction | No additional METS deducted during retry attempts |
| FR45 | No Mid-Retry Reversal | METS not reversed during intermediate retries |
| FR46 | Connection Retention | If lead connects (outbound or inbound) until the campaign is running, retain METS |
| FR47 | Duration Charges | Additional METS charged if call duration exceeds base METS allocation |
| FR48 | Exhaustion Reversal | If all retries exhausted with NO connection, reverse METS once |
| FR49 | Voicemail Handling | Voicemail is marked as connected in the system so retries will not be made on voicemail calls |
| FR50 | Billing Transparency | Show METS deduction/reversal status |

---

## 5. USER STORIES & ACCEPTANCE CRITERIA

### 5.1 Campaign Setup Journey

**US1: Select Calling Agent**
```
As a campaign manager,
I want to select which calling agent to use for my campaign,
So that calls are executed via the correct AI configuration.

Acceptance Criteria:
AC1.1 Agent dropdown displays only active agents
AC1.2 Agent selection is mandatory before proceeding
AC1.3 Selected agent configuration is validated
```

**US2: Enter Campaign Details**
```
As a campaign manager,
I want to enter a campaign name and type,
So that I can easily identify and categorize my campaigns.

Acceptance Criteria:
AC2.1 Campaign name field is required
AC2.2 Campaign type dropdown supports: Outbound, Inbound, Blended
AC2.3 Both fields are validated before campaign launch
AC2.4 Cannot proceed to next step if fields are empty
```

**US3: Configure Retry Strategy**
```
As a campaign manager,
I want to choose between Scheduled and Immediate retries,
So that I can optimize my outreach strategy.

Acceptance Criteria:
AC3.1 Default retry type is Immediate Retries (2 attempts)
AC3.2 Can toggle between Scheduled and Immediate retries
AC3.3 Only one retry type can be enabled at a time
AC3.4 When one is enabled, the other is automatically disabled
```

**US4: Set Retry Attempt Count**
```
As a campaign manager,
I want to configure how many retry attempts per lead,
So that I can control outreach frequency.

Acceptance Criteria:
AC4.1 Can select 1-5 retry attempts for both retry types
AC4.2 Default is 2 attempts
AC4.3 Cannot exceed 5 attempts
AC4.4 Count selection updates label in real-time
```

**US5: Configure Scheduled Retry Timing**
```
As a campaign manager,
I want to set specific times for retries,
So that I can reach leads at optimal times.

Acceptance Criteria:
AC5.1 Time picker allows hours-based timing (e.g., 2 hours later)
AC5.2 Day + time selection available (e.g., next day at 1 PM)
AC5.3 Only valid time slots shown (within agent's allowed calling window)
AC5.4 Cannot select times outside allowed calling hours
AC5.5 Invalid times shown as disabled/grayed out
```

**US6: Enforce Calling Window Constraints**
```
As a system,
I want to respect agent's allowed calling hours,
So that calls comply with regulations and agent configuration.

Acceptance Criteria:
AC6.1 Agent's allowed calling window is fetched from API
AC6.2 Retry time picker only shows valid slots
AC6.3 If retry scheduled outside window, auto-reschedule to next valid slot
AC6.4 User cannot manually select invalid times
AC6.5 Error shown if all retries would fall outside allowed window
```

**US7: Review Retry Configuration**
```
As a campaign manager,
I want to see my retry setup clearly before launching,
So that I can confirm the configuration is correct.

Acceptance Criteria:
AC7.1 Retry configuration summary shown before launch
AC7.2 Info callout explains immediate retry behavior
AC7.3 Info callout explains scheduled retry timing
AC7.4 Warning about call queue delays shown for both retry types
```

**US8: Test Campaign Before Launch**
```
As a campaign manager,
I want to test the campaign before deploying to all leads,
So that I can verify agent behavior and call quality.

Acceptance Criteria:
AC8.1 Test Call step available before campaign launch
AC8.2 Can trigger test call with configured agent
AC8.3 Can skip test call to proceed faster
AC8.4 Test calls do not affect campaign metrics or lead data
AC8.5 Test call result shown (success/failure)
```

**US9: Launch Campaign**
```
As a campaign manager,
I want to start the campaign with configured leads,
So that outreach begins immediately.

Acceptance Criteria:
AC9.1 Start Campaign button launches campaign
AC9.2 All leads moved to Pending state
AC9.3 User redirected to Running Campaign UI automatically
AC9.4 Campaign shows "Campaign Running" status
AC9.5 Campaign shows start date/time
```

### 5.2 Running Campaign Monitoring

**US10: View Campaign Metrics**
```
As a campaign manager,
I want to see real-time campaign metrics,
So that I can monitor campaign performance.

Acceptance Criteria:
AC10.1 Eight metric cards visible: Total Contacts, Contacts in Queue, Connected, Not Connected, Goal Achieved, Avg Call Duration, Avg Attempts/Contact, Retried
AC10.2 Metrics update in real-time (refresh every 10 seconds)
AC10.3 Connected shows percentage (e.g., 89 (46%))
AC10.4 Not Connected and Retried show percentage
AC10.5 Metrics color-coded: Indigo, Blue, Orange, Green
```

**US11: View Not Connected Breakdown**
```
As a campaign manager,
I want to click the Not Connected card to see why leads didn't connect,
So that I can understand call failure patterns.

Acceptance Criteria:
AC11.1 Not Connected card is clickable
AC11.2 Modal opens showing breakdown items
AC11.3 Items color-coded: Not Answered (Blue), Busy (Yellow), Call Failed (Red)
AC11.4 Each item shows count and percentage
AC11.5 Modal has 20px rounded corners
AC11.6 Modal shows professional subtle backgrounds
```

**US12: View Retry Status**
```
As a campaign manager,
I want to click the Retried card to see retry progress,
So that I can track how many leads are in retry cycles.

Acceptance Criteria:
AC12.1 Retried card is clickable
AC12.2 Modal opens showing retry status
AC12.3 Shows "Retry Exhausted" and "Next Retry Scheduled" sections
AC12.4 Color-coded: Red for exhausted, Green for scheduled
AC12.5 Shows call attempts breakdown with attempt numbers
```

**US13: View Lead Table**
```
As a campaign manager,
I want to see all leads with their current status,
So that I can track individual lead progress.

Acceptance Criteria:
AC13.1 Lead table displays all leads in campaign
AC13.2 Shows columns: Lead Name, Attempts (e.g., 2/3), Last Status, Last Contact Time, Outcome, Inbound Indicator
AC13.3 Sortable by any column
AC13.4 Filterable by status
AC13.5 Searchable by lead name/phone
```

**US14: Expand Lead Details**
```
As a campaign manager,
I want to expand a lead row to see the complete call journey,
So that I understand every attempt and outcome.

Acceptance Criteria:
AC14.1 Click arrow icon to expand lead row
AC14.2 Shows all attempts: Initial Call, Retry 1, Retry 2...Retry N
AC14.3 Each attempt shows: Status (Connected/Busy/Not Answered/Failed), Timestamp, Duration
AC14.4 Timeline shows attempt sequence
AC14.5 Expandable row shows transcription link per attempt
```

**US15: Access Call Details**
```
As a campaign manager,
I want to click on an attempt to see the call details,
So that I can review what was discussed.

Acceptance Criteria:
AC15.1 Call details link available in expanded lead row
AC15.2 Modal opens with call details
AC15.3 Audio player embedded in modal (not new tab)
AC15.4 Shows full transcript text
AC15.5 Shows AI summary for connected calls

```

### 5.3 Retry Logic

**US16: Trigger Retry for Non-Connected Calls**
```
As a system,
I want to automatically retry non-connected calls,
So that we maximize connection chances.

Acceptance Criteria:
AC16.1 Retries triggered only for: Not Answered, Busy, Call Failed
AC16.2 Retries NOT triggered for: Connected, Inbound, Voicemail , system marked failed calls
AC16.3 Retry scheduled based on configured timing (Scheduled or Immediate)
AC16.4 Retry does not trigger if max attempts reached
AC16.5 Retry job created in backend system
```

**US17: Stop Retries on Connection**
```
As a system,
I want to stop all retries when lead connects,
So that we don't spam leads.

Acceptance Criteria:
AC17.1 When lead answers outbound call, remaining retries cancelled immediately
AC17.2 When lead calls back (inbound) until the campaign is functional, remaining retries cancelled
AC17.3 Lead marked as "Connected" in system
AC17.4 No retry jobs created after connection
```

**US18: Respect Calling Time Windows**
```
As a system,
I want to enforce agent's allowed calling hours,
So that we comply with regulations.

Acceptance Criteria:
AC18.1 Agent's allowed calling window determined from API call
AC18.2 Retries only trigger during allowed hours
AC18.3 If scheduled outside window, reschedule to next valid slot
AC18.4 Retry rescheduled to earliest available next day slot if needed
AC18.5 No calls made outside allowed window
```

**US19: Handle Retry Limits**
```
As a system,
I want to prevent infinite retries,
So that campaigns end properly.

Acceptance Criteria:
AC19.1 Retry attempts capped at configured limit (1-5)
AC19.2 After max attempts reached, no further retries triggered
AC19.3 Lead marked as "Retry Exhausted"
AC19.4 METS reversal triggered if lead never connected
```

**US20: Handle Failed Pingback**
```
As a system,
I want to mark calls as failed if no status update received,
So that we don't retry indefinitely.

Acceptance Criteria:
AC20.1 If no pingback for >3 days or the day configured in the cron, mark call as "Failed"
AC20.2 Do NOT retry  system marked failed calls
```

### 5.4 METS Billing

**US21: Deduct METS on Initial Call**
```
As a system,
I want to deduct METS once per lead on initial call,
So that billing reflects outreach effort.

Acceptance Criteria:
AC21.1 METS deducted when lead moved to Pending (initial call placed)
AC21.2 Deducted exactly once per lead, regardless of call outcome
AC21.3 Amount deducted based on campaign type and agent configuration for CPM.

```

**US22: No METS Deduction for Retries**
```
As a system,
I want to NOT deduct METS for retry attempts,
So that billing reflects actual initial outreach cost.

Acceptance Criteria:
AC22.1 No additional METS deducted for Retry 1, Retry 2, etc.
AC22.2 All retries use same METS deduction from initial call
AC22.3 Transparent display that retries don't incur additional charges
```

**US23: Reverse METS if Lead Never Connects**
```
As a system,
I want to reverse METS only if all retries are exhausted with no connection,
So that billing fairly reflects conversions.

Acceptance Criteria:
AC23.1 METS not reversed during intermediate retries
AC23.2 METS reversed ONLY after all retry attempts exhausted
AC23.3 Reversal triggered when "Retry Exhausted" status set
AC23.4 Reversed ONLY if lead never connected (neither outbound nor inbound)
AC23.5 One-time reversal per lead (not multiple)
```

**US24: Charge Extra for Long Calls**
```
As a system,
I want to charge additional METS for calls longer than base allocation,
So that billing reflects actual engagement.

Acceptance Criteria:
AC24.1 Base METS allocation set per agent configuration
AC24.2 If connected call duration > base, additional METS charged
AC24.3 Additional charge calculated per configured increment
AC24.4 Shown in campaign metrics as "Duration-based charges"
```


### 5.5 Automation Integration

**US26: Configure Automation Retry**
```
As an automation builder,
I want to configure retry strategy in Mio Voice Node,
So that automation uses same retry logic as manual campaigns.

Acceptance Criteria:
AC26.1 Automation node has same retry config options as campaign UI
AC26.2 Can choose Scheduled or Immediate retries
AC26.3 Can set 1-5 retry attempts
AC26.4 Scheduled retries allow time configuration
AC26.5 Immediate retries configured with count only
AC26.6 Mutually exclusive toggles (enable one, disable other)
```

**US27: Publish Automation with Retries**
```
As an automation builder,
I want to publish automation and have retries execute,
So that automation runs with configured retry strategy.

Acceptance Criteria:
AC27.1 Publish button available after config
AC27.2 Publishing starts automation execution
AC27.3 Retry configuration saved and used for all calls
AC27.4 Same retry logic as manual campaigns applied
```

**US28: View Automation Summary**
```
As an automation builder,
I want to see quick summary of call results,
So that I can understand automation performance.

Acceptance Criteria:
AC28.1 Info icon (i) on Mio node shows tooltip
AC28.2 Tooltip shows: Total Leads, Connected, In Retry Cycle, Retry Exhausted, Pending
AC28.3 Updates in real-time
AC28.4 Professional styling matching campaign UI
```

### 5.6 Completed Campaign

**US29: View Final Campaign Summary**
```
As a campaign manager,
I want to see final results when campaign completes,
So that I can evaluate overall performance.

Acceptance Criteria:
AC29.1 Tab 4 shows campaign with "Campaign Completed" status
AC29.2 Displays same 8 metrics as running campaign (final values)
AC29.3 Contacts in Queue shows 0 (no pending)
AC29.4 Connected and Not Connected show final counts and %
AC29.5 Shows completed status with start and end date
```

**US31: Export Campaign Data**
```
As a campaign manager,
I want to download campaign results,
So that I can analyze externally.

Acceptance Criteria:
AC31.1 Download button available on completed campaign
AC31.2 CSV export includes: Lead Name, Phone, Status, Attempts, Duration, Outcome
AC31.3 Excel export format available
AC31.4 All lead data included
AC31.5 File includes campaign metadata (name, dates, etc.)
```

---

## 6. EDGE CASES

| Edge Case | Behavior | Status |
|-----------|----------|--------|
| **Lead connects during retry** | Stop all remaining retries immediately; mark as connected | ✅ Implemented |
| **Lead calls back (inbound)** | Mark as connected; stop retries; don't charge extra METS | ✅ Implemented |
| **Retry scheduled outside calling window** | Auto-reschedule to next valid slot; notify user | ⚠️ Backend verification needed |
| **Call has no pingback for 3+ days** | Mark as failed; do NOT retry; exclude from metrics | ⚠️ Cron job verification needed |
| **Retry count set to 1** | Only one retry attempt; if not connected, mark exhausted | ✅ Supported |
| **Max retries reached, lead still not connected** | Reverse METS; mark as "Retry Exhausted"; stop retries | ⚠️ METS reversal verification needed |
| **Agent calling window = "No calling hours"** | Cannot schedule retries; show error; prevent campaign launch | ⚠️ Validation needed |
| **All leads complete before timeout** | Campaign marked completed; show final metrics | ✅ Implemented |
| **Network failure during test call** | Show error; allow user to retry or skip | ⚠️ Error handling needed |
| **User modifies agent after campaign started** | Not allowed; campaign must use original agent | ⚠️ Needs verification |
| **Immediate retry enabled but no internet** | Retries queued and attempted when connection restored | ⚠️ Queue handling needed |
| **Lead has duplicate phone numbers** | Treat as separate leads; create separate retry chains | ⚠️ Data validation needed |
| **Campaign has 0 leads** | Show "No leads" state; allow campaign to launch but mark completed | ⚠️ Empty state handling needed |

---

## 7. OUT OF SCOPE

| Feature | Reason |
|---------|--------|
| **Campaign Pause** | Not required for MVP; can be added in Phase 2 |
| **Smart Retry Recommendations** | Requires historical data analysis; Phase 2 feature |
| **Lead Segments/Filtering for Retry** | Manual selection sufficient for now |
| **Retry Time Optimization Algorithm** | Manual configuration meets current needs |
| **Multi-language Support** | English only for MVP |
| **Mobile App** | Web-based only for MVP |
| **API for Third-Party Integrations** | Internal use only for MVP |
| **Campaign Templates** | Can be added post-MVP based on usage patterns |
| **Voicemail Handling** | Removed entirely from system per user request |
| **Custom Reporting Dashboards** | Static reports sufficient for MVP |
| **Lead Suppression Lists** | Manual lead list management sufficient |
| **A/B Testing Campaigns** | Sequential campaigns sufficient for MVP |

---

**Next Review Date:** May 5, 2026  
**Owner:** Product Team  
**Last Updated:** April 28, 2026
