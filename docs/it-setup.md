# Setting up Nodal for your organisation

Nodal is a Mac app for Microsoft Teams chats, built on Microsoft Graph. To sign in, it needs an
app registration in your organisation's Microsoft Entra ID. This is a one-off task for an Entra
admin and takes about ten minutes.

## How Nodal signs in

- **Delegated permissions only.** Nodal acts as the signed-in user, and Microsoft enforces that
  user's normal access: Nodal can only see what they can see in Teams. There are no application
  permissions.
- **No client secret.** Nodal is a public client, using the authorisation code flow with PKCE.
- **Standard Entra ID sign-in** in the user's browser, so MFA, Conditional Access and sign-in
  policies apply as usual.
- **No Nodal server, no telemetry.** The app talks to Microsoft, plus Giphy for GIFs (see
  [Network access](#network-access)). The [security overview](../SECURITY.md) covers data flows,
  token storage and how message content is handled.

## Create the app registration

1. In the Entra admin center, go to **App registrations → New registration**. Name it `Nodal`
   and choose *Accounts in this organizational directory only*.
2. Under **Authentication → Add a platform → Mobile and desktop applications**, add the custom
   redirect URI **`http://localhost`**. Entra accepts any port on localhost, and Nodal picks a
   free one each time.
3. Under **API permissions → Add a permission → Microsoft Graph → Delegated permissions**, add:

   | Permission | Used for |
   | --- | --- |
   | `User.Read` | The signed-in user's profile |
   | `User.ReadBasic.All` | Names and photos of colleagues, and the people picker |
   | `Chat.ReadWrite` | Reading chats, marking them read, reactions |
   | `ChatMessage.Send` | Sending chat messages |
   | `Team.ReadBasic.All` | Listing the user's teams |
   | `Channel.ReadBasic.All` | Listing channels |
   | `ChannelMessage.Read.All` | Reading channel messages (optional, see below) |
   | `ChannelMessage.Send` | Posting in channels |
   | `Presence.Read.All` | Presence dots (Available, Busy, Away) |
   | `offline_access` | Staying signed in |

4. Click **Grant admin consent** for your organisation. `ChannelMessage.Read.All` always needs
   admin consent, and granting everything up front means users never see a permissions prompt.
5. *Optional:* in **Enterprise applications → Nodal → Properties**, set **Assignment required**
   to **Yes**, then assign the users or groups allowed to use Nodal.

**Channels are optional.** If you'd rather not grant `ChannelMessage.Read.All`, leave it out.
Users then untick **Read Teams channels** on the login screen, and chats and DMs work as normal.

## Give users the two values

From the app registration's **Overview** page, send users:

- the **Application (client) ID**
- the **Directory (tenant) ID**, or your domain, such as `contoso.com`

Users enter them once on Nodal's login screen, under **Advanced**, together with whether to read
channels. They're identifiers for a public client, not secrets, so it's fine to share them by
email or on your intranet. The user guide is the [README](../README.md#sign-in).

## Optional: sign in with a code

The login screen also offers **Sign in with a code instead**, using the device code flow. It
needs **Authentication → Advanced settings → Allow public client flows: Yes**. It won't work if
Conditional Access blocks the device code flow, which Microsoft recommends, and most people won't
need it.

## Coming later

These features will ask for extra permissions once they're released. Each is requested only when
a user turns it on, so you can approve them later:

- `Calendars.ReadWrite`: show the user's Outlook calendar, answer invitations, create meetings.
- `Files.ReadWrite.All`: send file attachments to the user's OneDrive or the channel's SharePoint
  folder, as Teams does.
- `Tasks.ReadWrite`: sync Nodal's task list with Microsoft To Do.

## Deploying Nodal

- **Manual install:** users download the DMG from the
  [releases page](https://github.com/dezluan/nodal/releases/latest) and follow the
  [install steps](../README.md#install). Nodal is ad-hoc signed and not notarised yet, so the first
  launch needs **Open Anyway** in System Settings → Privacy & Security.
- **Managed install (Intune, Jamf and similar):** deploy `Nodal.app` from the DMG into
  `/Applications`. Apps installed by an MDM usually aren't quarantined, so users don't see the Gatekeeper
  step. The bundle id is `app.nodechat.node`.
- **Updates** are manual for now: there's no auto-updater. Watch the releases page (GitHub's
  **Watch → Custom → Releases**) to hear about new versions.

## Network access

Nodal makes outgoing HTTPS connections only, to:

| Host | Why |
| --- | --- |
| `login.microsoftonline.com` | Sign-in and token refresh |
| `graph.microsoft.com` | Chats, channels, messages, presence, photos, images you send |
| `media.giphy.com`, `media0`–`media4.giphy.com`, `i.giphy.com` | Loading GIFs shown in chats and GIFs you send. No token or cookies are sent. |

Sign-in completes through a one-off listener on `localhost` (`127.0.0.1` / `::1`), on a random
port, which only accepts Microsoft's redirect. Links people click, including Teams calls and
meetings, open in the user's browser, not in Nodal.

## What Nodal stores on the Mac

| Where | What |
| --- | --- |
| Keychain item `app.nodechat.node` | The refresh token, so users stay signed in. Removed on sign-out. |
| `~/Library/Application Support/Nodal/config.json` | Settings the user changed on the login screen or in Settings |
| `~/Library/WebKit/app.nodechat.node` | Interface preferences, custom sidebar sections, drafts, tasks, recent and favourite GIFs |
| `~/Library/Logs/Nodal/nodal.log` | A small diagnostic log (about 2 MB at most), with no message text, names or tokens |

Messages and images are kept in memory only. The one exception is tasks: a task made from a message
keeps its text.

## Removing access

Delete the app registration, or remove individual users in **Enterprise applications → Nodal →
Users and groups**. Users can also remove the grant themselves at
[myapps.microsoft.com](https://myapps.microsoft.com).
