# **Frontend Engineer Assignment (No Code, High-Quality UI Focus)**

## **Evaluation Criteria**

* Frontend tech selection and reasoning (framework, state, data fetching)
* UI architecture (routing, component design, reusable patterns)
* API calling strategy (error handling, retries, abort, pagination)
* Browser-level caching + offline-friendly patterns
* Debugging + observability (logging, tracing, error boundaries)
* Security basics on client (token handling, safe downloads, XSS considerations)
* UX quality for async jobs (progress, partial results, resilience)

---

## **Problem 1: Video-to-Notes Platform (Frontend System Design)**

**Goal:** Upload video → job runs async → user sees status + outputs: Summary.md, highlights (timestamps), assets. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

**Your solution must include**

* **Screens:** Upload, Jobs list, Job detail (status/logs), Results (markdown + highlights)
* **UI states:** loading, queued, processing, success, failed, retry, partial output
* **API calling plan:** how you poll/stream job progress (polling vs SSE), abort on navigation
* **Caching:** what to cache in browser (job list, job detail, results), TTL strategy, invalidation
* **Debugging plan:** how you would debug “stuck processing” from frontend side (network logs, correlation id display)

**Your Solution for problem 1:**

### Screens

**1. Upload Screen**
- Video file upload input (drag & drop + file picker)
- File validation (type, size >200MB supported)
- Upload progress bar with percentage and speed
- Submit button (disabled during upload)
- Error message area
- Success redirect to Jobs List after upload

**Purpose:** Allow user to upload large videos and start async processing safely.

---

**2. Jobs List Screen**
- Paginated list of all submitted jobs
- Status badge (Queued / Processing / Success / Failed)
- Created time and video name
- Click row to open Job Detail
- Retry button for failed jobs
- Manual refresh option

**Purpose:** Provide centralized tracking of all video processing jobs.

---

**3. Job Detail Screen**
- Prominent jobId / correlation ID
- Current status with visual indicator
- Step-wise processing logs/timeline
- Progress indicator while running
- Retry and Cancel (if supported)
- Link to Results when ready

**Purpose:** Deep visibility into async job lifecycle and debugging support.

---

**4. Results Screen**
- Render Summary.md using safe markdown viewer
- Highlights list with timestamps (click-to-seek)
- Assets preview (clips and screenshots)
- Download all assets button
- Processed timestamp display

**Purpose:** Enable quick 5–10 minute consumption of long video content.

---

### UI States

The UI clearly communicates the async job lifecycle.

**Loading**
- Shown during video upload
- Progress bar with percentage and speed
- Submit disabled to prevent duplicate uploads

**Queued**
- Status badge: "Queued"
- Inform user job is waiting for processing
- Safe to navigate away

**Processing**
- Animated spinner/progress
- Step-wise logs (e.g., extracting audio, generating summary)
- Live status updates via polling
- JobId visible for debugging

**Success**
- Green success badge
- Link to Results screen enabled
- Show processed timestamp

**Failed**
- Red error state with human-readable message
- Retry button available
- Preserve job metadata

**Retry**
- Show retry-in-progress state
- Disable repeated clicks
- Maintain previous logs for traceability

**Partial Output**
- Show available highlights/screenshots if ready
- Mark remaining items as "processing…"
- Improves perceived performance

---

### API Calling Plan

**Upload Flow**
- User uploads via `POST /videos/upload`
- Backend returns `jobId` and initial status
- Frontend redirects to Jobs List

**Job Progress Tracking (Polling)**
- Poll `GET /jobs/{jobId}` every 5–10 seconds
- Continue while status is `queued` or `processing`
- Stop automatically on `success` or `failed`
- Use exponential backoff on repeated failures

**Abort Handling**
- Use AbortController to cancel polling on navigation/unmount
- Prevent unnecessary network usage and memory leaks

**Error Handling**
- Normalize API errors
- Show user-friendly messages
- Auto-retry failed polling requests (max 3 retries)
- Surface persistent failures in UI

