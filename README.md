# Attendance — specification (overview)

Attendance is a browser app for one instructor to take attendance in several classes, keep attendance history, and email students who missed class. It runs entirely on the instructor's device with no server or account.

This document describes the features and the main decisions behind them. It leaves visual styling and fine-grained behavior to whoever builds it. Section 10 lists the choices made for this build as questions to ask before building a new copy.

---

## 1. Principles

- **One file, no dependencies.** The whole app is a single `index.html` that works offline, including when opened straight from disk.
- **Private by default.** All data stays in the browser's local storage. Nothing leaves the device unless the instructor downloads a file or opens an email draft. Exported files with student data carry a privacy reminder.
- **Hard to make mistakes.** Every assigned student starts out present, so only exceptions are marked. Days other than today are locked until unlocked. The selected date is always prominent, with a strong warning when it isn't today. Attendance changes can be undone. Destructive actions ask first.
- **Keyboard and touch friendly.** Every main task can be done with the keyboard, and the controls are large enough for an iPad.
- **Always dark theme.**

## 2. Core concepts

- **Views (tabs):** Take attendance · Review history · Send emails · Manage data. Tab names are actions. All interface text uses sentence-style capitalization (only the first word and proper nouns capitalized).

- **Class:** a course section with a roster, seat assignments, attendance records, meeting days, and printed sign-in sheets. The instructor works in one class at a time and switches between them with a class menu.
- **Room:** a layout only, made of rows of seats (with aisles) or tables. Each class uses one room, and sections that meet in the same place can share it.
- **Student:** first name, last name, and optional email. No student ID is kept.
- **Attendance:** for each date, each student is Present, Absent, or Excused. A date that has records is a recorded date.
- **Selected date:** the day being worked on. It defaults to today and is shared across views.

## 3. Taking attendance

### Seating chart
- The chart shows the room as seat cards with each student's name and status. Clicking a seat, or pressing P, A, or E, sets the status.
- Rooms can be **rows**, with any number of aisles, or **tables / pods**, with seats numbered clockwise around each table. Seats are named "Row 2, Seat 5" or "Table 3, Seat 2".
- **Instructor view** turns the chart 180° to match the view from the front of the room.
- **Edit seats** assigns students to seats, moves or swaps them (by dragging or by seat number), and changes the room layout. Layout changes that would leave students without a seat are refused, or the app offers to move only the students who don't fit. A shared room can't be changed in a way that unseats another class's students. Students can't be removed from this view.

### List
- The list is the default view the first time the app opens; after that it reopens on whatever tab and view were last used. Its toggle sits to the left of Seating chart.
- Creating a sign-in sheet (with or without a PDF) opens that sheet in Sheet mode.
- The list shows the roster sorted by last name, the same order as the printed sign-in sheet, with P / A / E controls on each row.
- **One date** mode is for the selected date. **Sheet** mode puts all the dates from one printed sheet side by side, so a filled-in sheet can be entered in one pass.
- **Start everyone as Present or Absent** sets how an unrecorded date is treated. With Absent, the instructor marks only the students who signed in, and the first mark records the date.
- The keyboard is fully supported: arrow keys move, P/A/E (or Q/W/E, which sit side by side) mark and move to the next student, and typing jumps to a last name.
- In Sheet mode, × on a recorded column deletes that date's records, the same as in History (undoable).

### Paper sign-in sheets
- A printable PDF (US Letter, landscape or portrait) with last name, first name, and empty date columns for students to initial. The date columns either show the class's next meeting dates or are left blank for students to write in. Optional blank rows at the bottom (none by default) cover late adds.
- Row height is adjustable, with an option to fit every student on one page.
- Each sheet is numbered and remembered. Sheet mode in the list view picks a sheet to transcribe, and a sheet can be downloaded again. Sheet mode doesn't require printing: "Recent class dates" is always available, and a sheet can be created without a PDF.
- The app generates the PDF itself, with no library.

### Dates
- A prominent date bar is always visible on the views that use a date. It has previous/next day buttons, a large calendar, a Today button, and shortcuts to recent class dates.
- Previous and next step through the class's meeting days. On any day other than today, the bar and the window take on a warning color.

## 4. Review history

- A grid with a row for each student and a column for each recorded date. It is editable, stays in sync with the classroom views, and shows absence and excused totals.
- Clicking anywhere in a row selects it, and the selected row and column are highlighted.
- Last and first names are separate columns. Rows can be sorted by seat (the default), last name, first name, absences, or excused, by clicking a column header; clicking again reverses. The sort is remembered. While marking cells the rows stay put, with a Re-sort button to refresh the order.
- Students can be found by searching name or email. A date can be found with a jump-to-date list, the calendar, or by stepping between recorded dates. Dates can be added or deleted.
- Escape resets the view.
- Cells with a note or an email sent about that date are marked. N adds a note and I opens the student's summary.

