# Release notes

What's new in each version of Wren. Wren was called Nodal until 0.5.0, so the notes below for 0.4.0 and earlier describe Nodal. Downloads are on the
[releases page](https://github.com/dezluan/wren/releases).

## 0.4.0 · 10 November 2026

Wren 0.4.0 makes Wren easier to read and use, and lets you make it your own.

### New
- **Accessibility settings** (Settings → Accessibility):
  - **Text size**, from Small to Largest. Shortcuts: ⌘+ bigger, ⌘− smaller, ⌘0 default.
  - **Reduce motion**, **Reduce transparency** and **Increase contrast**. Each follows your Mac's
    own accessibility settings unless you change it.
  - **Underline links** and **bolder keyboard focus**.
- **Better with VoiceOver and the keyboard:** every icon button and dialog now has a name, the
  sidebar, conversation and thread are named areas, and notices are read out. The focused control
  is always outlined, and message actions appear when you tab to a message.
- **Company logo:** pick an image in Settings → General to show it next to the name at the top of
  the sidebar. It stays on your Mac.
- **Feedback straight to GitHub:** 💬 next to Settings opens a pre-filled issue on this
  repository, with your Wren and macOS versions. You need a GitHub account, and issues are public.
- **A new login screen:** a gently moving network of dots behind the sign-in card. It stays still
  when Reduce motion is on.

### Fixed
- The version line in feedback said "Node" instead of "Wren".
- Wren now logs why a saved sign-in was removed, to help track down cases where people had to
  sign in again.

## 0.3.1 · 10 November 2026

The first public release.

- Direct messages, group chats and channels, with threads, @mentions, reactions (hover to see who
  reacted) and unread state kept in step with Teams.
- An Unreads page (⌘⇧A), a ⌘K switcher, desktop notifications and a Dock badge.
- Images and GIFs: paste a screenshot or a Giphy link, pick from recent and favourite GIFs, and
  click any image to see it large.
- A built-in task list: turn any message into a task.
- Call and video buttons that start a Teams call with everyone in the chat.
- Glass and Classic looks, in light and dark.
