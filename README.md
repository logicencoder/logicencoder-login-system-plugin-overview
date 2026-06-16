# Logic Encoder Login System — WordPress plugin

**Logic Encoder Login System** is the member identity layer for [logicencoder.com](https://logicencoder.com): branded login and registration, email verification, password reset, HTML transactional mail, per-page content gating, and protected downloads — configured in wp-admin and rendered through the [Logic Encoder theme](https://github.com/logicencoder/logic-encoder-theme-overview) page templates.

Members sign in at **`/login/`**, land on **`/dashboard/`** after activation (path is configurable), and manage passwords on **`/account/`**. Operators control SMTP sender identity, email skins, password rules, lockouts, and which pages require a logged-in reader.

- **Branded auth pages** — sign in, create account, forgot password, reset password, email verification URLs.
- **HTML email templates** — welcome, verification, password reset, email change, newsletter, and one-off **Write Email**.
- **Per-page members-only content** — teaser, fade, and inline login card on restricted pages and custom post types.
- **Protected downloads** — PDFs, archives, and uploads redirect guests to login with a return URL.
- **User roster** — marketing consent, verification state, resend activation, manual activate.
- **Email delivery log** — success/failure history, SPF/DKIM/DMARC checker, test sends.
- **Support tools** — debug event log, orphaned-email cleanup.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP single-file (`logicencoder-login-system.php`, ~4.4k LOC), inline admin CSS |
| Persistence | WordPress options + user meta |
| Auth UI | Theme templates (`page-login.php`, `page-account.php`, …) driven by plugin settings |
| Email | `wp_mail` with branded HTML wrapper, optional Hostinger SMTP, dual sender profiles (transactional vs contact) |
| Security | Login attempt limits, disposable-domain block, optional login event logging |
| Content gating | Per-post editor meta box, content overlay, attachment and upload-path guards |
| Hosting | WordPress on shared hosting; wp-admin + public member URLs |

## Main settings

Top-level wp-admin menu **Logic Encoder** (lock icon). The default **Main settings** screen shows a status banner (plugin active, user counts, mail readiness) and **Save Settings**.

| Section | What you configure |
|---------|-------------------|
| **Main** | Redirect after login (default `/dashboard/`), require marketing consent on registration |
| **Password requirements** | Minimum length, letters, numbers, special characters |
| **Email** | From email/name, contact-form From identity, SMTP host/port/encryption/credentials, sender logo upload, welcome subject line |
| **Security** | Max login attempts, lockout duration, block temporary email domains, custom blocked domains list, enable login logging |

Quick links at the foot of the screen jump to login URL, dashboard URL, and other Logic Encoder submenus.

To send branded verification mail, set **From name** and **From email** to your domain, enable **SMTP**, open **Email Templates** → **Email Verification**, edit the copy, send **Quick Test**, and confirm delivery in **Email Logs**.

## Users

The **Users** screen lists every WordPress account with Logic Encoder fields: registration IP, country, email verified flag, and marketing consent.

| Column / action | Use |
|-----------------|-----|
| Sortable table | ID, name, email, role, registered date, IP, country, verified, consent |
| **Edit** | Opens the standard WordPress user profile |
| **Resend Email** | Sends verification mail again |
| **Activate** | Marks account verified without waiting for the link click |
| **Delete** | Removes non-admin users |

For a stuck registration, find the row → **Resend Email**; if the address is orphaned, use **Database Cleanup** first, then ask the member to register again.

## Email templates

**Email Templates** splits the screen: template tabs on the left, editor on the right, **Quick Test** at the bottom.

| Tab | Typical use |
|-----|-------------|
| **Write Email** | One-off HTML message to a chosen address |
| **Welcome Email** | After verification succeeds |
| **Password Reset** | Forgot-password flow |
| **Email Change** | Account email change notice |
| **Email Verification** | Link to `/verify-email/` |
| **Test Email** | Template used for admin test sends |
| **Newsletter Subscription** | Footer/widget signup verification |

Each tab exposes merge chips such as `{name}`, `{email}`, `{verification_url}`, `{reset_url}`, `{dashboard_url}`, `{unsubscribe_url}`, `{site_name}`, `{date}` — click to insert into subject or body. **Save Template** per type; **Send Test Email** at the bottom fires a live message to any inbox you type.

To refresh post-reset copy, edit **Password Reset** subject and body → **Save Template** → **Quick Test** to your Gmail → check **Email Logs** for success.

## Email logs

**Email Logs** is the deliverability desk. A test box sends arbitrary messages; the **SPF / DKIM / DMARC** panel checks the From domain and surfaces Hostinger setup hints.

Filter cards by **type** (test, registration, verification, reset, welcome, newsletter, …), **status** (success/failed), **email address**, and **limit** (50–500). Each card expands to server response detail. **Clear All Logs** wipes history.

When mail stops arriving, filter **Failed** → read the expanded error → fix SMTP or DNS → send **Test Email** again.

## Content restriction

**Content Restriction** shows counts of restricted posts, pages, and downloads, plus global tuning fields:

| Setting | Effect |
|---------|--------|
| **Enable Content Restriction** | Master toggle stored for the feature |
| **Preview Length** | Character count for teaser text |
| **Fade Delay** | Milliseconds before the gradient overlay appears |
| **Restrict All Downloads** | Policy flag for download gating |

The **Restricted Content List** table lists every item with the restrict flag: title, type, status, **Edit**, **View**.

In the **page/post editor**, the **Content Restriction** sidebar meta box sets **Restrict to registered users only** and **Preview Height (pixels)** per item. Blog posts stay public; pages and custom post types can be members-only.

Logged-out visitors see a partial article, fade, and an inline **Restricted Content** card with email, password, **Sign In**, and **Register here** (return URL preserved). Search crawlers on the built-in bot list still receive full HTML for indexing.

For a members-only tool page, edit the MEXC dashboard page → check **Restrict to registered users only** → set preview height → publish. Anonymous readers see the teaser and login card; members see the full page after signing in at `/login/`.

## Debug logs and database cleanup

**Debug Logs** aggregates registration events, login logging (when enabled), and plugin errors. Stats show totals by type; **Clear All Logs** and **Refresh** maintain the table.

**Database Cleanup** helps support incidents:

| Tool | Use |
|------|-----|
| **Email Existence Checker** | See whether an address is tied to a pending or broken record |
| **Email Cleanup Tool** | Remove orphaned registration rows so the address can register again (destructive — confirm dialog) |

## Public member flows

These URLs live on logicencoder.com via the theme; the plugin supplies settings, mail, and gating rules.

### Login and registration (`/login/`)

**Sign in** tab: email, password, show/hide toggle, **Keep me signed in**, **Forgot password?** link. Unverified accounts cannot log in until they click the verification link.

**Create account** tab: name, email, password, confirm password, live checklist against your password rules, optional **marketing consent** checkbox. Submit stays disabled until requirements pass. Success sends verification mail — no automatic login.

Already logged in: administrators go to wp-admin; everyone else to the configured dashboard path.

### Email verification (`/verify-email/`)

Visitor clicks the link from mail → account marked verified → **welcome email** sends with dashboard and unsubscribe links → redirect to login after a short pause.

### Forgot and reset password

**Forgot password** (`/forgot-password/`): enter email → branded reset mail with link to **`/reset-password/`** → set a new password meeting your rules → auto-redirect to login.

### Protected downloads

Logged-out visitors hitting attachment pages, `?download=` links, or direct URLs to PDF/ZIP/DOC/XLS/CSV under uploads are sent to **`/login/?redirect_to=…`** and return to the file after sign-in.

To lock a PDF behind login, upload to Media Library and share the file URL — guests are prompted to sign in before the download starts.

### Unsubscribe

Welcome mail includes `{unsubscribe_url}`. Visiting it removes the account and shows a confirmation with a homepage link.

### Account and contact

**`/account/`** (theme) requires login; password changes enforce the same rules as registration. Email change triggers the **Email Change** template.

The contact page sends through the **contact-form** sender profile configured separately from transactional `noreply` mail.

## Dashboard widget

The WordPress home **Dashboard** widget **Logic Encoder User Stats** shows total users, marketing-consent count, manual registrations, and a link to plugin settings — member growth at a glance without opening the Users screen.

Private code: [logicencoder/logicencoder-login-system-plugin](https://github.com/logicencoder/logicencoder-login-system-plugin) (v2.2.x)

Theme templates: [logic-encoder-theme-overview](https://github.com/logicencoder/logic-encoder-theme-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
