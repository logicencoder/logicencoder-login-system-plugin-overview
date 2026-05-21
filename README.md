# Logic Encoder Login System — public overview

WordPress plugin that powers **registration, login, gated content, and transactional email** for [logicencoder.com](https://logicencoder.com) — the main LogicEncoder marketing and member surface, separate from the Telegram shop stack.

**Implementation (private):** [logicencoder-login-system-plugin](https://github.com/logicencoder/logicencoder-login-system-plugin)

---

## The problem

LogicEncoder ships downloadable tools and member-only material. WordPress default email and login flows are insufficient:

- Core password-reset and email-change messages do not match brand HTML or SMTP reality on Hostinger.  
- Operators cannot see **why** a welcome email failed without reading raw server logs.  
- Marketing pages must gate **posts and downloads** without breaking SEO for Google/Bing.  
- Disposable email domains and brute-force logins are ongoing abuse vectors on public crypto sites.

A single plugin centralizes identity UX, deliverability fixes, and restriction rules so themes and other plugins do not fork partial solutions.

---

## Who benefits

| Audience | Benefit |
|----------|---------|
| **New member** | Branded welcome/verification mail, clear login overlay on premium posts |
| **Returning member** | Reliable password reset, download access after login |
| **Site operator** | Template editor, live email logs, debug trail, cleanup tools |
| **SEO / marketing** | Search bots can still read restricted posts where configured — humans must log in |

---

## How this fits the stack (read before confusing plugins)

| Piece | Role |
|-------|------|
| **logicencoder-login-system-plugin** (this overview) | Login, email, content gates, downloads |
| **le-shop-plugin** | Telegram store **catalogue** (application CPT + webhook) |
| **le-settings-plugin** | Site SEO meta, security headers, unrelated shop settings |
| **le-crypto-app-store** (SOL) | Orders and chain payment — not WordPress login |

If documentation mentions “shop webhook” or “application post type”, that is **le-shop**, not this plugin.

---

## Capabilities (detailed)

### Branded transactional email

**What:** HTML templates for welcome, password reset, email verification, email change notice, test send, newsletter subscription, and contact-form notifications. Templates editable in wp-admin with variable placeholders (site name, user name, links).

**Why:** Default WordPress plain-text mail lands in spam and erodes trust for a crypto product brand.

**Who benefits:** New users receiving onboarding mail; operator A/B testing copy without code deploy.

### Dual “From” identity (transactional vs contact)

**What:** `wp_mail` filters switch between `noreply@…` style sender for account mail and `contact@…` for inbound form notifications.

**Why:** SPF/DMARC alignment and recipient expectations differ between system alerts and “someone filled the contact form” messages.

**Who benefits:** Operator deliverability; recipients replying to the correct mailbox.

### SMTP / PHPMailer hardening

**What:** Optional SMTP block in settings (host, port, TLS, credentials). `phpmailer_init` fixes hostname issues common on shared hosting; success and failure hooks write structured logs.

**Why:** Hostinger’s default `mail()` path is fragile for bulk or HTML messages.

**Who benefits:** Operator diagnosing “mail works in test but not registration”; support reducing ticket noise.

### Email log database and live admin UI

**What:** Every send attempt stored in `logicencoder_email_logs` with status SUCCESS/FAILED, email type, server_info JSON, and whether the address already had a WP user. Admin **Email Logs** page polls via AJAX for new rows without refresh.

**Why:** “User says they never got email” requires timestamped evidence, not grep across `/var/log`.

**Who benefits:** Support; operator during launch spikes.

### Debug log table

**What:** `logicencoder_debug` captures INIT/SMTP milestones and contextual messages; viewable and truncatable in admin.

**Why:** Separates noisy diagnostic lines from customer-facing email log history.

**Who benefits:** Developer maintaining the plugin; operator after plugin update.

### Login brute-force limits

**What:** `authenticate` filter tracks failed attempts per policy (`max_login_attempts`, `lockout_duration`); cleared on successful `wp_login`.

**Why:** Public login URLs on crypto sites attract credential stuffing.

**Who benefits:** Site owner avoiding compromised accounts; legitimate users protected by temporary lockout rather than silent failure.

### Password and registration policy

**What:** Configurable minimum length, letter/number/special character rules, disposable-domain blocklist, optional marketing consent checkbox.

**Why:** Reduce low-quality signups and compliance with newsletter consent expectations.

**Who benefits:** Operator list quality; legal clarity for EU-style consent.

### Per-post content restriction

**What:** Meta box on posts/pages: “restrict content”. Logged-out visitors see themed overlay with inline login form instead of body; logged-in users see full content.

**Why:** Premium articles and tool documentation should tease value without exposing full text to anonymous scrapers.

**Who benefits:** Member-only education content; operator monetization funnel toward registration.

### Search-engine bot exception

**What:** `logicencoder_is_search_bot()` allows listed crawlers to receive unrestricted `the_content` for SEO when restriction meta is enabled.

**Why:** Hard paywalls harm indexing; LogicEncoder needs discoverability for crypto keywords.

**Who benefits:** Marketing/SEO; Google-visible snippets while humans still gate.

### Download and uploads protection

**What:** `template_redirect` and `init` rules redirect anonymous users to `/login/` when accessing attachments, common file extensions in URLs, or files under `wp-content/uploads/` (pdf, zip, office docs, csv, etc.). `attachment_link` filter rewrites links for logged-out users to login with `redirect_to`.

**Why:** Direct file URLs bypass post content filters — common leak path for zip assets.

**Who benefits:** Operator protecting paid/free member downloads; members after single sign-in.

### User management and orphan cleanup

**What:** Admin screens to inspect whether an email exists in `users`, usermeta orphans, and tools to delete stale rows safely.

**Why:** Failed registrations and imports leave half-created users that block re-signup with same email.

**Who benefits:** Support clearing “email already registered” false positives.

### Unsubscribe handling

**What:** Dedicated `init` route for newsletter unsubscribe links in mail footers.

**Why:** CAN-SPAM/GDPR-style opt-out must work without manual admin removal.

**Who benefits:** Newsletter recipients; operator compliance.

### Dashboard widget

**What:** wp-admin summary of recent email activity and restriction counts.

**Why:** Quick health check when operator logs in for unrelated tasks.

**Who benefits:** Operator daily workflow.

### User profile extensions

**What:** Extra fields on user profile screens in admin (marketing consent, custom meta — see private ARCHITECTURE).

**Why:** Central place to audit member state without SQL.

**Who benefits:** Operator reviewing account before manual support reply.

### Contact form notification API

**What:** `send_contact_form_notification()` wraps plain body to HTML and sends with contact From profile — callable from theme/forms.

**Why:** Contact page should not duplicate mail formatting logic in theme files.

**Who benefits:** Developer wiring forms; operator consistent inbox labeling.

### Disable core email-change notification

**What:** Filters off WordPress default email-change email in favour of branded template.

**Why:** Avoid duplicate/confusing messages when user updates email in account settings.

**Who benefits:** Members changing email once, with one branded explanation.

---

## Security and privacy posture (public summary)

- Write operations (settings, logs truncate, cleanup) require administrator capability.  
- Public visitors cannot read email logs via AJAX.  
- Error messages intentionally omit WordPress `admin_email` leakage.  
- Live SMTP passwords and API secrets stay in private repo / wp_options only — not in this overview.

---

## What this overview does not include

- Full PHP source, SMTP passwords, or reCAPTCHA secret keys  
- Telegram shop order flow (`le-crypto-app-store`)  
- Site-wide SEO schema from `le-settings-plugin`

---

## Operator scenarios

| Scenario | Where to look |
|----------|----------------|
| Welcome mail not received | Email Logs → FAILED row → server_info |
| User cannot download zip | Confirm logged in; check restriction meta on parent post |
| Lockout complaints | Users → attempt settings; adjust `lockout_duration` |
| SEO drop on gated article | Confirm bot exception still matches crawler User-Agent list |

---

## Related repositories

See [REPOS.md](REPOS.md).

---

## Design philosophy

The plugin intentionally uses a **singleton** class plus a small set of procedural hooks for content restriction so only one instance runs — avoiding duplicate `wp_mail` filters when copies were loaded during past refactors. New features should extend the class or documented hooks rather than spawning second plugins that fight over `phpmailer_init`.

---

**Made by [logicencoder](https://github.com/logicencoder)**
