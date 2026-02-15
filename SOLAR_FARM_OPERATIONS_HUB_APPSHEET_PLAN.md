# Solar Farm Operations Hub (AppSheet Build Plan)

This document is written for a beginner building a **mobile-first app in AppSheet** for solar farm operations in Australia.

---

## Data Model

Use Google Sheets (or Excel/SQL) as your data source first. Create one table per sheet below.

### 1) Users
**Purpose:** All people who use the app.

| Field | Type | Description |
|---|---|---|
| UserID | Text (Key) | Unique ID (e.g., `USER-0001`) |
| FullName | Name/Text | Worker name |
| Email | Email | Login email (used for security filters) |
| Mobile | Phone | Contact number |
| Role | Enum | `Worker`, `TeamLeader`, `Supervisor`, `Admin` |
| Active | Yes/No | Can this user use the app? |
| DefaultSiteID | Ref -> Sites | Optional default site |
| CreatedAt | DateTime | Record created time |

**Relations:**
- Users 1->many Timesheets
- Users 1->many PrestartChecks
- Users 1->many Reports
- Users 1->many Messages
- Users 1->many TeamMemberships
- Users 1->many TeamDailyReports (as team leader)

---

### 2) Sites
**Purpose:** Site and zone details.

| Field | Type | Description |
|---|---|---|
| SiteID | Text (Key) | Unique site key |
| SiteName | Text | Site name |
| ClientName | Text | Client |
| Address | LongText | Site address |
| Lat | Decimal | Latitude |
| Long | Decimal | Longitude |
| Notes | LongText | Notes |
| Active | Yes/No | Active site |

**Relations:**
- Sites 1->many Timesheets
- Sites 1->many Reports
- Sites 1->many Documents
- Sites 1->many TeamDailyReports
- Sites 1->many DailyTeamBoardLines

---

### 3) Vehicles
**Purpose:** Fleet list for pre-start checks.

| Field | Type | Description |
|---|---|---|
| VehicleID | Text (Key) | Unique vehicle key |
| VehicleCode | Text | Short label (e.g., `UTE-12`) |
| Rego | Text | Registration |
| MakeModel | Text | Make/model |
| SiteID | Ref -> Sites | Where vehicle is usually based |
| QRToken | Text | Token used in QR deep link |
| Active | Yes/No | In service |

**Relations:**
- Vehicles 1->many PrestartChecks

---

### 4) Timesheets
**Purpose:** Start/end shift and hours.

| Field | Type | Description |
|---|---|---|
| TimesheetID | Text (Key) | Unique key |
| UserID | Ref -> Users | Worker |
| WorkDate | Date | Shift date |
| SiteID | Ref -> Sites | Site |
| StartTime | DateTime | Shift start |
| EndTime | DateTime | Shift end |
| BreakMinutes | Number | Break minutes |
| TotalHours | Decimal (Virtual or real) | Calculated hours |
| Status | Enum | `Draft`, `Submitted`, `Approved`, `Rejected` |
| SupervisorComment | LongText | Approval comments |
| CreatedAt | DateTime | Created |
| UpdatedAt | DateTime | Updated |

**Relations:**
- Many Timesheets -> one User
- Many Timesheets -> one Site

**Validation goals:**
- One open shift at a time per user
- No overlap with existing shifts

---

### 5) PrestartChecks
**Purpose:** Vehicle checklist before work.

| Field | Type | Description |
|---|---|---|
| PrestartID | Text (Key) | Unique key |
| VehicleID | Ref -> Vehicles | Vehicle |
| UserID | Ref -> Users | Worker doing check |
| CheckDateTime | DateTime | When completed |
| SiteID | Ref -> Sites | Site |
| TyresOK | Yes/No | Tyres condition |
| LightsOK | Yes/No | Lights |
| ToolsOK | Yes/No | Tools |
| FluidsOK | Yes/No | Fluids |
| DamageFound | Yes/No | Damage found |
| GeneralCondition | Enum | `Good`, `Fair`, `Poor` |
| Comments | LongText | Notes |
| DamagePhoto | Image/File | Photo evidence |
| Signature | Signature | Worker signature |
| OverallStatus | Enum | `OK`, `NeedsRepair` |
| RepairFlag | Yes/No | Computed/maintained flag |

**Relations:**
- Many PrestartChecks -> one Vehicle
- Many PrestartChecks -> one User
- Many PrestartChecks -> one Site

