# Logic Encoder Login System

Site-wide **identity, email, and content gating** for logicencoder.com — branded transactional mail, SMTP deliverability fixes, login protection, per-post access rules, and protected file downloads.

Private plugin: [logicencoder/logicencoder-login-system-plugin](https://github.com/logicencoder/logicencoder-login-system-plugin). Page templates: [logic-encoder-theme](https://github.com/logicencoder/logic-encoder-theme) — [theme overview](https://github.com/logicencoder/logic-encoder-theme-overview).

## Visitor flows

- Branded login overlay when viewing restricted posts
- Registration, email verification, password reset, newsletter unsubscribe URLs
- Forced login before downloading protected attachments
- Default redirect after login: **`/dashboard/`** (configurable)
- Search engines may still index unrestricted content when restriction meta allows SEO visibility

## Content restriction

Per-post meta marks members-only content. `the_content` filter and upload guards enforce access at read time. Admin dashboard shows restriction statistics across the site.

## Email system

Skinned HTML templates for transactional mail. PHPMailer tuning (`phpmailer_init`, `wp_mail_*` filters). Send logging with admin AJAX refresh. Debug table viewer and orphaned-email cleanup tools for support incidents.

## Security hooks

Failed login tracking, authenticate filter, profile field extensions, attachment link protection on `template_redirect`. Works with [le-settings-plugin](https://github.com/logicencoder/le-settings-plugin-overview) Telegram alerts for complementary site-level lockout.

## Admin menu

**Logic Encoder** submenu pages:

| Page | Purpose |
|------|---------|
| Main settings | SMTP, branding, redirect defaults |
| Users | Quick user management links |
| Email templates | HTML skins per mail type |
| Email logs | Delivery history with AJAX refresh |
| Content restriction | Usage stats |
| Debug | Table viewer, diagnostics |

Dashboard widget for at-a-glance mail queue health.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