### Notes, email log, and student summary
- **Notes:** one short note per student per date (for example, "doctor's note received"), added from a seat, a list row, a sheet cell, or a history cell. Notes appear in history, beside the classroom, next to students in Send emails, and in the summary. Note changes can be undone.
- **Email log:** opening a draft in the mail app, downloading a .eml, or copying addresses or a personal email records who it was for, the date it was about, the subject, and the template. Students already emailed about the selected date are flagged, with a warning before emailing them again. Entries can be removed, since the app can't tell whether a draft was actually sent.
- **Student summary:** a printable page per student with totals, attendance rate, absences and excused days with notes, and emails sent, with a shortcut to email that student.

## 5. Send emails

- **Recipients:** filters show All, Absent today, Excused today, or Absent N+ times (adjustable). The list shows only the filtered students. Students can be searched and selected individually or as a range.
- **Sending:** either one email with all recipients in **BCC**, or **separate personal emails**. In a BCC email the name placeholders are left blank, so the greeting reads "Hi,". Drafts open in the mail app, download as .eml files, or copy to the clipboard. The app never sends anything itself.
- **Message editor:** the editor is a live working draft that uses placeholders for first name, name, date, class, seat, instructor, office hours, office, topic, focus, and videos. The instructor's name, office hours, and office are set once in fields above Save template; templates put {instructor} under their closing and {officehours}/{office} on their own lines, and a line whose detail is blank is left out. Templates are starting points: editing one changes only the current draft until it is saved as a template. Thirty-three starter templates, written in an empathetic, supportive tone, cover:
  missed class check-in · excused absence documentation · repeated absences · missed quiz or exam · several missed classes · excused absence confirmation · missed group work · nearing the attendance limit · missed exam review · welcome back after time away · missed announcement · eight general notes that suit a single missed day (a short note, easy ways to catch up, a no-reply-needed check-in, a how-are-things-going check-in, an office-hours invitation, where to find what was covered, a glad-you're-here note, and campus support if things are busy), plus four more gentle notes that say no reply is needed (a quick note, help is available, materials are posted, and see you next class). Every starter that mentions office hours also lists {officehours} and {office} just above its closing. Four informational templates give office hours and the office location (help catching up, a no-reply-needed reminder, where to get help, and please stop by office hours). Templates keep a professional, course-focused tone with neutral closings, avoiding wording that could read as too personal.
- **Edited starters can be reset.** A starter whose saved wording differs from the built-in version is marked "(edited)" in the template menu and offers "Restore original wording"; the edited version is kept in Recently deleted and can come back as a separate template.
- **Video guides and topics:** each class can load a video guide (Markdown: `##` chapters, `###` topics such as "§2.3 The Limit Laws", a `**Focus:**` line, and bulleted video links; everything else is ignored), optionally reused for other sections. In Send emails a Topic menu under the date picks what was covered; the choice is remembered per class and date and fills `{topic}`, `{focus}`, and `{videos}`. With no topic, paragraphs using those tags are left out. Six starter templates are built around them; every other starter carries an optional "If it helps, these videos cover what we worked on that day ({topic}): {videos}" paragraph that appears only when a topic is picked. Guides and topics travel with backups and merge restore.
- **The library never shrinks by accident.** Starter templates are part of the app: every one stays in the library unless the instructor deletes it (deleted ones can be restored), new starters added in later versions appear automatically, and restoring older data keeps templates that exist only on this device.

## 6. Managing data

- **Classes and rooms:** create, rename, and delete classes; set each class's meeting days; and assign, share, copy, or delete rooms. Each class in the list has **Import…** (a CSV goes into that class, ignoring any class column) and **Check for drops…**; a separate **Import a CSV** card below handles files with several classes. The roster card is titled with the current class's name. The **New class** dialog asks for the name, meeting days, and room, plus two optional files, each with a brief formatting guide, a downloadable template or sample, and a check of what was found: a roster (Brightspace export, a last/first/email CSV, or pasted names) and a video guide (.md). Empty classes show an Import a roster panel in every view. Imports accept a headerless list of names and skip non-student roles in Brightspace files.
- **Roster:**
  - **Edit** a student's first name, last name, and email in place, one row at a time. Changes are checked: a name is required and emails must look valid. Seats are changed only in Edit seats.
  - **Check for drops:** compare a new class list (for example, a Brightspace classlist export with only names) with the open class. Matches are by email, then name (ignoring accents, case, order, and extra surnames); near matches are shown for confirmation; missing students are listed with absences and last day present and can be dropped as of a chosen date, keeping their history. New names can be added to open seats, and dropped students who reappear can be restored. Nothing changes until the instructor confirms.
  - **Remove** a student with a choice: **Drop** keeps their history, lists them as dropped with their last day attended, and can be restored; **Delete** erases them completely.
  - A review step checks first/last name splits when the app had to guess.
- **CSV import and export:** one row per student, with a class column so files can hold one class or all classes. On import, rows go to the class named in each row, and new class names create classes. Existing classes can either be **updated** (the default: students are matched by email or name, and seats and history are kept) or **replaced** completely. Files can be dragged onto this view.
- **Backups:**
  - a full .json backup of everything, from a prominent card at the top of Manage data (or Ctrl/⌘+S anywhere). The card shows when the last backup was made and warns when there are changes since; a Restore button sits next to it
  - restoring a backup offers **Merge** (the default) or **Replace**. Merge never removes anything; it adds the classes, students, dates, marks, notes, email log entries, sheets, and templates the backup has that this device doesn't, which keeps a computer and an iPad in step. When both sides have a different mark for the same student and date, the instructor chooses which side wins. Drops carry over: a student dropped in the backup is dropped here too (on by default, with the backup's drop date), and a student dropped here but active in the backup can be restored (off by default). Either way the current data is kept as an automatic copy first
