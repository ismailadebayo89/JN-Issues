# Repro for GitHub #6776 — Mailbox 1.8 / internetHeaders.setAsync

This is a minimal, hosted reproduction for
[microsoft/office-js#6776](https://github.com/OfficeDev/office-js/issues/6776):
`Office.context.requirements.isSetSupported("Mailbox", "1.8")` reports
`true` against an on-premise Exchange SE mailbox on both Legacy Outlook
Mac and New Outlook Mac, but `internetHeaders.setAsync` only succeeds
on Legacy Outlook Mac — it fails on New Outlook Mac despite the same
requirement-set check passing.

**Live taskpane:** https://ismailadebayo89.github.io/JN-Issues/6776-mailbox18-setasync/taskpane.html
**Manifest:** [`manifest.xml`](./manifest.xml) — sideload this directly, no editing required.

## Files

- `taskpane.html` — the whole repro. Logs requirement-set checks
  (`Mailbox` 1.1–1.9) and the `internetHeaders.setAsync` result straight
  to the page and browser console.
- `manifest.xml` — classic XML add-in manifest, already pointed at the
  live URL above. Sideloadable in both Legacy Outlook Mac and New
  Outlook Mac.
- `icon-16.png`, `icon-32.png`, `icon-80.png` — ribbon icons.

No build step, no dependencies — static HTML/JS hosted on GitHub Pages.

## How to reproduce (for Microsoft engineers)

1. Download [`manifest.xml`](./manifest.xml) from this folder.
2. Sideload it: Outlook on the web → Settings (gear) → **View all
   Outlook settings** → **Mail → Customize actions** → **My add-ins**
   → **Add a custom add-in** → **Add from file** → select the
   downloaded `manifest.xml`.
   (Sideloading is mailbox-scoped, so it will also appear in New
   Outlook Mac and Legacy Outlook Mac automatically once added.)
3. Against a mailbox hosted on **on-premise Exchange SE**, open a new
   message compose.
4. Click **Repro 6776 → Run 1.8 Check** in the compose ribbon, then
   **Run check** in the taskpane.
5. Compare the output between Legacy Outlook Mac and New Outlook Mac:
   - `isSetSupported(Mailbox, 1.8)` → reports `true` on **both**.
   - `internetHeaders.setAsync` result →
     `Office.AsyncResultStatus.Succeeded` on Legacy,
     `Office.AsyncResultStatus.Failed` on New (with an error code/message
     printed in the pane).

## What we're asking

Given both clients report Mailbox 1.8 as supported against the same
on-prem Exchange SE environment, we'd like to understand:

- Why `isSetSupported` returns `true` for New Outlook Mac against
  Exchange SE when the underlying `setAsync` call is not actually
  supported there.
- Whether this is a New Outlook Mac gap, an Exchange SE requirement-set
  reporting gap, or expected behavior we're missing documentation for.

Happy to provide further diagnostics (`Office.context.mailbox.diagnostics`
output, exact Exchange SE build/CU) on request — the taskpane above
already surfaces host, version, and full requirement-set results if
useful for triage.