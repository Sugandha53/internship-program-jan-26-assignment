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

You need to put your solution here.

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

You need to put your solution here.

---

## **Problem 4: Character-Based Video Series Generator (Frontend System Design)**

**Goal:** Define characters once → create episode from story → view episode package (script/scenes/assets/render plan). [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

**Your solution must include**

* **Screens:** Character library, Relationship editor, Episode creator, Episode detail (scenes), Asset gallery
* **Consistency UX:** show “locked character profile” per episode, version badges
* **API calling:** long-running generation job UI (progress, resume)
* **Caching:** character library caching, episode package caching, asset thumbnails caching

**Your Solution for problem 4:**

You need to put your solution here.

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