**Pagination**
- Jobs list fetched via paginated API
- Example: `GET /jobs?page=1&limit=10`
- Prevents large payload rendering

---

### Caching Strategy

**What to Cache**
- Jobs list responses
- Individual job detail
- Final results metadata

**Where to Cache**
- In-memory cache via React Query for active session
- localStorage for lightweight job metadata
- IndexedDB optional for larger offline-friendly data

**TTL Strategy**
- Jobs list: ~30 seconds
- Job detail: ~15 seconds during processing
- Results: ~24 hours (mostly immutable)

**Cache Invalidation**
- Invalidate job detail on retry
- Refresh jobs list after new upload
- Invalidate when job transitions to success
- Provide manual refresh option

**Fresh Job Status Handling**
- While polling, bypass stale cache
- Always prefer latest server state for active jobs

---

### Debugging & Observability

**Correlation ID Visibility**
- Display jobId prominently in Job Detail
- Enables backend traceability
- Helps debug “stuck processing”

**Network Monitoring**
- Log API failures with status codes
- Surface meaningful errors to user
- Enable easy inspection via browser DevTools

**Client-side Logging**
- Capture upload failures, polling errors, unexpected states
- Structured logs in development
- Support remote logging integration in production

**Error Boundaries**
- Wrap major UI sections with React Error Boundaries
- Show graceful fallback UI
- Log crash details for investigation

**User Support Hooks**
- Provide optional “Report a Problem”
- Include jobId, last status, and client logs
- Helps support team debug intermittent failures


---

## **Problem 2: LinkedIn Automation Platform (Frontend System Design)**

**Goal:** Connect LinkedIn → persona setup → draft preview → approve → schedule → posting history. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**Your solution must include**

* **Screens:** Connect, Persona editor, Drafts (3 variants), Approval, Scheduler, Post history
* **Form UX:** persona inputs validation, topic input rules, guardrails for scheduling
* **API calling:** draft generation request lifecycle, optimistic UI vs strict confirmation
* **Caching:** drafts caching, schedule list caching, refetch triggers after approval/post
* **Debugging:** how you surface posting failures to user and capture details for support

**Your Solution for problem 2:**

### Screens

**1. Connect Screen**
- “Connect LinkedIn” CTA button
- OAuth authorization flow trigger
- Connection status indicator (Connected / Not Connected)
- Reconnect option if token expired
- Security note about permissions

**Purpose:** Securely connect user’s LinkedIn account before automation.

---

**2. Persona Editor Screen**
- Form inputs:
  - Background / Experience (textarea)
  - Preferred tone (dropdown)
  - Language style (dropdown)
  - Do’s and Don’ts (textarea)
- Save and Update buttons
- Live preview snippet (optional)
- Validation errors inline

**Purpose:** One-time persona setup to maintain consistent voice.

---

**3. Drafts Screen (3 Variants)**
- Topic input + optional context fields
- “Generate Posts” button
- Loading skeleton while generating
- Display 3 draft cards side-by-side:
  - Style label (Insight / Story / Checklist)
  - Character count
  - Copy button
  - Select button
- Regenerate option

**Purpose:** Allow user to compare and choose best variant.

---

**4. Approval Screen**
- Selected draft preview (editable optional)
- Warning: “You are about to publish”
- Two actions:
  - Post Now
  - Schedule Post
- Back to drafts option

**Purpose:** Explicit human approval before publishing.

---

**5. Scheduler Screen**
- Date picker
- Time picker
- Timezone selector (auto-detected default)
- Validation for past time
- Schedule confirmation summary
- Success toast after scheduling

**Purpose:** Reliable future publishing with timezone safety.

---

**6. Post History Screen**
- Paginated table of posts
- Status badges:
  - Draft
  - Approved
  - Scheduled
  - Published
  - Failed
- Posted timestamp
- Retry button for failed posts
- Filter by status

**Purpose:** Visibility and control over automation outcomes.

---

### Form UX

