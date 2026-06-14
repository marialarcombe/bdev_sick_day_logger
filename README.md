# BDEV Sick Day Logger

A single-file internal payroll tool for logging staff absences across Best Day Ever Venues and generating month-end return-to-work reports for payroll.

Kept entirely separate from The Tawny's logger (separate repo, separate data, separate recipients).

## What it does

1. **Select an employee** and **set a date range** (last day worked through to first day back).
2. **Mark each day** as Worked, Sickness, Holiday, or Day off. Only Sickness days count as missed shifts.
3. **Generate a payroll report** in the agreed format, then **save** it.
4. **At month end**, pick the month and click *Open Email Draft*. The tool opens a pre-addressed email to **laura@ensarb.com** with all that month's entries, formatted and colour-coded. Review and send. If a Google Sheet is linked (see below), the same click also appends that month's data to the sheet.
5. **Return to Work form.** On each saved entry there's a *RTW form* button. It opens the full Return to Work & Self Certification form (Sections A–D), pre-filled with the employee name and absence dates. Complete it with the employee, type both signatures, and click *Send*. The tool generates a BDEV-branded PDF and emails it to **Tamara (tamara@ensarb.com)** and **Laura (laura@ensarb.com)** to add to the employee's file. *Preview PDF* lets you check or download a copy first.

## Storing data

Entries, the employee list, and your sign-off name are saved in the **browser's local storage** on the device you use it on. There is no server.

Use **Settings → Export Backup** regularly (and before clearing your browser or switching computers). Restore with **Import Backup**.

## Hosting

It is a single `index.html` with no build step or dependencies (fonts load from Google Fonts, the PDF library from a CDN). To publish:

1. Commit `index.html` (and this README) in GitHub Desktop and push.
2. Enable **Settings → Pages → Deploy from branch → main / root** to get a live URL.

Or just open `index.html` in a browser to use it locally.

## Brand

Palette and type follow the BDEV house style: pink `#BC93AF`, with Domine headings and Nunito Sans body. The status colours (red sickness / green worked / blue holiday) are kept functional for quick scanning. The Return to Work PDF carries the Best Day Ever Venues logo.

## The Google backend (optional, powers two features)

One small Google Apps Script "web app" does two jobs for the tool:

1. **Month-end sheet sync** — when you click *Open Email Draft*, it appends one
   row per absence to a Google Sheet:
   `Logged to sheet | Month | Employee | Sick shifts missed | From | To | Date logged | Full report`
2. **Return to Work forms** — when you click *Send* on a form, it emails the
   completed PDF to Tamara and Laura (and can also file it in a Drive folder).

You paste **one link** into Settings and it powers both. Because the tool is a
static page with no server, it talks to this script. One-time setup, ~5 minutes:

1. **Create the sheet.** Make a new Google Sheet (e.g. *BDEV Sick Days*) and
   share it with whoever needs it.
2. **Add the script.** In the sheet: **Extensions → Apps Script**. Delete any
   placeholder code, paste the contents of [`google-sheet-sync.gs`](google-sheet-sync.gs),
   and click **Save**.
3. **Deploy it.** Top right: **Deploy → New deployment**. Click the gear and
   choose **Web app**. Set:
   - **Description:** BDEV sick day sync
   - **Execute as:** Me
   - **Who has access:** Anyone
   Click **Deploy**, then **Authorize access** and allow it (it's your own script).
4. **Copy the link.** Copy the **Web app URL** (it ends in `/exec`).
5. **Paste it into the tool.** Open **⚙ Settings → Google Sync** and paste the
   URL. You'll see *"✓ Linked"*.

That's it. Opening the email draft now also sends the month to the sheet, and
*Send* on a Return to Work form emails the PDF to Tamara and Laura. The tool
remembers what it has already sent on that device, so reopening the draft won't
create duplicate rows.

**Recipients for RTW forms** are set at the top of `google-sheet-sync.gs`
(`RTW_RECIPIENT = 'tamara@ensarb.com,laura@ensarb.com'`). To also file each PDF
in Drive, paste a folder ID into `RTW_DRIVE_FOLDER_ID`.

**Already deployed the sheet-only version?** The Return to Work feature sends
email, which needs an extra permission. Paste the newer `google-sheet-sync.gs`,
then **Deploy → Manage deployments → Edit → Version: New version → Deploy**, and
authorise again when prompted.

**Notes**
- Treat the web-app URL like a password, it lets anyone with it add rows or
  trigger an email. Don't publish it. (For extra safety, set `SHARED_SECRET` in
  the script and add `?secret=YOURWORD` to the URL you paste into Settings.)
- The URL is stored only in each person's browser, never in this repo or the
  published page.

## Return to Work forms & data protection

The Return to Work form captures **health information** (reason for absence,
medication, GP, fitness to return). Under UK GDPR this is special-category data,
so the tool is built to minimise it:

- **Nothing from the form is stored in the browser.** The answers are used to
  build the PDF and send it, then discarded. The only thing kept against an entry
  is a flag that a form was sent (and the date), no medical detail, so it isn't
  in exports/backups either.
- Non-sensitive details (payroll number, job title, department) are remembered
  per employee to save re-typing. These are not health data.
- The PDF is emailed to internal recipients (Tamara and Laura). Make sure those
  mailboxes are appropriate for sickness records, and confirm the approach with
  whoever owns data protection.
- Typed names act as electronic signatures. If wet-ink signatures are required,
  use *Preview PDF*, print, and sign by hand instead.

## Changing the email recipients

The **monthly** recipient (Laura) is set in `index.html` in the `openEmail()`
function (`mailto:laura@ensarb.com`) and the greeting in `buildEmailBodyPlain()`
/ `buildEmailBodyHtml()`. The **Return to Work** recipients (Tamara and Laura)
are set in `google-sheet-sync.gs` (`RTW_RECIPIENT`).

---

Best Day Ever Venues · Internal use only