---

### 6) Documents
**Purpose:** Central document library.

| Field | Type | Description |
|---|---|---|
| DocumentID | Text (Key) | Unique key |
| Title | Text | Document title |
| Category | Enum | `Map`, `Procedure`, `Safety`, `ClientDoc`, `SWMS`, `Other` |
| SiteID | Ref -> Sites (optional) | Linked site |
| FileUrl | File | Uploaded PDF/doc |
| Version | Text | Version number |
| EffectiveDate | Date | Start date |
| UploadedBy | Ref -> Users | Uploader |
| UploadedAt | DateTime | Upload time |
| Active | Yes/No | Visible to workers |

**Relations:**
- Many Documents -> one Site (optional)
- Many Documents -> one User (uploader)

---

### 7) Reports
**Purpose:** Incident / issue tracking.

| Field | Type | Description |
|---|---|---|
| ReportID | Text (Key) | Unique key |
| Type | Enum | `Incident`, `NearMiss`, `SafetyHazard`, `QualityIssue`, `EquipmentDamage`, `Other` |
| SiteID | Ref -> Sites | Site |
| ReportedBy | Ref -> Users | Submitter |
| ReportedAt | DateTime | Date/time |
| Severity | Enum | `Low`, `Medium`, `High`, `Critical` |
| Description | LongText | Full details |
| Photo1 | Image/File | Optional photo |
| Photo2 | Image/File | Optional photo |
| Status | Enum | `Open`, `InProgress`, `Closed` |
| AssignedTo | Ref -> Users | Supervisor owner |
| SupervisorComment | LongText | Progress notes |
| ClosedAt | DateTime | Close time |

**Relations:**
- Many Reports -> one Site
- Many Reports -> one User (reported by)
- Many Reports -> one User (assigned supervisor)

---

### 8) Groups
**Purpose:** Messaging channels.

| Field | Type | Description |
|---|---|---|
| GroupID | Text (Key) | Unique key |
| GroupName | Text | Channel name |
| GroupType | Enum | `Announcements`, `Team`, `Zone`, `Maintenance`, `HSE`, `Custom` |
| SiteID | Ref -> Sites (optional) | Site-specific group |
| Active | Yes/No | Enabled |
| CreatedBy | Ref -> Users | Admin/supervisor |
| CreatedAt | DateTime | Created time |

**Relations:**
- Groups 1->many Messages
- Groups many-to-many Users via GroupMembers

---

### 9) GroupMembers
**Purpose:** Which users are in each group.

| Field | Type | Description |
|---|---|---|
| GroupMemberID | Text (Key) | Unique key |
| GroupID | Ref -> Groups | Group |
| UserID | Ref -> Users | Member |
| RoleInGroup | Enum | `Member`, `Moderator`, `Owner` |
| JoinedAt | DateTime | Joined time |
| Active | Yes/No | Still in group |

**Relations:**
- Many GroupMembers -> one Group
- Many GroupMembers -> one User

---

### 10) Messages
**Purpose:** Group chat messages.

| Field | Type | Description |
|---|---|---|
| MessageID | Text (Key) | Unique key |
| GroupID | Ref -> Groups | Channel |
| SenderUserID | Ref -> Users | Sender |
| MessageText | LongText | Text body |
| Attachment | File/Image | Optional file/photo |
| SentAt | DateTime | Timestamp |
| EditedAt | DateTime | Optional |

**Relations:**
- Many Messages -> one Group
- Many Messages -> one User (sender)

**Query support:**
- “All messages in group X” = filter `Messages[GroupID]=X`, order by `SentAt` ascending.

---

### 11) Teams
**Purpose:** Team master list.

| Field | Type | Description |
|---|---|---|
| TeamID | Text (Key) | Unique team key |
| TeamName | Text | Team name/number |
| TeamLeaderUserID | Ref -> Users | Current team leader |
| Active | Yes/No | Active team |

**Relations:**
- Teams 1->many TeamMemberships
- Teams 1->many DailyTeamBoardLines
- Teams 1->many TeamDailyReports

---

### 12) TeamMemberships
**Purpose:** Worker-team membership over time.

| Field | Type | Description |
|---|---|---|
| TeamMembershipID | Text (Key) | Unique key |
| TeamID | Ref -> Teams | Team |
| UserID | Ref -> Users | Worker |
| StartDate | Date | From date |
| EndDate | Date | To date (optional) |
| Active | Yes/No | Current member |

