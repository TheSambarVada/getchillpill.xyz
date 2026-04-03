# Reset Password Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create `reset-password.html` so users who click a Supabase password-recovery email link can set a new password instead of landing on a broken homepage.

**Architecture:** Single standalone HTML file. No build step, no framework — same pattern as `feedback.html`. Supabase JS SDK v2 loaded from CDN. JavaScript parses the URL hash fragment, calls `supabase.auth.setSession()` with the recovery token, then `supabase.auth.updateUser()` on form submit.

**Tech Stack:** Vanilla HTML/CSS/JS, Supabase JS SDK v2 (CDN), `@supabase/supabase-js`

---

## Scene Setting

**Repository:** `TheSambarVada/getchillpill.xyz`. Local path: `C:/Users/man4m/FreeFlow/getchillpill.xyz/`

**File to create:** `getchillpill.xyz/reset-password.html`

**Supabase credentials (safe to embed — anon key only allows auth operations):**
- `SUPABASE_URL` = `https://qnpnmxdvblygckzpofac.supabase.co`
- `SUPABASE_ANON_KEY` = `sb_publishable_0kLok6guLj8AaP3dFlHwKw_H0sYcNNM`

**How the recovery flow works:**
Supabase sends the user an email with a link to `https://getchillpill.xyz/#access_token=...&refresh_token=...&type=recovery`. The page must parse the `#` fragment (not query params), call `supabase.auth.setSession({ access_token, refresh_token })` to authenticate the client, then call `supabase.auth.updateUser({ password })` to save the new password.

**Style reference:** `getchillpill.xyz/feedback.html` — copy CSS variables and component classes exactly. The card pattern, input styles, button styles, nav, and footer all come from there.

---

## Files

- **Create:** `getchillpill.xyz/reset-password.html`

---

## Task 1: Create `reset-password.html` with all UI states

**File:**
- Create: `getchillpill.xyz/reset-password.html`

This task creates the complete file in one shot — it is a single self-contained HTML page, and splitting it into sub-steps would require editing a half-broken file multiple times.

- [ ] **Step 1: Create the file**