**Persona Validation**
- Required fields: background, tone, language style
- Character limits with live counter
- Prevent empty persona submission
- Save button disabled until valid
- Auto-save draft locally (optional)

**Topic Input Rules**
- Minimum character requirement
- Optional audience/goal fields
- Show helpful placeholder examples
- Prevent generation if topic empty

**Scheduling Guardrails**
- Prevent selecting past date/time
- Show timezone clearly
- Confirmation modal before scheduling
- Warn if LinkedIn not connected
- Prevent duplicate scheduling clicks

---

### API Calling Strategy

**Draft Generation Flow**
- User clicks Generate → `POST /posts/generate`
- Show loading skeleton
- Receive 3 variants in response
- Store in local state

**Optimistic vs Strict Confirmation**
- Use strict confirmation for publishing (safer)
- No optimistic publish for LinkedIn actions
- Show success only after server confirms

**Approval & Publish**
- Approve → `POST /posts/approve`
- Immediate publish → `POST /posts/publish`
- Scheduled publish → `POST /posts/schedule`

**Error Handling**
- Normalize API errors
- Retry generation failures (max 2 retries)
- Show user-friendly error messages
- Disable buttons during in-flight requests

**Abort Handling**
- Cancel draft generation if user navigates away
- Use AbortController for cleanup

---

### Caching Strategy

**What to Cache**
- Generated drafts (short-term)
- Persona configuration
- Post history list

**Where**
- React Query in-memory cache
- localStorage for persona persistence
- IndexedDB optional for offline draft safety

**TTL**
- Drafts: short (~10 minutes)
- Persona: long (~7 days)
- Post history: medium (~60 seconds)

**Refetch Triggers**
- After approval or publish
- After scheduling
- Manual refresh in history screen
- Window focus refetch (optional)

---

### Debugging & Observability

**User-visible Failure States**
- Clear error banner for failed publishing
- Show failure reason if available
- Retry action for failed posts

**Logging**
- Capture generation failures
- Capture publish/schedule errors
- Include requestId/jobId in logs

**Support-friendly Details**
- Show postId in history
- Include timestamp and status transitions
- “Report issue” action with payload

**Network Monitoring**
- Track LinkedIn auth failures
- Detect token expiration
- Surface rate-limit responses gracefully

**Error Boundaries**
- Wrap drafts and scheduler flows
- Prevent full page crash
- Log unexpected UI failures


---

## **Problem 3: DOCX Template → Bulk Generator (Frontend System Design)**

**Goal:** Upload template → review fields → single generate → bulk via CSV → ZIP download + per-row report. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

**Your solution must include**

* **Screens:** Template upload, Field review/editor, Single fill form, Bulk upload, Bulk run status, Report table, Downloads
* **Field UI:** field types (text/number/date), required/default, inline validation
* **Bulk UX:** CSV upload constraints, mapping UI (optional), progress + partial success
* **Browser caching:** template metadata caching, field schema caching, bulk report pagination caching
* **Downloads:** safe download UX (signed URL flow assumed), progress indicator

**Your Solution for problem 3:**

### Screens

**1. Template Upload Screen**
- DOCX upload (drag & drop + file picker)
- File validation (type, size limits)
- Upload progress indicator
- Template name input
- Success redirect to Field Review

**Purpose:** Allow user to upload and register reusable DOCX templates.

---

**2. Field Review / Editor Screen**
- Auto-detected field list from template
- Field type selector (text / number / date / currency)
- Required toggle
- Default value input
- Inline validation errors
- Add/remove field option
- Save template button
- Template preview (read-only)

**Purpose:** Let user confirm and configure dynamic fields accurately.

---

**3. Single Fill Form Screen**
- Dynamic form generated from field schema
- Field-wise validation
- Real-time error messages
- Generate button (disabled until valid)
- Output format selector (DOCX / PDF)
- Generation loading state

**Purpose:** Generate one document quickly with correct validation.

---