**Relations:**
- Many TeamMemberships -> one Team
- Many TeamMemberships -> one User

---

### 13) DailyTeamBoards
**Purpose:** Board header for a date.

| Field | Type | Description |
|---|---|---|
| BoardID | Text (Key) | Unique key |
| BoardDate | Date | Date for work |
| SiteID | Ref -> Sites (optional) | Optional site-level board |
| Status | Enum | `Draft`, `Published`, `Archived` |
| PublishedBy | Ref -> Users | Supervisor |
| PublishedAt | DateTime | Publish time |
| Notes | LongText | General notes |
| LastUpdatedAt | DateTime | Last update |

**Relations:**
- DailyTeamBoards 1->many DailyTeamBoardLines

---

### 14) DailyTeamBoardLines
**Purpose:** Team allocations within a board.

| Field | Type | Description |
|---|---|---|
| BoardLineID | Text (Key) | Unique key |
| BoardID | Ref -> DailyTeamBoards | Board header |
| TeamID | Ref -> Teams | Team |
| TeamLeaderUserID | Ref -> Users | Leader for this day |
| SiteID | Ref -> Sites | Site |
| Zone | Text | Zone/area |
| Task | Text | Assigned task |
| WorkerListText | LongText (optional) | Optional readable list |

**Relations:**
- Many DailyTeamBoardLines -> one DailyTeamBoard
- Many DailyTeamBoardLines -> one Team
- Many DailyTeamBoardLines -> one Site

**Query support:**
- “Board for today” = `BoardDate = TODAY()` and `Status = Published`.

---

### 15) TeamDailyReports
**Purpose:** Team production report (one per team per day).

| Field | Type | Description |
|---|---|---|
| TeamDailyReportID | Text (Key) | Unique key |
| ReportDate | Date | Work date |
| TeamID | Ref -> Teams | Team |
| TeamLeaderUserID | Ref -> Users | Submitted by |
| SiteID | Ref -> Sites | Site |
| Zone | Text | Zone |
| WorkType | Enum | `TrackerInstall`, `Piling`, `Cabling`, `Stringing`, `Other` |
| QtyTrackers | Number | Count |
| QtyPiles | Number | Count |
| QtyStrings | Number | Count |
| HoursWorked | Decimal | Team hours |
| Blockers | LongText | Issues/blockers |
| Photo1 | Image/File | Optional |
| Photo2 | Image/File | Optional |
| SubmittedAt | DateTime | Submit time |
| UniqueTeamDate | Text | Helper key = `TeamID-YYYYMMDD` |

**Relations:**
- Many TeamDailyReports -> one Team
- Many TeamDailyReports -> one Site
- Many TeamDailyReports -> one User (team leader)

**Uniqueness rule:**
- Enforce one report per team/day with `UniqueTeamDate` and mark it as key or unique validated field.

---

## Screens & Navigation

Build bottom navigation with 5 main tabs: **Home, Tasks, Board, Docs, Chat**.

### Home / Dashboard
- **Roles:** all users (content changes by role)
- **Shows:** welcome name, today’s site/team, quick status cards
- **Buttons:** Start/End Shift, Vehicle Pre-start, Report Issue, Team Board, Documents, Messages
- **Tables:** reads Users, Timesheets, DailyTeamBoards, Reports

### Timesheet Form
- **Roles:** Worker, TeamLeader
- **Shows:** start/end shift fields, site, breaks
- **Actions:** Save start, Save end, Submit
- **Tables:** writes Timesheets

### Timesheet History
- **Roles:** Worker sees own only; Supervisor sees all
- **Shows:** list by date/site/status
- **Actions:** filter, open detail
- **Tables:** reads Timesheets

### Approve Timesheets (Supervisor)
- **Roles:** Supervisor/Admin
- **Shows:** pending entries with filters
- **Actions:** approve/reject + comments
- **Tables:** updates Timesheets

### Vehicle Pre-start Form
- **Roles:** Worker, TeamLeader
- **Shows:** checklist + photo + signature
- **Actions:** submit check
- **Tables:** writes PrestartChecks

### Pre-start History / Vehicle Status
- **Roles:** Supervisor/Admin
- **Shows:** all checks, “NeedsRepair” list
- **Actions:** filter by date/vehicle/site
- **Tables:** reads PrestartChecks, Vehicles

