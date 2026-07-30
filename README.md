# Comply Globally — Landing Page + Dr. CV Voice Agent

Static site: a lead-capture landing page that hands the visitor off into a
live AI voice call ("Dr. CV") right after they submit the form.

## Files

| File         | Purpose                                                          |
|--------------|-------------------------------------------------------------------|
| `index.html` | **Live entry point.** The landing page + lead form. GitHub Pages serves this automatically. |
| `call.html`  | The live voice call screen. Visitors land here right after submitting the form, with their info attached as URL params. |
| `main.html`  | Legacy copy of the landing page, kept only for reference — not used anywhere. Safe to delete once you're confident `index.html` is working. |
| `package.json` | Optional — lets you preview the site locally with `npm start`. Not needed for deployment. |

## Config you need to check/update

### In `index.html`
```js
const WEBHOOK_URL = "https://script.google.com/macros/s/XXXX/exec";
```
This is your Google Apps Script endpoint that logs form submissions to a
Sheet. Replace with your own deployed Apps Script Web App URL if this one
isn't yours or has changed.

### In `call.html`
```js
const CFG = {
  BACKEND: 'https://drcv.onrender.com',   // ← must match your Render service URL exactly
  CAL:     'https://cal.com/...',         // ← your booking link for the end-of-call CTA
  GAS:     'https://script.google.com/macros/s/YYYY/exec', // ← logs call scores/transcripts
  VOICE:   'marin',
  ...
};
```

> ⚠️ `index.html`'s `WEBHOOK_URL` and `call.html`'s `GAS` are currently two
> *different* Apps Script deployments. If you meant for form leads and call
> outcomes to land in the same sheet, point them at the same URL.

## How the flow works

1. Visitor lands on `index.html`, fills out the form.
2. Form data POSTs to `WEBHOOK_URL` (Google Sheet).
3. Browser redirects to `call.html?name=...&email=...&company=...&country=...&stage=...&leadId=...&sessionId=...`
4. `call.html` reads those URL params, opens a mic-enabled WebRTC voice call
   with OpenAI's Realtime API (routed through your Render backend so your
   OpenAI key stays server-side).
5. As the call progresses, `call.html` scores engagement/intent and posts
   updates + a final summary to `GAS`.

## Deploying with GitHub Pages

1. Repo → **Settings → Pages**.
2. Source: **Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)`.
4. Save. Your site goes live at `https://<username>.github.io/<repo>/`.
   (If you want a custom domain, add a `CNAME` file with just your domain
   name in it, and point your DNS at GitHub Pages per their docs.)

## Local preview

```bash
npm start
```
Opens the site at `http://localhost:5500`. Note: the Google Sheets POST
uses `mode:'no-cors'` so it works from any origin, but the voice call still
needs your Render backend to be live and reachable.

## Related repo

The voice backend (`/session` token proxy) lives in the `jsjs` repo. See
its README for setup and the `OPENAI_API_KEY` requirement.