**4. Bulk Upload Screen**
- Download sample CSV/Excel template
- CSV/XLSX upload input
- File validation (size, format)
- Optional column mapping UI
- Row count preview
- Start Bulk Run button

**Purpose:** Enable large-scale document generation.

---

**5. Bulk Run Status Screen**
- Job progress bar (% complete)
- Processed rows counter
- Live status updates
- Partial success indicator
- Cancel (if supported)
- Background processing notice

**Purpose:** Provide visibility into long-running bulk jobs.

---

**6. Report Table Screen**
- Paginated per-row results
- Status badge (Success / Failed)
- Error reason column
- Search and filter
- Export report option

**Purpose:** Clear audit trail for bulk generation.

---

**7. Downloads Screen**
- ZIP download button
- Individual file downloads
- File naming preview
- Expiry notice for signed URLs
- Download progress indicator

**Purpose:** Safe and predictable access to generated outputs.

---

### Field UI

**Supported Field Types**
- Text
- Number
- Date
- Currency
- Optional blocks (future-ready)

**Validation Rules**
- Required field enforcement
- Type-specific validation
- Inline error messaging
- Character limits where applicable
- Default value support

**UX Enhancements**
- Auto-focus on first invalid field
- Tooltip help for field meaning
- Consistent formatting preview

---

### Bulk UX

**CSV Upload Constraints**
- Accept CSV/XLSX only
- File size limit enforcement
- Header validation against template fields
- Show row count before processing

**Mapping UI (Optional)**
- Auto-map matching column names
- Manual dropdown mapping fallback
- Highlight unmapped required fields

**Progress & Partial Success**
- Show real-time progress bar
- Display processed vs total rows
- Allow partial success completion
- Provide clear failure reasons per row

---

### Browser Caching Strategy

**What to Cache**
- Template metadata
- Field schema definitions
- Bulk report pages

**Where**
- React Query in-memory cache
- localStorage for recent templates
- IndexedDB optional for large reports

**TTL**
- Template metadata: long (~24 hours)
- Field schema: long (~24 hours)
- Bulk reports: medium (~2 minutes per page)

**Invalidation**
- After template update
- After new bulk run
- Manual refresh option

---

### Downloads (Safe UX)

**Signed URL Flow**
- Request download via secure endpoint
- Receive time-limited signed URL
- Prevent direct public file access

**Download Experience**
- Show download progress
- Disable repeated clicks
- Handle expired links gracefully
- Retry option on failure

**Security Considerations**
- Validate file ownership
- Prevent open redirect risks
- Sanitize file names

---

## **Problem 4: Character-Based Video Series Generator (Frontend System Design)**

