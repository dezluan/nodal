# Wren: security overview

This document is for IT and security reviewers. Wren is a desktop client for Microsoft Teams:
a Rust backend (Tauri v2) with a bundled HTML/JS user interface. It talks only to Microsoft
endpoints and has no backend server of its own.

## Data flows

| From → To | What | How |
| --- | --- | --- |
| Wren → `login.microsoftonline.com` | Sign-in, token refresh | OAuth 2.0 auth code + PKCE (S256) |
| Wren → `graph.microsoft.com/v1.0` | Chats, channels, messages, presence, photos, inline images you send | HTTPS (rustls), bearer token |
| Wren → `*.sharepoint.com` | Files you attach, only if file uploads are on | Graph upload-session URL (pre-authorised, no token sent) |
| Wren → `media.giphy.com`, `i.giphy.com` | GIFs shown in chats and GIFs you send | HTTPS, no token or cookies; redirects only to Giphy |
| Wren → your browser → `github.com` | Feedback you choose to send | A pre-filled public issue page; you press Create yourself |
| Wren → system browser | Sign-in page; links you click | macOS `open` |
| Wren → macOS Notification Center | New-message banners (sender, chat name, short preview) | UserNotifications framework, with user permission |
| Microsoft → Wren (localhost) | OAuth redirect | One-shot listener on `127.0.0.1`/`::1`, random port |

No telemetry or analytics. Apart from Giphy for GIFs, there are no third-party services, and
nothing is sent anywhere else.

## Identity and permissions

- **Delegated permissions only.** Wren acts as the signed-in user, and Microsoft Graph enforces
  that user's normal access on every request. There are no application permissions and no
  client secret (Wren is a public client).
- Requested scopes: `User.Read`, `User.ReadBasic.All`, `Chat.ReadWrite`, `ChatMessage.Send`,
  `Team.ReadBasic.All`, `Channel.ReadBasic.All`, `ChannelMessage.Read.All` (optional), `ChannelMessage.Send`, `Presence.Read.All`, `offline_access`. Wren doesn't request
  mail or directory-write access, and asks for calendar access only when you turn it on (below).
- **Files (opt-in):** `Files.ReadWrite.All` is requested only when a user turns on *Settings → Files*. Until then Wren has no file access. Wren uses
  it only to upload files the user attaches. Chat files go to the user's OneDrive folder
  "Microsoft Teams Chat Files" and are shared (edit, sign-in required, no email) with that chat's
  members, as Teams does. Channel files go to the channel's SharePoint folder, which channel
  members can already reach. Wren never lists, reads or deletes other files. If the tenant
  requires admin consent, Microsoft shows *Need admin approval* and nothing changes until an admin
  approves.
- **Calendar (opt-in):** `Calendars.ReadWrite` is requested only when a user clicks *Connect
  calendar* (Calendar view or *Settings → Calendar*). Wren uses it to show the user's own events
  for the week on screen, to accept, tentatively accept or decline invitations (the organiser is
  told, as in Outlook), and to create meetings the user fills in. It never deletes or edits
  existing events and doesn't read other people's calendars. Invitation bodies are shown as
  Outlook's plain-text preview only, never as HTML. Join links are opened only if they point at
  `https://teams.microsoft.com/`. Turning the calendar off in Settings stops Wren using it; the
  grant itself can be removed in Entra ID or at myapps.microsoft.com.
- Your **Conditional Access, MFA and sign-in policies** apply as normal. Admins can restrict the
  app to assigned users and revoke it at any time in Entra ID.

## Tokens and local data

- The **refresh token** is stored in the macOS **Keychain** and deleted on sign-out or when
  Microsoft rejects it. **Access tokens** live in memory only.
- Tokens are never written to logs. Wren keeps a small diagnostic log at
  `~/Library/Logs/Wren/wren.log`, capped at about 2 MB. It contains only errors, counts,
  conversation types and Graph resource ids, never message text, names or tokens.
