# Logic Encoder Login System

Site-wide **identity, email, and content gating** for logicencoder.com — branded transactional mail, SMTP fixes, login protection, per-post access rules, and protected uploads.

Private plugin: [logicencoder/logicencoder-login-system-plugin](https://github.com/logicencoder/logicencoder-login-system-plugin). Page shells: [logic-encoder-theme](https://github.com/logicencoder/logic-encoder-theme) — [theme overview](https://github.com/logicencoder/logic-encoder-theme-overview).

## The problem it solves

Default WordPress login and `wp_mail` deliverability are not enough for a branded membership site. Operators need restricted posts, skinned auth flows, email templates, and attachment protection without bolting on a separate SaaS auth product.

## Visitor flows

- Branded login overlay on restricted posts; forced login for protected attachments
- Registration, verification, password reset, newsletter unsubscribe URLs
- Search engines may still index unrestricted content when restriction meta allows SEO visibility

Default post-login redirect: **`/dashboard/`** (configurable).

## Content restriction

Per-post meta marks content as members-only. `the_content` and upload guards enforce access. Admin stats show restriction usage across the site.

## Email system

Skinned HTML templates for transactional mail, PHPMailer tuning, send logging, debug table viewer, orphaned-email cleanup. Admin AJAX refresh for email logs.

## Admin

**Logic Encoder** menu: Main settings, Users, Email templates, Email logs, Content restriction stats, Debug tools. Dashboard widget for quick status.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