**Goal:** Define characters once → create episode from story → view episode package (script/scenes/assets/render plan). [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

**Your solution must include**

* **Screens:** Character library, Relationship editor, Episode creator, Episode detail (scenes), Asset gallery
* **Consistency UX:** show “locked character profile” per episode, version badges
* **API calling:** long-running generation job UI (progress, resume)
* **Caching:** character library caching, episode package caching, asset thumbnails caching

**Your Solution for problem 4:**

### Screens

**1. Character Library Screen**
- Grid/list of saved characters
- Character card with avatar, name, role
- Add Character button
- Edit/Delete actions
- Search and filter
- Version badge on each character

**Purpose:** Central source of truth for reusable characters across episodes.

---

**2. Relationship Editor Screen**
- Visual graph or table of relationships
- Relationship type selector (friend, rival, mentor, etc.)
- Add/Edit/Delete relationship
- Conflict warning if inconsistent rules
- Save changes CTA

**Purpose:** Maintain behavioral consistency between characters.

---

**3. Episode Creator Screen**
- Story prompt textarea
- Character multi-select (from library)
- Episode tone selector (comedy, drama, etc.)
- Duration target (~5 minutes)
- Language selector
- Narration vs dialogue ratio slider
- Generate Episode button
- Validation before submission

**Purpose:** Configure and trigger new episode generation.

---

**4. Episode Detail Screen**
- Episode status badge
- Generation progress timeline
- Scene-by-scene script viewer
- Regenerate scene option
- Resume generation (if interrupted)
- Version badge for episode
- Download package button

**Purpose:** Deep visibility into long-running generation and outputs.

---

**5. Asset Gallery Screen**
- Scene-wise asset grouping
- Character visuals preview
- Background assets
- Audio plan preview
- Thumbnail lazy loading
- Download individual assets

**Purpose:** Easy inspection and reuse of generated assets.

---

### Consistency UX

**Locked Character Profile**
- Once used in an episode, show “Locked” badge
- Prevent accidental personality changes
- Allow versioned edits only
- Show warning if user attempts breaking changes

**Version Badges**
- Version number on characters and episodes
- Show “updated after episode creation” warning
- Allow viewing previous versions
- Maintain audit trail

**Relationship Safety**
- Validate incompatible relationships
- Warn if selected cast violates rules
- Maintain series bible integrity

---

### API Calling Strategy

**Episode Generation Flow**
- User submits → `POST /episodes/generate`
- Backend returns `jobId`
- Frontend redirects to Episode Detail

**Progress Tracking (Polling)**
- Poll `GET /episodes/{jobId}` every 5–10 seconds
- Show step-wise progress
- Stop polling on success/failed
- Use exponential backoff on failures

**Resume Support**
- If user refreshes, resume using jobId
- Persist active job in localStorage
- Restore progress UI on return

**Abort Handling**
- Cancel polling on navigation
- Use AbortController cleanup
- Prevent duplicate generation requests

**Error Handling**
- Normalize API errors
- Retry transient failures
- Show actionable user messages

---

### Caching Strategy

**Character Library Caching**
- Cache character list in React Query
- TTL ~5 minutes
- Invalidate on character edit/add/delete
- Prefetch on app load

**Episode Package Caching**
- Cache completed episode metadata
- TTL long (~24 hours)
- Bypass cache while job is processing
- Manual refresh option

**Asset Thumbnail Caching**
- Browser HTTP cache + lazy loading
- IndexedDB optional for heavy assets
- Use low-res preview first
- Prevent re-downloading unchanged assets

**Stale Data Protection**
- Always refetch active jobs
- Use background refresh on focus
- Show last-updated timestamp

---

### Long-Running Job UX

**Progress Experience**
- Multi-step progress indicator
- Estimated time remaining (if available)
- Background processing friendly
- Allow safe navigation away

**Resilience**
- Auto-reconnect polling on network loss
- Resume after page refresh
- Preserve job state locally

**Failure Handling**
- Clear failure reason
- Retry generation CTA
- Preserve previous logs for debugging

**Observability**
- Show jobId prominently
- Capture client-side errors
- Provide “Report issue” with payload


---

## **Cross-Cutting** 

Answer these in **bullet points** (max 1 page total):

1. **Frontend stack choice**

* EDIT YOUR ANSWER HERE: Framework (Next.js/Vue/etc), state management, router, UI kit, why.
  `<EDIT YOUR ANSWER HERE>`

2. **API layer design**

* Fetch/Axios choice, typed client generation (OpenAPI), error normalization, retries, request dedupe, abort controllers.
  `
  <EDIT YOUR ANSWER HERE>`

3. **Browser caching plan**

* What you cache (GET responses, derived state), where (memory, IndexedDB, localStorage), TTL/invalidation rules.
* How you handle “job status updates” without stale UI.
  `
  <EDIT YOUR ANSWER HERE>`

4. **Debugging & observability**

* Error boundaries, client-side logging approach, correlation id propagation, “report a problem” payload.
* How you would debug: slow uploads, failed downloads, intermittent 500s.
  `
  <EDIT YOUR ANSWER HERE>`

5. **Security basics**

* Token storage approach, CSRF considerations (if cookies), XSS avoidance for markdown rendering, safe file download patterns.
  ` A<EDIT YOUR ANSWER HERE>`