Create `C:/Users/man4m/FreeFlow/getchillpill.xyz/reset-password.html` with the following complete content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reset Password — Chill Pill</title>
    <meta name="description" content="Set a new password for your Chill Pill account.">
    <link rel="icon" type="image/x-icon" href="chillpill.ico">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #08080f;
            --bg-soft: #10101a;
            --panel: rgba(16, 16, 26, 0.92);
            --panel-strong: #141420;
            --text: #f3f4fb;
            --muted: #8b90a7;
            --border: rgba(255, 255, 255, 0.08);
            --border-strong: rgba(167, 139, 250, 0.22);
            --accent: #a78bfa;
            --accent-2: #38bdf8;
            --green: #22c55e;
            --green-strong: #16a34a;
            --danger: #fb7185;
            --shadow: 0 24px 80px rgba(0, 0, 0, 0.45);
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            min-height: 100vh;
            font-family: 'Inter', 'Segoe UI', sans-serif;
            color: var(--text);
            background:
                radial-gradient(circle at top, rgba(167, 139, 250, 0.18), transparent 28%),
                radial-gradient(circle at 85% 18%, rgba(56, 189, 248, 0.12), transparent 20%),
                linear-gradient(180deg, #090910 0%, #07070d 100%);
        }

        a { color: inherit; text-decoration: none; }

        .page-shell {
            position: relative;
            overflow: hidden;
        }

        .grid-glow {
            position: absolute;
            inset: 0;
            background-image:
                linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
            background-size: 48px 48px;
            mask-image: linear-gradient(180deg, rgba(0, 0, 0, 0.65), transparent 92%);
            pointer-events: none;
        }

        .nav {
            max-width: 1120px;
            margin: 0 auto;
            padding: 24px 32px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            position: relative;
            z-index: 1;
        }

        .nav-brand {
            display: inline-flex;
            align-items: center;
            gap: 12px;
            font-weight: 700;
            letter-spacing: -0.2px;
        }

        .nav-brand img {
            width: 42px;
            height: 42px;
            border-radius: 12px;
            box-shadow: 0 10px 24px rgba(0, 0, 0, 0.26);
        }

        .nav-links {
            display: flex;
            gap: 22px;
            align-items: center;
            color: #727893;
            font-size: 14px;
        }

        .nav-links a:hover { color: var(--text); }

        .main {
            max-width: 480px;
            margin: 0 auto;
            padding: 60px 24px 96px;
            position: relative;
            z-index: 1;
        }

        .card {
            background: linear-gradient(180deg, rgba(18, 18, 31, 0.96), rgba(12, 12, 20, 0.96));
            border: 1px solid var(--border-strong);
            border-radius: 28px;
            padding: 32px;
            box-shadow: var(--shadow);
            position: relative;
            overflow: hidden;
        }

        .card::before {
            content: "";
            position: absolute;
            inset: 0;
            background: radial-gradient(circle at top right, rgba(167, 139, 250, 0.08), transparent 40%);
            pointer-events: none;
        }

        .card-title {
            font-size: 22px;
            font-weight: 700;
            letter-spacing: -0.5px;
            margin-bottom: 6px;
            position: relative;
        }

        .card-copy {
            color: var(--muted);
            font-size: 14px;
            line-height: 1.6;
            margin-bottom: 24px;
            position: relative;
        }

        form {
            display: grid;
            gap: 16px;
            position: relative;
        }

        .label {
            display: block;
            font-size: 13px;
            font-weight: 600;
            color: #d5d9ef;
            margin-bottom: 8px;
        }

        input[type="password"] {
            width: 100%;
            border: 1px solid var(--border);
            background: rgba(255, 255, 255, 0.035);
            color: var(--text);
            border-radius: 16px;
            padding: 14px 15px;
            font: inherit;
            outline: none;
            transition: border-color 0.18s ease, box-shadow 0.18s ease, transform 0.18s ease;
        }

        input[type="password"]:focus {
            border-color: rgba(167, 139, 250, 0.42);
            box-shadow: 0 0 0 4px rgba(167, 139, 250, 0.11);
            transform: translateY(-1px);
        }

        .error-msg {
            display: none;
            color: var(--danger);
            font-size: 13px;
            margin-top: 4px;
        }

        .error-msg.is-visible { display: block; }

        .submit-btn {
            width: 100%;
            border: 0;
            border-radius: 16px;
            padding: 14px 18px;
            font: inherit;
            font-weight: 700;
            cursor: pointer;
            background: linear-gradient(135deg, var(--accent) 0%, #818cf8 100%);
            color: white;
            box-shadow: 0 10px 30px rgba(167, 139, 250, 0.25);
            transition: transform 0.16s ease, box-shadow 0.16s ease, opacity 0.16s ease;
            margin-top: 4px;
        }

        .submit-btn:hover:not(:disabled) { transform: translateY(-1px); }

        .submit-btn:disabled {
            opacity: 0.55;
            cursor: not-allowed;
        }

        /* Invalid-link state */
        .invalid-panel {
            display: none;
            position: relative;
        }

        .invalid-panel.is-visible { display: block; }

        .invalid-panel .icon {
            font-size: 36px;
            margin-bottom: 14px;
        }

        .invalid-panel p {
            color: var(--muted);
            font-size: 14px;
            line-height: 1.6;
            margin-bottom: 20px;
        }

        .back-link {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            font-size: 14px;
            font-weight: 600;
            color: var(--accent);
        }

        .back-link:hover { opacity: 0.8; }

        /* Success state */
        .success-panel {
            display: none;
            position: relative;
        }

        .success-panel.is-visible { display: block; }

        .success-panel .icon {
            font-size: 36px;
            margin-bottom: 14px;
        }

        .success-panel p {
            color: #b7ebc6;
            font-size: 14px;
            line-height: 1.6;
        }

        .hidden { display: none !important; }

        .foot {
            max-width: 1120px;
            margin: 0 auto;
            padding: 0 32px 38px;
            position: relative;
            z-index: 1;
        }

        .foot-bar {
            border-top: 1px solid rgba(255, 255, 255, 0.06);
            padding-top: 18px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 18px;
            color: #646a84;
            font-size: 13px;
        }

        .foot-links {
            display: flex;
            gap: 18px;
            flex-wrap: wrap;
        }

        .foot-links a:hover { color: var(--text); }

        @media (max-width: 600px) {
            .main { padding: 32px 16px 64px; }
            .card { padding: 24px; }
            .nav { padding: 20px 16px; }
            .foot { padding: 0 16px 32px; }
            .foot-bar { flex-direction: column; align-items: flex-start; }
        }
    </style>
</head>
<body>
    <div class="page-shell">
        <div class="grid-glow"></div>

        <nav class="nav">
            <a class="nav-brand" href="/">
                <img src="logo-new-128.png" alt="Chill Pill logo">
                <span>Chill Pill</span>
            </a>
            <div class="nav-links">
                <a href="/">Home</a>
                <a href="/privacy.html">Privacy</a>
                <a href="/terms.html">Terms</a>
            </div>
        </nav>

        <main class="main">
            <div class="card">

                <!-- Invalid link state (shown when token missing or type != recovery) -->
                <div class="invalid-panel" id="invalid-panel">
                    <div class="icon">🔗</div>
                    <div class="card-title">Link expired</div>
                    <p>This password reset link has expired or is invalid. Open Chill Pill and use "Forgot password?" to request a new one.</p>
                    <a class="back-link" href="/">← Back to site</a>
                </div>

                <!-- Password form (shown when token is valid) -->
                <div id="form-panel">
                    <div class="card-title">Set new password</div>
                    <p class="card-copy">Enter a new password for your Chill Pill account.</p>

                    <form id="reset-form" novalidate>
                        <label>
                            <span class="label">New password</span>
                            <input type="password" id="password" placeholder="At least 6 characters" autocomplete="new-password">
                        </label>

                        <label>
                            <span class="label">Confirm password</span>
                            <input type="password" id="confirm-password" placeholder="Repeat your password" autocomplete="new-password">
                        </label>

                        <div class="error-msg" id="error-msg"></div>

                        <button class="submit-btn" type="submit" id="submit-btn">Set New Password</button>
                    </form>
                </div>

                <!-- Success state -->
                <div class="success-panel" id="success-panel">
                    <div class="icon">✅</div>
                    <div class="card-title">Password updated</div>
                    <p>Your password has been changed. Redirecting you to the homepage…</p>
                </div>

            </div>
        </main>

        <footer class="foot">
            <div class="foot-bar">
                <div>Chill Pill</div>
                <div class="foot-links">
                    <a href="/">Home</a>
                    <a href="/privacy.html">Privacy</a>
                    <a href="/terms.html">Terms</a>
                </div>
            </div>
        </footer>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.js"></script>
    <script>
        const SUPABASE_URL      = 'https://qnpnmxdvblygckzpofac.supabase.co';
        const SUPABASE_ANON_KEY = 'sb_publishable_0kLok6guLj8AaP3dFlHwKw_H0sYcNNM';

        const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

        const invalidPanel  = document.getElementById('invalid-panel');
        const formPanel     = document.getElementById('form-panel');
        const successPanel  = document.getElementById('success-panel');
        const resetForm     = document.getElementById('reset-form');
        const passwordInput = document.getElementById('password');
        const confirmInput  = document.getElementById('confirm-password');
        const errorMsg      = document.getElementById('error-msg');
        const submitBtn     = document.getElementById('submit-btn');

        function showError(msg) {
            errorMsg.textContent = msg;
            errorMsg.classList.add('is-visible');
        }

        function clearError() {
            errorMsg.textContent = '';
            errorMsg.classList.remove('is-visible');
        }

        function showInvalid() {
            formPanel.classList.add('hidden');
            invalidPanel.classList.add('is-visible');
        }

        function showSuccess() {
            formPanel.classList.add('hidden');
            successPanel.classList.add('is-visible');
            setTimeout(() => { location.href = '/'; }, 2000);
        }

        // Parse the URL hash fragment (#access_token=...&refresh_token=...&type=recovery)
        function parseHash() {
            const hash = window.location.hash.slice(1); // remove leading #
            const params = {};
            hash.split('&').forEach(pair => {
                const [key, ...rest] = pair.split('=');
                if (key) params[decodeURIComponent(key)] = decodeURIComponent(rest.join('='));
            });
            return params;
        }

        async function init() {
            const params = parseHash();

            if (!params.access_token || params.type !== 'recovery') {
                showInvalid();
                return;
            }

            // Authenticate the Supabase client with the recovery token
            const { error } = await supabase.auth.setSession({
                access_token:  params.access_token,
                refresh_token: params.refresh_token || '',
            });

            if (error) {
                showInvalid();
                return;
            }
        }

        resetForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            clearError();

            const password = passwordInput.value;
            const confirm  = confirmInput.value;

            if (password.length < 6) {
                showError('Password must be at least 6 characters.');
                return;
            }

            if (password !== confirm) {
                showError('Passwords don\'t match.');
                return;
            }

            submitBtn.disabled = true;
            submitBtn.textContent = 'Saving…';

            const { error } = await supabase.auth.updateUser({ password });

            if (error) {
                submitBtn.disabled = false;
                submitBtn.textContent = 'Set New Password';
                const msg = error.message || '';
                if (msg.toLowerCase().includes('expired') || msg.toLowerCase().includes('invalid')) {
                    showError('This link has expired. Request a new one in the app.');
                } else if (msg.toLowerCase().includes('same password')) {
                    showError('New password must be different from your current password.');
                } else {
                    showError('Something went wrong. Please try again.');
                }
                return;
            }

            showSuccess();
        });

        init();
    </script>