### Sites List
- **Roles:** all
- **Shows:** site cards with address/client
- **Actions:** open details/map
- **Tables:** reads Sites

### Site Details
- **Roles:** all
- **Shows:** metadata + related docs
- **Actions:** open document, navigate map
- **Tables:** reads Sites, Documents

### Map View
- **Roles:** all
- **Shows:** pin map of sites
- **Actions:** tap pin to open site
- **Tables:** reads Sites

### Documents List
- **Roles:** all
- **Shows:** searchable docs by category/site/version
- **Actions:** open file, filter/search
- **Tables:** reads Documents

### Document Details
- **Roles:** all
- **Shows:** file, version, date
- **Actions:** open/download
- **Tables:** reads Documents

### Incident / Issue Form
- **Roles:** Worker, TeamLeader, Supervisor
- **Shows:** type, severity, site, description, photos
- **Actions:** submit report
- **Tables:** writes Reports

### Incident List / Detail
- **Roles:** Worker sees own; Supervisor sees all
- **Shows:** status board (Open/InProgress/Closed)
- **Actions:** update status, assign supervisor, add comments
- **Tables:** reads/writes Reports

### Groups List (Messaging)
- **Roles:** all (only groups they belong to)
- **Shows:** group names + latest message preview
- **Actions:** open group chat
- **Tables:** reads Groups, GroupMembers, Messages

### Group Chat Screen
- **Roles:** group members
- **Shows:** message feed + input box
- **Actions:** send message/photo/file
- **Tables:** reads/writes Messages

### Team Board (Today/Tomorrow)
- **Roles:** all
- **Shows:** mobile cards: Team, leader, workers, zone, task
- **Actions:** switch today/tomorrow/history
- **Tables:** reads DailyTeamBoards, DailyTeamBoardLines, Teams

### Board Management
- **Roles:** Supervisor/Admin
- **Shows:** board draft editor
- **Actions:** create date board, add team lines, publish/update
- **Tables:** writes DailyTeamBoards, DailyTeamBoardLines

### Team Daily Report Form
- **Roles:** TeamLeader (and Supervisor optional)
- **Shows:** quantities, hours, blockers, photos
- **Actions:** submit one report per day
- **Tables:** writes TeamDailyReports

### Team Daily Reports Summary
- **Roles:** Supervisor/Admin
- **Shows:** list + totals by day/week/team/site
- **Actions:** filter, export
- **Tables:** reads TeamDailyReports

### Admin Settings (optional)
- **Roles:** Admin
- **Shows:** users, teams, vehicles, groups
- **Actions:** manage master data
- **Tables:** Users, Teams, TeamMemberships, Vehicles, Groups, GroupMembers

---

## Daily Flows (Simple)

### Worker flow
1. Open app -> check **Team Board** for today.
2. Reach site -> tap **Start Shift**.
3. Scan vehicle QR -> complete **Pre-start**.
4. During day -> if issue happens, submit **Report Issue** with photo.
5. Read updates in **Messages**.
6. End of day -> tap **End Shift**.

### Team leader flow
1. Same as worker + submit **Team Daily Report** once per day.
2. Confirm team quantities and blockers.

### Supervisor flow
1. End of day -> build/publish **Tomorrow Board**.
2. Morning -> review incidents and pre-start issues.
3. Approve/reject **Timesheets**.
4. Monitor production totals from team reports.

---

## Workflows & Automations

## A) Starting and ending a shift (AppSheet)

### Step 1: Create Timesheets form
- Data -> Columns -> `Timesheets`.
- Set:
  - `StartTime` type: DateTime
  - `EndTime` type: DateTime
  - `BreakMinutes` type: Number (default 0)
  - `UserID` as Ref -> Users
  - `SiteID` as Ref -> Sites

### Step 2: Auto-fill worker and date
- In `UserID` **Initial value**:
```appsheet
ANY(SELECT(Users[UserID], [Email] = USEREMAIL()))
```
- In `WorkDate` **Initial value**:
```appsheet
TODAY()
```

### Step 3: Start shift action
Create an action on Users or Home row: “Start Shift”.
- Behavior -> Actions -> New action -> go to another view in this app.
- Target:
```appsheet
LINKTOFORM(
  "Timesheet_Form",
  "UserID", ANY(SELECT(Users[UserID], [Email]=USEREMAIL())),
  "WorkDate", TODAY(),
  "StartTime", NOW()
)
```