- **Wren never writes messages or images to disk**, apart from tasks you choose to keep (below).
  They are held in memory, and image
  responses are sent to the webview with `Cache-Control: no-store`. Wren's image cache is
  bounded: 8 MB per image, and it is cleared once it passes 64 MB, so it peaks at about 72 MB.
  macOS's own WebKit may still keep transient data, such as rendered pages, the way any
  browser does.
- The UI's local storage holds:
  - preferences (theme, accessibility options, starred and collapsed sections, your own sidebar
    sections as names plus conversation ids, last open conversation)
  - the company logo you picked, as a 96 px PNG
  - recent and favourite GIFs, as Giphy ids
  - names of people seen in each channel, for @mention suggestions
  - the notification bell's list: who, where and when, but **not** the message preview, which
    stays in memory. It's cleared on sign-out.
  - **Tasks**: the tasks you add, including message text when you use *Add to tasks*. That's the
    point of a task list; they stay until you delete them.
- `~/Library/Application Support/Wren/config.json` holds any tenant or client ID the user
  overrides. These are public identifiers, not secrets.

## Sign-in

- The **default flow is authorization code + PKCE** with a loopback redirect
  (`http://localhost:<random port>`), as recommended for native apps (RFC 8252).
- A random 128-bit `state` is checked before the code is redeemed, and the PKCE verifier is
  384 bits. The listener binds to loopback only, answers only the expected request, applies a
  5-second deadline to each request and a 10-minute deadline overall, and closes when a newer
  attempt starts.
- The **device-code flow** is an opt-in fallback hidden under "Advanced". It works only if the app
  registration allows public client flows and Conditional Access doesn't block device code.

## Untrusted content (messages from colleagues and guests)

Message HTML is treated as hostile:

- **Sanitised in Rust** with [ammonia] (html5ever-based) before it reaches the UI.
  - Scripts, event handlers, styles, forms, iframes and `javascript:`/`data:` URLs are removed.
  - Links get `rel="noopener noreferrer"`.
  - Element `id`s from messages are dropped, so content can't clash with the app's own
    elements. The UI also holds direct references to its security-relevant elements, like the
    link tooltip.
  - Sender-supplied `title` attributes on links are dropped, and sender-supplied `class`
    attributes are stripped.
- **Mention highlighting can't be forged.** Mentions are styled only after sanitising, and
  only for `<at>` elements in the sanitiser's output that match the message's Graph `mentions`
  data. In that output a literal `<at` can only be a real element, so entity tricks and
  attribute hiding don't work.
- **No remote images.** Only images proxied through Wren can load, which blocks tracking pixels
  and read receipts. External images are shown as a link instead. This is enforced in the
  sanitiser and again in the CSP. The one exception is GIFs from Teams' GIF picker: a URL
  that matches Giphy's media hosts and path shape exactly is reduced to its Giphy id, and Wren
  fetches `https://media.giphy.com/media/{id}/giphy.gif` itself, with no token or cookies,
  redirects limited to Giphy, an 8 MB cap, and the bytes sniffed as GIF or WebP. Giphy sees
  your IP address and that a GIF was loaded, just as it does in Teams.
  Sending a GIF works the same way: a pasted giphy.com link is reduced to its id, Wren
  downloads the GIF (a smaller version if the original is over 4 MB) and sends it as an inline
  image. Recent and favourite GIFs are only ids, kept in local storage on this Mac.
- The **image proxy** (`graphimg://`) only fetches two strict path shapes: user photos and
  message hosted contents, plus `/giphy/{id}.gif` (see above). The path is fully percent-decoded and validated, dot-segments are
  rejected, and the final URL must parse to exactly the validated path. Responses must be
  `image/*` and are served with `nosniff` and a sandboxing CSP.
- **Clicking a link** opens it in the system browser. Only `http`, `https` and `mailto` are
  opened. The hover tooltip leads with the real host and makes hidden characters visible.
  Links that disguise where they go are refused: ones with a `user@` part
  (`https://contoso.com@evil.example`) or with invisible or right-to-left control characters.
  `mailto` links keep only `to`, `cc`, `subject` and `body`, so a link can't ask your mail app
  to attach a file.
