# Reset Password Page — Design Spec
**Date:** 2026-04-03  
**Repo:** `TheSambarVada/getchillpill.xyz`  
**File to create:** `getchillpill.xyz/reset-password.html`

---

## Problem

Supabase's "forgot password" flow emails the user a recovery link that redirects to the site URL (`https://getchillpill.xyz`) with a URL hash fragment:

```
https://getchillpill.xyz/#access_token=...&refresh_token=...&type=recovery
```

No page currently handles this. Users land on the homepage with no way to set a new password.

---

## Solution

Create `reset-password.html` — a standalone page that:
1. Parses the hash fragment on load
2. Validates `type=recovery` and presence of `access_token`
3. Shows a "Set new password" form
4. Calls Supabase `updateUser` with the new password
5. Shows success state + redirects to homepage after 2s

---

## Architecture

**Single HTML file** — no build step, no framework. Same pattern as `feedback.html`, `terms.html`, etc.

**Supabase JS SDK** loaded from CDN (`@supabase/supabase-js` v2).

**Auth flow:**
1. Parse `window.location.hash` → extract `access_token`, `refresh_token`, `type`
2. Call `supabase.auth.setSession({ access_token, refresh_token })` using both values from the hash — this authenticates the Supabase client with the recovery token. (`refresh_token` is present in the hash for recovery links.)
3. On form submit → `supabase.auth.updateUser({ password: newPassword })`
4. On success → show success message, `setTimeout(() => location.href = '/', 2000)`

---

## UI States

| State | What's shown |
|-------|-------------|
| **Invalid link** | Error card: "This link has expired or is invalid. Request a new one in the app." |
| **Form** | Two fields: "New password" + "Confirm password". Submit button: "Set New Password". |
| **Loading** | Button disabled, shows "Saving…" |
| **Success** | Green message: "Password updated. Redirecting…" |
| **Error** | Inline error below the form (e.g. "Password must be at least 6 characters", "Link has expired") |

---

## Validation

- Both fields must match before submitting
- Minimum 6 characters (Supabase minimum)
- Show inline error without page reload on mismatch

---

## Styling

Identical to `feedback.html`:
- Background: `#08080f`
- Font: Inter (Google Fonts CDN)
- Accent: `#a78bfa` (purple)
- Card: `rgba(16, 16, 26, 0.92)` with border `rgba(255,255,255,0.08)`
- Max width: 420px, centered
- Inputs: dark background, purple focus ring

---

## Error Handling

| Error | User-facing message |
|-------|-------------------|
| No token / wrong type | "This link has expired or is invalid." (shown on load, no form) |
| Passwords don't match | "Passwords don't match." (inline, before submit) |
| Password too short | "Password must be at least 6 characters." (inline) |
| Supabase error (expired token) | Map error message → "This link has expired. Request a new one in the app." |
| Network error | "Something went wrong. Please try again." |

---

## Out of Scope

- Email change confirmation (different `type` value — not needed now)
- Magic link handling
- Any server-side logic — purely client-side