</body>
</html>
```

- [ ] **Step 2: Open the file in a browser and verify the invalid-link state**

Open `C:/Users/man4m/FreeFlow/getchillpill.xyz/reset-password.html` directly in a browser (file:// is fine for this check).

Expected: the "Link expired" panel is shown (no hash fragment → `showInvalid()` fires on load). The form panel is hidden.

- [ ] **Step 3: Verify the form state by appending a fake hash**

In the browser address bar, append a fake recovery hash:

```
reset-password.html#access_token=fake_token&refresh_token=fake_refresh&type=recovery
```

Expected: the form panel is shown ("Set new password" heading, two password fields, "Set New Password" button). The `init()` call will call `supabase.auth.setSession()` which will fail with an invalid token and call `showInvalid()` — that is correct behaviour for a fake token. The important thing is that the form renders before `setSession` resolves and then transitions to invalid.

Note: to see the form render briefly, you can comment out the `await supabase.auth.setSession(...)` block temporarily, verify the form looks right, then restore it.

- [ ] **Step 4: Verify client-side validation**

With the form visible (comment out `setSession` block as above):
- Submit with empty fields → error: "Password must be at least 6 characters."
- Enter `abc` in both fields → error: "Password must be at least 6 characters."
- Enter `abcdef` in password, `abcxyz` in confirm → error: "Passwords don't match."
- Enter `abcdef` in both fields → no client-side error, form submits (will hit Supabase which will fail with invalid session — that's fine)

Restore the `setSession` block after verifying.

- [ ] **Step 5: Commit**

```bash
cd "C:/Users/man4m/FreeFlow/getchillpill.xyz"
git add reset-password.html
git commit -m "feat: add reset-password.html for Supabase password recovery flow"
```

---

## Task 2: End-to-end smoke test with a real recovery email

- [ ] **Step 1: Trigger a real recovery email**

Open Chill Pill on Windows. On the sign-in screen, click "Forgot password?" and enter the dev account email. A recovery email will arrive from Supabase (check spam if needed).

- [ ] **Step 2: Click the link in the email**

The link opens `https://getchillpill.xyz/#access_token=...&type=recovery` in a browser. 

