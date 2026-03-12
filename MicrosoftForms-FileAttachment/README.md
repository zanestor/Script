# Microsoft Forms – File Attachment Workaround
## Recruitment Process Blueprint (Step-by-Step)

> **Goal:** Collect job applications (CV, motivation letter, diplomas, etc.) through Microsoft Forms while securely gathering supporting documents that Forms itself cannot accept as attachments.

---

## Table of Contents

1. [Why Microsoft Forms Cannot Accept Attachments](#why)
2. [Option 1 – Power Automate + SharePoint (Recommended)](#option-1)
3. [Option 2 – Google Forms + Google Drive + Power Automate](#option-2)
4. [Option 3 – Encrypted Email Reply (Simplest)](#option-3)
5. [Comparison Table](#comparison)
6. [Pre-requisites & Licences](#pre-requisites)
7. [Frequently Asked Questions](#faq)

---

<a name="why"></a>
## 1. Why Microsoft Forms Cannot Accept Attachments

Microsoft Forms supports file uploads **only** when:
- The form is **posted inside a Microsoft 365 tenant** (i.e. the respondent must be signed in with a company account in the same tenant), **and**
- The file-upload question is enabled.

For **public recruitment forms** (candidates outside your organisation), the file-upload question is disabled by Microsoft design. The workarounds below solve this gap.

---

<a name="option-1"></a>
## 2. Option 1 – Power Automate + SharePoint (Recommended)

### How It Works

```
Candidate fills form
       │
       ▼
Power Automate trigger: "When a new response is submitted"
       │
       ├─► Create a personal SharePoint folder  (e.g. /Recruitment/2024-Software-Engineer/jane.doe@gmail.com/)
       │
       ├─► Generate a time-limited sharing link  (30-minute expiry, Upload-only, no Delete)
       │
       └─► Send email to candidate with the link
```

The candidate clicks the link within 30 minutes, uploads their files, and the link expires automatically.

---

### Step-by-Step Implementation

#### Step 1 – Create the Microsoft Form

1. Go to [https://forms.microsoft.com](https://forms.microsoft.com) and click **New Form**.
2. Add a **title**, e.g. *"Software Engineer Application – Step 1"*.
3. Add the following questions (use **Required** toggle):
   | # | Question | Type |
   |---|----------|------|
   | 1 | Full Name | Text |
   | 2 | Email Address | Text |
   | 3 | Position applied for | Choice or Text |
   | 4 | Phone number (optional) | Text |
4. Click **Share** → choose **"Anyone can respond"** → copy the form link.
5. **Do NOT** add a file-upload question (it will be greyed out for external respondents).

---

#### Step 2 – Create the SharePoint Document Library

1. Open your SharePoint site (e.g. `https://yourcompany.sharepoint.com/sites/HR`).
2. Click **+ New → Document library**.
3. Name it **`Recruitment`**.
4. Inside the library create a folder for each job opening, e.g. **`2024-Software-Engineer`**.
5. Set the library permissions so that **only HR staff** have access by default:
   - Library settings → Permissions → Stop inheriting permissions → Add HR group.

---

#### Step 3 – Build the Power Automate Flow

1. Go to [https://make.powerautomate.com](https://make.powerautomate.com).
2. Click **+ Create → Automated cloud flow**.
3. Name the flow `[Recruitment] Create Upload Folder and Notify Candidate`.

##### Trigger
- Search for **"Microsoft Forms"** → select **"When a new response is submitted"**.
- **Form Id:** select the form you created in Step 1.

##### Action 1 – Get response details
- Click **+ New step** → search **"Microsoft Forms"** → **"Get response details"**.
- **Form Id:** same form.
- **Response Id:** select `List of response notifications Response Id` (dynamic content).

##### Action 2 – Initialize a variable for the candidate folder path
- **+ New step** → **"Initialize variable"**
- **Name:** `CandidateFolderPath`
- **Type:** String
- **Value:** (use the expression editor — click the `fx` button)
  ```
  concat('/Recruitment/2024-Software-Engineer/', outputs('Get_response_details')?['body/r2'])
  ```
  > `r2` is the internal name for question 2 (Email). Verify the actual field name in the dynamic content panel.

##### Action 3 – Create folder in SharePoint
- **+ New step** → search **"SharePoint"** → **"Create new folder"**.
- **Site Address:** your SharePoint site URL.
- **List or Library:** `Recruitment`.
- **New Folder Name:** use dynamic content → `variables('CandidateFolderPath')`.

##### Action 4 – Create a sharing link with expiry
- **+ New step** → search **"Send an HTTP request to SharePoint"** (available in SharePoint connector).
- **Site Address:** your SharePoint site URL.
- **Method:** `POST`
- **Uri:**
  ```
  _api/web/GetFolderByServerRelativeUrl('@{variables('CandidateFolderPath')}')/ListItemAllFields/ShareLink
  ```
- **Headers:**
  ```json
  {
    "Accept": "application/json;odata=verbose",
    "Content-Type": "application/json;odata=verbose"
  }
  ```
- **Body:**
  ```json
  {
    "request": {
      "createLink": true,
      "settings": {
        "linkKind": 5,
        "expiration": "@{addMinutes(utcNow(), 30)}",
        "role": 2,
        "allowAnonymousAccess": true
      }
    }
  }
  ```
  > `linkKind: 5` = anonymous link. `role: 2` = Edit (upload) without delete. Adjust `linkKind` and `role` to match your tenant's sharing settings.

##### Action 5 – Parse the sharing link response
- **+ New step** → **"Parse JSON"**.
- **Content:** `Body` from the previous HTTP step.
- **Schema** (click *"Generate from sample"* and paste a real response, or use):
  ```json
  {
    "type": "object",
    "properties": {
      "d": {
        "type": "object",
        "properties": {
          "ShareLink": {
            "type": "object",
            "properties": {
              "sharingLinkInfo": {
                "type": "object",
                "properties": {
                  "Url": { "type": "string" }
                }
              }
            }
          }
        }
      }
    }
  }
  ```

##### Action 6 – Send email to the candidate
- **+ New step** → search **"Send an email (V2)"** (Office 365 Outlook connector).
- **To:** dynamic content → candidate email field from the form response.
- **Subject:** `[Action Required] Please upload your documents – expires in 30 minutes`
- **Body:**
  ```
  Dear @{outputs('Get_response_details')?['body/r1']},

  Thank you for applying for the @{outputs('Get_response_details')?['body/r3']} position.

  Please upload the following documents using the secure link below:
    • CV / Résumé
    • Motivation letter
    • Copies of diplomas / certificates

  ⚠️  This link will expire in 30 minutes.

  Upload link: @{body('Parse_JSON')?['d']?['ShareLink']?['sharingLinkInfo']?['Url']}

  If the link has expired, please reply to this email and we will send a new one.

  Best regards,
  HR Team
  ```

##### Action 7 – (Optional) Post a notification to Teams or send an internal alert
- **+ New step** → **"Post a message in a chat or channel"** (Microsoft Teams connector).
- Channel: `#recruitment-new-applications`.
- Message: `New application received from @{outputs('Get_response_details')?['body/r1']} – folder created at @{variables('CandidateFolderPath')}`.

4. Click **Save**, then click **Test → Manually** and submit a test form response to verify.

---

#### Step 4 – Restrict Permissions on the Folder (No Delete)

After the flow creates the folder, restrict what the candidate can do:

In Action 4 above, set `"role": 2` which grants **Edit** permission (can upload, cannot delete library items unless you also send a `MERGE` request to set `ReadOnly = true`).

For a stricter lockdown after upload, add a scheduled flow (trigger: **Recurrence**, every 5 minutes) that:
1. Checks if files have been uploaded to the folder.
2. Removes the sharing link once files are present.

---

#### Step 5 – Organise and Review Applications

All uploaded files land in:
```
SharePoint → Recruitment → 2024-Software-Engineer → jane.doe@gmail.com/
```

HR staff can:
- Open the folder in SharePoint or via Teams' Files tab.
- Add metadata columns (Status: *Shortlisted / Rejected*) to the library.
- Use Power BI connected to the SharePoint list to build a recruitment dashboard.

---

<a name="option-2"></a>
## 3. Option 2 – Google Forms + Google Drive + Power Automate

### How It Works

```
Candidate fills a Google Form (or sends email to a Gmail inbox)
       │
       ├─► Attachments land in Google Drive (auto-saved by Google Forms)
       │
       └─► Power Automate (scheduled or triggered) copies files from Google Drive → SharePoint
```

---

### Step-by-Step Implementation

#### Step 1 – Create a Gmail/Google Forms intake

1. Go to [https://docs.google.com/forms](https://docs.google.com/forms).
2. Create a form with the same fields as Option 1.
3. Add a **File upload** question — Google Forms allows this for anyone (no tenant restriction).
4. Files are automatically saved to a linked Google Drive folder.

---

#### Step 2 – Connect Power Automate to Google Drive

> **Requirement:** A Power Automate plan that includes the Google Drive connector (Microsoft 365 Business Standard or higher, or a standalone Power Automate plan).

1. Go to [https://make.powerautomate.com](https://make.powerautomate.com) → **+ Create → Automated cloud flow**.
2. Name it `[Recruitment] Move Google Drive Files to SharePoint`.

##### Trigger
- Search **"Google Drive"** → **"When a file is created"**.
- **Folder:** select the Google Drive folder linked to your Google Form responses.

##### Action 1 – Get file content
- **+ New step** → **"Google Drive"** → **"Get file content"**.
- **File:** dynamic content → `Id` from the trigger.

##### Action 2 – Create file in SharePoint
- **+ New step** → **"SharePoint"** → **"Create file"**.
- **Site Address:** your SharePoint site.
- **Folder Path:** `/Recruitment/2024-Software-Engineer/`
- **File Name:** dynamic content → `Name` from the Google Drive trigger.
- **File Content:** dynamic content → `File Content` from Action 1.

##### Action 3 – (Optional) Delete source file from Google Drive
- **+ New step** → **"Google Drive"** → **"Delete file"**.
- **File:** dynamic content → `Id` from the trigger.

##### Action 4 – Notify HR via email
- **+ New step** → **"Send an email (V2)"**.
- Same pattern as Option 1, Action 6.

---

#### Limitations of Option 2

- Requires a Google Workspace or Gmail account as the intake address.
- Power Automate's Google Drive connector uses OAuth; you must re-authenticate periodically.
- Files do not carry the expiry-link security of Option 1.
- You lose the Microsoft-centric audit trail (all actions visible in SharePoint/Teams).

---

<a name="option-3"></a>
## 4. Option 3 – Encrypted Email Reply (Simplest, No Extra Licences)

### How It Works

When the form is submitted, Power Automate sends the candidate a pre-addressed email. The candidate simply **replies with attachments**. Power Automate watches the HR mailbox and saves any attachments from known senders to SharePoint.

### Step-by-Step

#### Step 1 – Trigger on Form submission (same as Option 1, Steps 1–3, Actions 1–2).

#### Step 2 – Send a reply-to email
- **+ New step** → **"Send an email (V2)"**.
- **To:** candidate email.
- **Subject:** `[Application] Please reply with your documents`
- **Body:**
  ```
  Dear [Name],

  Please reply to this email attaching:
    • CV
    • Motivation letter
    • Diplomas

  Do not change the subject line. This helps us match your documents to your application.

  HR Team
  ```

#### Step 3 – Watch for incoming attachments
1. Create a **second** Power Automate flow.
2. Trigger: **"When a new email arrives (V3)"** (Office 365 Outlook).
   - **Folder:** Inbox.
   - **Has Attachments:** Yes.
   - **Subject Filter:** `[Application]` (to avoid false positives).
3. **+ New step** → **"Apply to each"** (loop over attachments).
4. Inside the loop → **"SharePoint" → "Create file"**:
   - **File Name:** `Attachment Name` (dynamic content).
   - **File Content:** `Attachment Content` (dynamic content).
   - **Folder Path:** `/Recruitment/2024-Software-Engineer/` + sender email.

#### Limitations
- No access control: the email could be forwarded.
- Relies on the candidate not changing the subject line.
- Large attachments may be blocked by email server limits (typically 25 MB).

---

<a name="comparison"></a>
## 5. Comparison Table

| Feature | Option 1 (SharePoint Link) | Option 2 (Google Forms + Drive) | Option 3 (Email Reply) |
|---------|---------------------------|------------------------|----------------------|
| Extra licence needed | No (M365 included) | Google Drive connector | No |
| Security | ✅ High (expiring link, upload-only) | ⚠️ Medium | ⚠️ Low |
| File size limit | SharePoint: 250 GB | Google Drive: 5 TB | Email: ~25 MB |
| Newbie difficulty | ⭐⭐⭐ (medium) | ⭐⭐⭐⭐ (harder) | ⭐⭐ (easy) |
| Audit trail | ✅ Full SharePoint/M365 | ⚠️ Partial | ⚠️ Email only |
| Candidate experience | Good (one-click link) | Good (Google Form) | Familiar (email) |
| Recommended for | Most organisations | Google-first orgs | Backup / fallback |

---

<a name="pre-requisites"></a>
## 6. Pre-requisites & Licences

| Requirement | Option 1 | Option 2 | Option 3 |
|-------------|----------|----------|----------|
| Microsoft 365 Business Basic or higher | ✅ Required | ✅ Required | ✅ Required |
| Power Automate (included in M365 Business) | ✅ | ✅ | ✅ |
| SharePoint | ✅ Required | ✅ Required | ✅ Required |
| Google Workspace / Gmail account | ❌ | ✅ Required | ❌ |
| Google Drive Power Automate connector | ❌ | ✅ Required (premium) | ❌ |

> **Note:** The Google Drive connector in Power Automate is a **premium connector** and requires a Power Automate Per User or Per Flow plan (not included in the base M365 licence).

---

<a name="faq"></a>
## 7. Frequently Asked Questions

**Q: Can I re-send the upload link if the candidate misses the 30-minute window?**
> Yes. Add a **"When an email is received"** trigger looking for a subject line such as `[RE-SEND LINK]`, then re-run Actions 4–6 from Option 1 on demand.

**Q: How do I make sure two candidates with the same name don't overwrite each other's folders?**
> Use the candidate's email address as the folder name (it is unique), not their full name. The flow template above already does this.

**Q: Can I automate shortlisting using AI?**
> Yes. After files land in SharePoint, use Power Automate's **AI Builder** action **"Extract information from documents"** to parse CVs and populate a SharePoint list with structured data (skills, years of experience, etc.).

**Q: What if the candidate uploads a virus?**
> SharePoint Online scans uploaded files with Microsoft Defender. To add a second layer, enable **Defender for Cloud Apps** policies on the Recruitment library.

**Q: Can I use this for multiple job openings simultaneously?**
> Yes. Add a **Choice** question in the form for "Position applied for". Use the answer in the `CandidateFolderPath` variable to route files to the correct subfolder automatically.