### Step 4: End shift action
Create slice `OpenMyTimesheets` with condition:
```appsheet
AND([UserID] = ANY(SELECT(Users[UserID], [Email]=USEREMAIL())), ISBLANK([EndTime]))
```
Create action “End Shift” on Timesheets row:
- Set value of `EndTime` to:
```appsheet
NOW()
```
- Optionally set `Status` to `Submitted`.

### Step 5: Total hours formula
In `TotalHours` (App formula):
```appsheet
IF(
  ISNOTBLANK([EndTime]),
  ROUND((([EndTime] - [StartTime]) * 24) - ([BreakMinutes] / 60.0), 2),
  ""
)
```

### Step 6: Prevent overlapping shifts
Use `Valid_If` on `StartTime`:
```appsheet
COUNT(
  SELECT(
    Timesheets[TimesheetID],
    AND(
      [UserID] = [_THISROW].[UserID],
      ISBLANK([EndTime])
    )
  )
) = 0
```
Also add `Valid_If` on new row key or form save:
```appsheet
COUNT(
  SELECT(
    Timesheets[TimesheetID],
    AND(
      [UserID] = [_THISROW].[UserID],
      [WorkDate] = [_THISROW].[WorkDate],
      ISBLANK([EndTime])
    )
  )
) <= 1
```

### Step 7: Supervisor approval workflow
- Add enum `Status` values: Draft/Submitted/Approved/Rejected.
- Create supervisor actions “Approve” and “Reject”.
- Create slice `PendingTimesheets` where `[Status]="Submitted"`.

---

## B) Vehicle pre-start via QR

### Step 1: Create QR token per vehicle
- In Vehicles table, fill `QRToken` (unique random text).
- QR content format:
```text
https://www.appsheet.com/start/<YourAppID>#control=Prestart_Form&VehicleToken=<QRToken>
```
(Use your real App ID and form view name.)

### Step 2: Capture token in app
Add column in PrestartChecks: `ScannedToken` (Text).
Set form deep link to prefill token.

Alternative safer approach (recommended):
- Use LINKTOFORM generated by AppSheet for each vehicle row and convert that URL to QR.
- Action on Vehicles:
```appsheet
LINKTOFORM("Prestart_Form", "VehicleID", [VehicleID], "CheckDateTime", NOW())
```
Generate QR for this link externally once.

### Step 3: Save pre-start data
- `VehicleID` as Ref.
- `UserID` initial value from USEREMAIL() lookup.
- `CheckDateTime` initial value `NOW()`.
- Signature column type = Signature.

### Step 4: Auto flag Needs Repair
In `OverallStatus` app formula:
```appsheet
IF(
  OR(
    NOT([TyresOK]),
    NOT([LightsOK]),
    NOT([ToolsOK]),
    NOT([FluidsOK]),
    [DamageFound] = TRUE,
    [GeneralCondition] = "Poor"
  ),
  "NeedsRepair",
  "OK"
)
```
Create slice `VehiclesNeedingAttention` on latest checks where `OverallStatus="NeedsRepair"`.

---

## C) Incident / issue reporting

### Build form
- Table: Reports
- Required fields: Type, SiteID, Severity, Description
- Auto fields:
  - `ReportedBy` initial value via USEREMAIL lookup
  - `ReportedAt` initial value `NOW()`
  - `Status` initial value `"Open"`

### Photos
- Set `Photo1`/`Photo2` column type to Image.
- Turn on “Store content for offline use” if crews lose signal often.

### Supervisor handling
- Create `OpenReports` slice where `[Status]<>"Closed"`.
- Add actions:
  - Assign to me
  - Move to InProgress
  - Close report

### High severity notification (optional)
Create Bot:
- Event: Adds only on Reports.
- Condition:
```appsheet
IN([Severity], {"High", "Critical"})
```
- Task: Send push/email to supervisor list.

---

## D) Messaging & groups

### Structure
Use `Groups`, `GroupMembers`, `Messages`.
- Security filter on Messages:
```appsheet
IN(
  [GroupID],
  SELECT(GroupMembers[GroupID], [UserID] = ANY(SELECT(Users[UserID], [Email]=USEREMAIL())))
)
```

### Group list view
- Base table: Groups
- Slice: only groups where current user is active member.

### Chat view
- Ref from Groups to Messages gives inline child list.
- Sort Messages by `SentAt` ascending.
- Add action/button “New Message” with LINKTOFORM prefilled GroupID.