Expected: `reset-password.html` does NOT currently handle this URL — the homepage loads. After deploying (Task 3), this will load `reset-password.html` instead. For now, manually navigate to `https://getchillpill.xyz/reset-password.html#access_token=...&type=recovery` (copy the full hash from the email link and append it).

- [ ] **Step 3: Set a new password**

Enter a new password in both fields, click "Set New Password".

Expected:
- Button shows "Saving…" while the request is in flight
- Success panel appears: "Password updated. Redirecting you to the homepage…"
- After 2 seconds, browser redirects to `https://getchillpill.xyz/`

- [ ] **Step 4: Verify sign-in with new password**

Open Chill Pill. Sign in with the dev account using the new password. Expected: sign-in succeeds.

- [ ] **Step 5: Test expired link**

Wait for the recovery token to expire (Supabase tokens expire in 1 hour by default), then try to use the same link again.

Expected: "Link expired" panel shown (either from `setSession` failing or from missing/wrong token).

---

## Task 3: Update Supabase redirect URL + push to website

The Supabase "forgot password" email currently links to `https://getchillpill.xyz/#access_token=...`. Supabase uses the **Site URL** for these links. Since the site URL is already `https://getchillpill.xyz`, the recovery link already points to the root — and our page lives at `/reset-password.html`.