- **File chips** show their host when the link isn't SharePoint or OneDrive, so a message
  can't pass off a web page as a Teams file.
- **The webview can't navigate away from the app's bundled pages.** A navigation guard refuses
  everything else, so remote content can never reach the app's IPC bridge.
- **Content Security Policy:**
  see `tauri.conf.json`. Scripts are allowed only from the app itself (`script-src 'self'`), images only from the app and `graphimg:`, and objects, frames, `<base>` and form submissions are blocked.
  There are no inline scripts and no `eval`.
- **Tauri capabilities** granted to the UI are only `core:default`, window dragging and
  notifications. The UI can't spawn processes or read files. It can ask the backend to open
  a link in the system browser, but only for `http`, `https` and `mailto`.

## Outgoing messages

What a user types is converted from Markdown to HTML in Rust:

- Typed HTML is shown literally, never interpreted.
- Only `http(s)` and `mailto` links are kept.
- Images become links, including images nested inside links.
- A link whose text could pass for an address but doesn't match its destination is rewritten
  to show the real destination. This covers leading spaces and lookalike dots such as `。`.
- Mentions are placed only in plain text, never inside links or code.

## Attachments

- Attachment bytes go from the UI to the backend as a raw IPC body and stay in memory (capped at
  250 MB per file and 512 MB in total) until the message is sent or the attachment is removed.
- A file is treated as an image only if its **bytes** are PNG, JPEG, GIF or WebP. The name and the
  type the webview reports are not trusted. Only those images (up to 4 MB) go inline; anything
  else, SVG included, is uploaded as a file.
- File names are reduced to one safe path segment before they're used in a OneDrive/SharePoint
  path. Drive and item ids are allowlisted.
- Upload-session URLs must be HTTPS on `*.sharepoint.com`. As Microsoft requires, no access token
  is sent to them.

## Requests the UI can make

The UI talks to the backend through a fixed set of commands. Every ID used in a Graph URL is
checked against a strict character allowlist, so paths and queries can't be injected. Paging
links are accepted only if they continue a Graph message listing. Paging-link validation
hasn't yet been tested against a live tenant; if Graph returns an unexpected link form, Wren
just stops loading older history rather than following it. All Graph traffic is pinned to
`https://graph.microsoft.com/v1.0/`.

## Supply chain

- Rust dependencies are locked in `Cargo.lock`. There are no npm or JavaScript dependencies;
  the UI is plain hand-written HTML/CSS/JS.
- `cargo audit` (RustSec database, 2026-10-30) reports **no vulnerabilities**. It shows 5
  warnings about unmaintained or unsound crates (`glib`, `zbus`/`zvariant`,
  `proc-macro-error`). These are only used in Linux builds and aren't compiled into the macOS
  app.

## Testing

The automated test suite covers the security-relevant logic:

- sanitiser bypass attempts
- tracking images and forged mentions
- Giphy URL matching (look-alike hosts, ports, credentials, traversal, other file types)
- image-proxy path traversal (`%2e%2e`, `%2f`, query injection)
- paging-link validation
- outgoing link and mention handling
- attachment checks (image sniffing, file names, upload hosts)
- calendar checks (Teams join links, meeting chat ids, event ids, new-meeting validation)
- the OAuth redirect listener (noise, stale `state`, errors, cancellation)
- the navigation guard

## Known limitations

- **Ad-hoc signed:** the app bundle is ad-hoc signed (identifier `app.nodechat.node`), but not
  signed with a Developer ID or notarised. For managed rollout, sign it with
  the organisation's Apple Developer ID or distribute it via Intune/Jamf.
- **Polling:** Wren polls Graph every few seconds, staying within Graph's throttling limits.
  It doesn't use change notifications, which would need a public webhook.

## Reporting

Please report security issues privately through this repository's **Security → Report a vulnerability** tab, not in a public issue.

[ammonia]: https://github.com/rust-ammonia/ammonia
