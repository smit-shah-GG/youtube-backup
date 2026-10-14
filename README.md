# youtube-backup

Exports your own YouTube subscriptions and playlists to CSV, on a weekly timer, to your
own disk. Read-only. Single user. No server.

A YouTube account can be lost without warning, and deleting or losing a channel takes its
subscriptions, playlists and history with it irreversibly. This means the rebuild starts
from a list rather than from memory.

## Usage

```
youtube-backup --auth      one-time browser consent, writes the refresh token
youtube-backup             pull and write CSVs (what the timer runs)
youtube-backup --status    when it last succeeded, and what is on disk
```

Output goes to `~/.local/share/youtube-backup/`. A new dated CSV is written only when the
content actually changes, so the dated files are the change history; `*-latest.csv` is
always current.

## Setup

1. Google Cloud Console → new project → enable **YouTube Data API v3**.
2. Google Auth Platform → **Branding**: app name, support email, homepage URL, privacy
   policy URL. (The last two are the pages in this repo, served by GitHub Pages.)
3. Google Auth Platform → **Audience** → **Publish app**. This matters: while publishing
   status is "Testing", refresh tokens expire after 7 days and the timer dies quietly.
   Unverified production is fine — the 100-user cap is irrelevant for one user.
4. **Clients** → Create OAuth client → **Desktop app** → download JSON →
   save as `~/.config/youtube-backup/client_secret.json`.
5. `youtube-backup --auth`
6. `systemctl --user enable --now youtube-backup.timer`

## Failure behaviour

Deliberately asymmetric, because a backup that fails silently is worse than no backup:

- Real problems (expired token, revoked auth, API errors) raise a critical desktop
  notification and exit non-zero.
- Transient problems (no network, YouTube 5xx) exit 75 and stay quiet, so systemd just
  retries. A tool that shouts about a closed laptop gets muted, and then the failure that
  mattered goes unseen too.

## Known gap

Watch Later and Liked Videos are not retrievable through the Data API. Run a one-off
Google Takeout export if you want those.

## Requires

`uv` (dependencies are declared inline, PEP 723). Nothing to install.