- **Recently deleted:** deleted students, dates, classes, rooms, and templates, and cleared classes, are kept for 30 days and can be restored (with attendance, notes, and seat) or deleted forever
  - a copy of the app with its data built in, which can be opened on another device to pick up where you left off
  - a clean copy of the app with no data, safe to host

## 7. CSV format (summary)

```
class,last_name,first_name,email,row,seat,2026-10-13,2026-10-15
Calculus I – Section 01,Garza,Ana,ana@example.edu,1,1,P,A
```

- Each date column holds P, A, or E. Within a recorded date, a blank cell means present. A date counts as recorded for a class only if at least one of its students has a mark on it, which lets classes that meet on different days share one file.
- Single-class files can include optional `#` lines with the room layout.
- Imports accept common header variations: a single "Name" column (including "Last, First"), US-style dates (10/13/2026), a student ID column (ignored), and comma, semicolon, or tab separators. Students without seats are placed in open seats automatically.

## 8. Hosting and offline use

- When hosted (for example on GitHub Pages), a small service worker saves the app so it opens without internet. It always loads the latest version when online and never stores student data.
- A manifest and icon let it be added to an iPad Home Screen as an app.
- Data is tied to the app's address in each browser, so moving between devices is done with the backup file or the copy of the app with data built in.
- **Updates must never lose data.** Publishing a new version leaves saved data alone. The app also protects against address changes and bad data: `…/` and `…/index.html` share data (real data wins over an untouched demo), data saved under any other address on the same site is listed under **Recover data** (with a banner when the app opens with nothing of yours), data that can't be read is kept rather than overwritten, and an automatic copy is kept from each of the last 7 days.

## 9. Help and onboarding

- A Help dialog explains each workflow and lists the keyboard shortcuts.
- Demo data (one class) is included so the app can be tried right away, and it can be replaced or deleted.

## 10. Questions to ask before building

The answers in italics are the choices made for this build.

**Basics**
1. What should the app be called? *Attendance.*
2. One offline file with browser storage, or a server/shared database? *One file, browser storage.*
3. Theme: dark, light, or follow the system? *Always dark.*
4. Should demo data be included? *Yes.*

**Taking attendance**
5. Mark only exceptions (everyone starts present), or mark every student? *Exceptions, with an option to start everyone absent in the list view.*
6. Which statuses? Is Late needed? *Present, Absent, Excused.*
7. Lock days other than today against accidental edits? *Yes, until unlocked, with strong visual warnings.*
8. Seating chart, list, or both? *Both.*
9. Room shapes needed: rows, tables, curved, U-shape? *Rows with aisles, and tables.*
10. How are seats numbered? *Row 1 at the front, seat 1 on the students' left; table seats clockwise.*
11. Should there be paper sign-in sheets? With dates printed or blank? Which page orientation? *Yes, PDF; dates printed or blank; landscape or portrait; adjustable row height, with a fit-on-one-page option.*

**Students and classes**
12. Several classes in one app? Separate room layouts? *Yes; rooms separate from classes and shareable.*
13. Store first and last names separately? *Yes, with a review step for guessed splits.*
14. Should student IDs be kept? *No; names and emails are enough.*
15. How are students removed? Keep their history? *Only from Manage data; drop (keeps history, restorable) or delete.*

**History and email**
16. Should history be editable? *Yes, synced with the classroom views.*
17. How should email work: mailto, .eml files, or a sending service? *mailto, .eml, and copy; never sends automatically.*
18. One BCC email, separate personal emails, or both? *Both.*
19. Tone and content of the starter templates? *Empathetic and supportive; 33 templates, including twelve general ones suitable for a single missed day and four that give office hours and location.*

**Data**
20. Which CSV columns and in what order? *class, last name, first name, email, seat, then dates.*
21. Should imports into an existing class merge or replace? *Both, with merge as the default.*
22. Which backup options? *Full .json, a copy of the app with data, and a clean copy of the app.*
23. Should it work offline when hosted and install to an iPad Home Screen? *Yes.*
23a. How should the app protect saved data across updates and address changes? *Recover data from other addresses, keep unreadable data, and keep daily automatic copies (last 7).*

**Interaction**
24. Which keyboard shortcuts, and are single-letter keys acceptable? *Number keys switch views; letters for common actions; single letters work whenever you aren't typing.*
25. Is undo needed? *For attendance and note changes, plus a 30-day bin for deletions.*
26. Should notes, an email log, and per-student summaries be kept? *Yes.*
26a. Should emails be able to include the day's topic and videos? How is the topic chosen? *Yes, from a per-class video guide; picked per date in Send emails (no schedule, not set during attendance).*
27. How should two devices be kept in step? *Backups with a merge restore; the instructor picks which side wins on conflicts.*
