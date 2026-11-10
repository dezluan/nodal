<p align="center"><img src="docs/images/icon.png" width="96" alt="Nodal icon"></p>

<h1 align="center">Nodal</h1>

<p align="center">A fast, Slack-style Mac app for your Microsoft Teams chats.</p>

<p align="center"><a href="https://github.com/dezluan/nodal/releases/latest"><b>Download for macOS</b></a> · <a href="docs/it-setup.md">IT setup</a> · <a href="SECURITY.md">Security</a></p>

![Nodal showing a group chat](docs/images/chat.png)

Nodal talks to Teams through Microsoft Graph, so everything you send shows up in Teams for your
colleagues. You just never have to open Teams yourself. It's a native Mac app written in Rust:
it starts quickly, stays light on memory, and works the way Slack does.

Nodal is an independent app. It isn't made by or affiliated with Microsoft.

## What you get

- **Slack-style layout:** direct messages, group chats, meetings and one section per team with
  its channels. Star conversations, collapse sections, or make your own sections and drag chats in.
- **Unreads** (⌘⇧A): every chat with new messages on one page, showing just what's new.
  Unread state stays in step with Teams, with a Dock badge and desktop notifications.
- **Threads** in a side panel, like Slack.
- **A composer with Slack muscle memory:** `*bold*`, `_italic_`, `~strike~`, `` `code` ``,
  `:shortcodes:`, @mentions, Enter to send, and drafts kept per conversation.
- **Images and GIFs:** paste a screenshot to send it. Paste a Giphy link to send the GIF, or pick
  from GIFs in your chats and your favourites. Click any image to see it large.
- **Reactions:** click to react, hover to see who reacted.
- **Tasks:** a simple to-do list in the sidebar. Turn any message into a task, and the task
  links back to it.
- **Calls:** the 📞 and 🎥 buttons start a Teams call with everyone in the chat. The call itself
  runs in Teams, because Microsoft doesn't let other apps place calls.
- **⌘K** to jump to any chat, person or channel, and **⌘N** for a new message.
- **Glass or Classic look**, in light and dark, with several colour palettes.

**Coming soon:** your Outlook calendar, sending files (images already work), and syncing Tasks
with Microsoft To Do.

## Install

1. Download `Nodal_<version>_universal.dmg` from the
   [latest release](https://github.com/dezluan/nodal/releases/latest). It runs on Apple Silicon
   and Intel Macs.
2. Open the DMG and drag **Nodal** to **Applications**.
3. The first time, **right-click Nodal → Open**, then click **Open**. Nodal isn't notarised by
   Apple yet, so macOS asks once. If macOS still refuses, run
   `xattr -dr com.apple.quarantine /Applications/Nodal.app` in Terminal.

## Sign in

Nodal signs in with your work Microsoft account, in your browser, with your usual MFA. It needs
an **app registration** in your organisation's Microsoft Entra ID, which your IT team sets up
once. Send them the [IT setup guide](docs/it-setup.md).

IT then gives you two values. On Nodal's login screen, open **Advanced** and enter the
**Application (client) ID** and **Tenant**. If IT didn't approve channel reading, untick **Read Teams channels**. Then click **Sign in with Microsoft**. Nodal remembers these settings.

## Keyboard shortcuts

| Keys | Action |
| --- | --- |
| ⌘K / ⌘T | Jump to a conversation |
| ⌘N | New message |
| ⌘⇧A | Unreads |
| ⌥↑ / ⌥↓ | Previous / next conversation |
| ⌥⇧↑ / ⌥⇧↓ | Previous / next unread conversation |
| ⌘, | Settings |
| Esc | Close thread or dialog |
| Start typing | Focuses the composer |

## Privacy

Nodal talks only to Microsoft. There's no Nodal server, and no telemetry or analytics. Your
sign-in is kept in the macOS Keychain, and messages are never written to disk. The
[security overview](SECURITY.md) has the details.

## Known limitations

- You can't edit or delete your own messages from Nodal yet. ↗ opens the chat in Teams for that.
- Channels don't show unread state yet, because Graph has no cheap way to read it.
- No "Seen" receipts: Graph only shares your own read position, not other people's.
- New messages are checked every few seconds rather than pushed instantly.
- Your sections and tasks stay on one Mac and aren't synced.

## Feedback

Found a bug or have an idea? [Open an issue](https://github.com/dezluan/nodal/issues).
For security issues, see [SECURITY.md](SECURITY.md).

## Licence

Nodal is free to download and use. © 2026 Dez-Luan Pieterse. See [LICENSE](LICENSE).

Microsoft, Microsoft Teams and Microsoft Entra are trademarks of the Microsoft group of companies.