To make Supabase send users directly to `/reset-password.html`, update the redirect URL in Supabase:

- [ ] **Step 1: Update Supabase redirect URL**

Go to [Supabase Dashboard](https://supabase.com/dashboard) → project `qnpnmxdvblygckzpofac` → Authentication → URL Configuration.

In **Email Templates** → **Reset Password**, check what the redirect URL is set to. It should be `{{ .SiteURL }}`. 

In **URL Configuration → Redirect URLs**, add:
```
https://getchillpill.xyz/reset-password.html
```

Then update the **Reset Password** email template's redirect to:
```
https://getchillpill.xyz/reset-password.html
```

Save.

- [ ] **Step 2: Push website to GitHub**

```bash
cd "C:/Users/man4m/FreeFlow/getchillpill.xyz"
git push origin main
```

Expected: push succeeds. The page is live at `https://getchillpill.xyz/reset-password.html`.

- [ ] **Step 3: Verify live page loads**

Open `https://getchillpill.xyz/reset-password.html` in a browser (no hash).

Expected: "Link expired" panel shows — correct, because there's no token in the URL.

- [ ] **Step 4: Trigger another recovery email and test the full flow end-to-end**

Repeat Task 2 steps 1–4, but this time click the link directly from the email. With the Supabase redirect URL updated, the link should open `reset-password.html` directly with the hash fragment already populated.

---

## Self-Review

**Spec coverage check:**
- ✅ Parses hash fragment for `access_token`, `refresh_token`, `type`
- ✅ Validates `type=recovery` and `access_token` presence → `showInvalid()` 
- ✅ Calls `supabase.auth.setSession()` before `updateUser()`
- ✅ `updateUser({ password })` on submit
- ✅ Success state + redirect after 2s
- ✅ Invalid link state (no token, wrong type, setSession error)
- ✅ Passwords don't match → inline error
- ✅ Password too short (< 6 chars) → inline error
- ✅ Supabase error (expired/invalid) → user-friendly message
- ✅ Network/unknown error → "Something went wrong"
- ✅ Button disabled + "Saving…" during submit
- ✅ Styling matches `feedback.html` (same CSS variables, card, inputs, Inter font)
- ✅ No server-side logic — purely client-side
