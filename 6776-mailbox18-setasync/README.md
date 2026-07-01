# Repro for GitHub #6776 — Mailbox 1.8 / internetHeaders.setAsync

Files in this folder:

- `taskpane.html` — the whole repro. Logs requirement-set checks and the
  `internetHeaders.setAsync` result straight to the page (and console).
- `manifest.xml` — classic XML add-in manifest, sideloadable in both
  Legacy Outlook Mac and New Outlook Mac.
- `icon-16.png`, `icon-32.png`, `icon-80.png` — placeholder ribbon icons
  (swap for your own if you care about branding; not load-bearing for the bug).

There's no build step — it's static HTML/JS, which is all Azure Static
Web Apps needs.

---

## 1. Deploy to Azure Static Web Apps (no GitHub repo needed)

Easiest path is the SWA CLI, deploying straight from this folder:

```bash
npm install -g @azure/static-web-apps-cli
az login
swa deploy ./repro --env production --app-name repro-6776 --resource-group <your-rg>
```

If you'd rather click through the portal instead:

1. Portal → **Create a resource** → **Static Web App**.
2. Hosting plan: **Free**.
3. Deployment source: **Other** (this skips GitHub Actions entirely —
   you'll just drag-and-drop / `swa deploy` afterward).
4. Once created, grab the **URL** from the resource overview
   (`https://<random-name>.azurestaticapps.net`).
5. Deploy the files: `swa deploy ./repro --deployment-token <token from portal>`.

Either way, end state is a public HTTPS URL serving `taskpane.html` —
that's the only hard requirement for Office Add-ins (must be HTTPS).

## 2. Point the manifest at your real URL

Find-and-replace every occurrence of `YOUR-SWA-URL.azurestaticapps.net`
in `manifest.xml` with your actual Static Web App hostname.

```bash
sed -i 's/YOUR-SWA-URL.azurestaticapps.net/<your-real-host>/g' manifest.xml
```

## 3. Sideload it for testing

**Outlook on the web / New Outlook (any platform):**
Settings (gear) → **View all Outlook settings** → **Mail → Customize actions**
→ **My add-ins** → **Add a custom add-in** → **Add from file** → pick
`manifest.xml`. New Outlook for Mac picks this up automatically.

**Legacy Outlook for Mac:**
Same custom-add-in flow via Outlook on the web works there too (mailbox-scoped
sideloading), or use Outlook's own **Get Add-ins** dialog if your build
supports local sideload.

## 4. Reproduce against the on-prem Exchange SE mailbox

1. Open the add-in on a **new message compose** (it's scoped to
   `MessageComposeCommandSurface`).
2. Click **Run 1.8 Check** in the ribbon.
3. Capture what the pane shows on both Legacy Outlook Mac and New Outlook Mac
   — specifically the `isSetSupported` lines and the `setAsync` result/error
   code. That's the side-by-side Microsoft needs.

## 5. Hand it to Microsoft

In the GitHub issue, add a comment with:
- The deployed taskpane URL (so they can sideload `manifest.xml` themselves
  in under a minute).
- A copy of `manifest.xml` attached directly (don't make them edit the URL).
- The two console outputs (Legacy vs New) as text, not just a screenshot —
  makes it greppable/diffable on their end.