Target formula:
```appsheet
LINKTOFORM(
  "Message_Form",
  "GroupID", [GroupID],
  "SenderUserID", ANY(SELECT(Users[UserID], [Email]=USEREMAIL())),
  "SentAt", NOW()
)
```

### Notifications
If AppSheet push is enabled:
- Bot on new Messages
- Condition: exclude sender
- Send notification to group members.

---

## E) Daily team board / roster

### Data entry design
- `DailyTeamBoards` = one header per date.
- `DailyTeamBoardLines` = one row per team assignment.

### Supervisor process
1. Create board header (date, status Draft).
2. Add board lines for each team: team, leader, site, zone, task.
3. Publish (set status to `Published`, set PublishedAt/By).

### Worker view
Create slices:
- `TodayBoardLines`:
```appsheet
[BoardID].[BoardDate] = TODAY()
```
- `TomorrowBoardLines`:
```appsheet
[BoardID].[BoardDate] = (TODAY()+1)
```
Use Deck/Card view for easy mobile reading.

### Publish/update notifications
Bot on DailyTeamBoards update:
- Condition:
```appsheet
[Status] = "Published"
```
- Send push to all active users (or only site users).

---

## F) Team daily production reports

### One report per team per day (important)
In `UniqueTeamDate` Initial/App formula:
```appsheet
CONCATENATE([TeamID], "-", TEXT([ReportDate], "YYYYMMDD"))
```
Set `Valid_If`:
```appsheet
COUNT(
  SELECT(
    TeamDailyReports[TeamDailyReportID],
    [UniqueTeamDate] = [_THISROW].[UniqueTeamDate]
  )
) = 0
```
(Or make `UniqueTeamDate` the key column.)

### Team leader submission
- Role-based show if TeamLeader or Supervisor.
- Prefill leader and date.
- Required quantities for relevant work type only (optional Show_If).

### Supervisor summary totals
Create virtual columns or dashboard slices.
Example daily trackers total:
```appsheet
SUM(SELECT(TeamDailyReports[QtyTrackers], [ReportDate] = TODAY()))
```
Example weekly piles total:
```appsheet
SUM(
  SELECT(
    TeamDailyReports[QtyPiles],
    AND(
      [ReportDate] >= (TODAY()-6),
      [ReportDate] <= TODAY()
    )
  )
)
```

---

## Step-by-Step Implementation in AppSheet (Beginner Sequence)

1. **Create Google Sheets file** with all tables listed above.
2. **Open AppSheet -> New app -> from Google Sheets**.
3. **Add all tables** and confirm column types (Ref, Enum, Image, Signature, DateTime).
4. **Set keys** (`...ID` columns) and labels (friendly names like FullName, SiteName).
5. **Build roles**:
   - Add Role in Users table.
   - Create slices per role (Worker, TeamLeader, Supervisor).
6. **Create main views**:
   - Home dashboard
   - Timesheet form/history
   - Pre-start form/history
   - Reports form/list
   - Team board
   - Documents
   - Messages
7. **Add quick actions on Home** using deep links (`LINKTOFORM`, `LINKTOVIEW`).
8. **Implement Timesheet logic** (start/end actions, total hours formula, overlap checks).
9. **Implement Pre-start logic** (QR deep link, auto status, NeedsRepair slice).
10. **Implement Incident workflow** (open/in progress/closed + optional high severity bot).
11. **Implement Messaging** (Groups + GroupMembers + Messages + security filter).
12. **Implement Team board** (header + lines + publish bot).
13. **Implement Team daily report** (unique team-date constraint + summary views).
14. **Set security filters** so workers only see their own sensitive data where needed.
15. **Enable offline/sync** for field conditions with poor signal.
16. **Test on phone** (AppSheet app on iOS/Android; browser also works on macOS).
17. **Pilot with one site/team for 1 week**, then expand.

---

## macOS access (what you asked)

Yes, your app can be opened on macOS:
- Use browser: Chrome/Safari -> AppSheet app URL.
- For admins/supervisors, browser is great for larger tables and board planning.
- Workers should mainly use phone app for camera, QR, signature, and offline support.

---

## Build Tips for No-Code Success

- Start with **MVP first**: Timesheets + Pre-start + Reports + Documents.
- Add Messaging and Team Board next.
- Keep forms short and mobile-friendly.
- Use clear names and emojis in button labels.
- Train team leaders first; they help workers adopt the app quickly.
