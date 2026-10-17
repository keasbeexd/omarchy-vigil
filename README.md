## This repo has been archived as the scope has gotten to large for my expertise.

# Vigil - Security Suite

A security suite for [Omarchy](https://omarchy.org): one lighthouse in the bar,
one panel, five pillars. The widget and panel take every color from your
active Omarchy theme.

<img src="docs/lighthouse.svg" width="96" alt="Vigil's lighthouse, in 16 EGA colors">

![Vigil in a Turbo Pascal-style screen: the panel's Overview beside its program listing](preview.png)

The panel above is running under a DOS-blue Omarchy theme. It is theme
dependent: every color it draws comes from whichever Omarchy theme is active,
so under yours it takes on your theme's colors.

| Pillar | The question it answers |
|---|---|
| **Harden** | Can someone get in? Firewall, SSH, kernel protections, disk encryption, screen lock, sudo, security keys, updates, open ports. |
| **Detect** | Is something bad already here? Malware hiding places, files that changed, virus scanning through [ClamUI](https://github.com/linx-systems/clamui), rootkits. |
| **Supply** | Can I trust what I installed? Known-compromised packages through Perplexity's [Bumblebee](https://github.com/perplexityai/bumblebee), known vulnerabilities, AUR packages, your other Omarchy plugins, AI model files that run code. |
| **Privacy** | What does this machine tell the networks it joins, and the apps on it tell their makers? DNS fallback, hardware address, connectivity pings, announcements, telemetry opt-outs, AI agent telemetry, blocked telemetry hosts, Chromium's privacy policy, the weather location. |
| **Care** | Is the system tidy? Thumbnails, trash, caches, what removed apps left behind, old packages and logs. |

Vigil keeps no threat lists of its own. Everything it checks against comes
from the people who maintain it (the Arch security team, OSV.dev and the
OpenSSF, ClamAV, Perplexity's Bumblebee, the AUR itself): each scan gets
the newest list right before it runs, and ClamAV's own update service keeps
the virus signatures current. It is tuned for no false alarms:
only confirmed problems lower the score, and anything that is merely worth
knowing is shown as information.

Every finding says, in plain language, what Vigil found and why it matters.
Most come with a fix. Nothing changes until you press **Run** on a
confirmation in the panel that says exactly what will happen; anything that
needs root then asks for your password, and the panel shows the result.

Vigil is written for people who are new to Linux security. If you are not,
the [security model](docs/SECURITY-MODEL.md) explains how it is built.

> **Status: 0.31.0.** The [roadmap](docs/ROADMAP.md) lists what comes next, and
> [docs/LANDSCAPE.md](docs/LANDSCAPE.md) compares Vigil with the other
> security plugins for Omarchy.

## Install

```sh
omarchy plugin add https://github.com/keasbeexd/omarchy-vigil --enable
```

After updating Vigil, restart the shell so the new panel loads (the panel
offers a **Restart shell** button when it notices the update); until then
the old panel keeps running against the new helper.

The lighthouse appears on the right of the bar. Nothing else is needed: Vigil
uses Python and commands Omarchy already ships. ClamUI, ClamAV, the
scanners, fwupd, gitleaks, rkhunter and Bumblebee are optional. When any are missing, the Overview shows
a red **MISSING … INSTALL** button before the run buttons; it lists
what it will install, asks once more, and then asks for your password.

## Using it

| | |
|---|---|
| Left-click the lighthouse | Open or close the panel |
| Middle-click the lighthouse | Read your settings and files again now (no scans, no network) |
| `←` `→` or `1`–`6` | Switch tabs |
| `↑` `↓`, `Enter` | Move through findings, open one; `Enter` also runs the confirmation that is showing |
| `F` | Fix the selected finding (asks first) |
| `D` | Mark the selected finding as reviewed, or show it again |
| `R` or `S` | Run all (asks first) |
| `C` | On the Care tab: Clean all |
| `Esc` | Cancel a confirmation, or close |

From a keybinding or script:

```sh
omarchy-shell keasbeexd.vigil toggle
omarchy-shell keasbeexd.vigil refresh
```

The lighthouse is dim until the first check finishes, about eight seconds
after login, and shows its beams while a check runs. It takes your
theme's urgent color when something needs fixing and its accent color when
only improvements are left; every color in Vigil comes from your Omarchy
theme.

Each pillar's tab says, right under it, what matters most there: how many
findings to fix (in your theme's urgent color), else to improve (accent),
else notes, else "all good". Hover a tab for every count; hover **Overview**
for the score. **Care** sits apart, past a divider: it is upkeep, not
security (see [Care](#care)).

The **Overview** tab shows your score out of 100, the run buttons under
**Run now** with when each check next runs by itself, a line on Care, and the
Last run list. The line beside the title says when Vigil last checked; on the
other tabs it shows the score.

In the pillar tabs, rows are grouped under headings that say which check
brings them in: **Always current** (see below), then **Daily check** and
**Weekly check**. The virus scan row on the Detect tab holds both virus
scans (new downloads daily, your home folder weekly), so it sits under
**Daily and weekly checks** and names each scan with its run. A tab
with only one group has no headings. Rows with nothing to do (passing, not
applicable, or marked as reviewed) are dimmed. The key hints at the bottom
show the keys of the tab you are on.

### When Vigil runs what

Most rows are **always current**: Vigil reads them from your settings and
files on this computer, with no network and no scanning, in about a second,
whenever you open the panel, after every fix, when you middle-click the
lighthouse, and every 30 minutes (the `refreshMinutes` setting). That is
not a run you start, so it has no button.

On top of that, Vigil has two runs, the daily check and the weekly check.
In both, **each scan gets its list right before it runs**, so it compares
against the newest one the people who maintain it have published. The
virus signatures are the exception: ClamAV's own update service keeps them
current (see below).

| Run | What it does, in order | When |
|---|---|---|
| **Daily check** | Looks up waiting system, Omarchy and Flatpak updates. Then: the **Arch security scan** (arch-audit downloads the Arch security team's list and compares your packages with it), the **OSV malware scan** (osv-scanner checks your AUR build folders and global npm, Bun, mise and pnpm installs against OSV.dev as it runs), the **AUR status lookup**, the **Bumblebee list** download followed by the **Bumblebee scan**, and the **Virus signatures** check followed by the **virus scan of what is new in Downloads** | once a day, in the background, at low priority |
| **Weekly check** | The firmware lookup (fwupd fetches LVFS's list first), the **secrets scan of your git repositories** (gitleaks), the **AI model file scan**, then the **Virus signatures** check followed by the **virus scan of your whole home folder** | once a week, in the background, at low priority |
| Omarchy update (`omarchy-update`) | | only when you press **Omarchy Update**, or **Update now** on the System updates row, in a terminal |
| Password scan (accounts without a password) and rootkit scan (rkhunter) | | only when you press **Run all**, since they need your password |
| Cleaning (Care) | | only when you ask; Vigil only measures sizes by itself |

**Virus signatures.** ClamAV keeps its signatures in a system folder, so
updating them needs root. ClamAV's own update service (`clamav-freshclam`,
which Vigil turns on when it installs ClamAV) refreshes them every couple of
hours, so they are normally never more than a few hours old. Right before
every virus scan, Vigil checks that they are there and how old they are, and
leaves updating them to ClamAV: the daily and weekly checks never ask for a
password. If the service falls behind and they are **over two days old**
(the service is off, or the computer was offline), Vigil sends one
notification a day saying so (with `notifyOnNew` on), the Virus signatures row on the Detect tab turns to a warning,
and **Run all** updates them first, behind your password, with that exact
command on its card. Until then, scans use what is there and say how old
it was. The Virus signatures row also offers to turn the update service
back on if it is off.

**The virus scans say which signatures they used**: their Last run lines
read "with signatures from Oct 14". The Bumblebee scan says "with the list
from Oct 14" (on its row and its Last run line) only when that day's
download failed and it used the last good list. arch-audit and osv-scanner
fetch their lists as they run, so their lines need no such note. A list or
signatures that were over two days old when the scan ran turn that line
your theme's red, and the Infected files row then says so too. A scan with
no list at all does not run, and is tried again the next day.

Under **Run now**, the Overview has a button for each run, and **Run all**
for both. Under each button is when that run next happens by itself. Each
asks first, on a card that lists exactly what it runs, in order:

- **Daily check** runs the daily check now. Pressed, it also installs
  updates for your own Flatpak apps. No password.
- **Weekly check** runs the weekly check now: the firmware lookup, the
  secrets scan of your git repositories, the AI model file scan, then a
  virus scan of your whole home folder (skipping caches, game libraries,
  Flatpak and container data, language toolchains, `node_modules` and
  `.git` folders, and other disks; at low priority). It can take a while;
  a notification says when it is done.
- **Run all** (`R` or `S`) runs the daily check, then the weekly check, plus
  the password scan and the rootkit scan, and updates the virus signatures
  first when they are over two days old. Those are the only password prompts
  in any run, and they come first, while you are at the screen. The virus
  scan of your home folder includes Downloads, so Run all does not scan
  Downloads twice. Run all does not update the system; that is the
  **Omarchy Update** button.
- **Omarchy Update** (under the run buttons) opens a terminal and runs
  Omarchy's own `omarchy-update`, so it does everything that command
  normally does: system packages, your AUR packages, your mise tools and
  the rest. It asks first, on the usual card with the exact script, then
  sudo and Omarchy's own questions (including any restart) happen in the
  terminal. Vigil does not follow or alter it. The line beside the button
  says when the last full update was, from pacman's log, and the Last run
  list shows the same under **Omarchy Update button**. It is separate from
  **Update now** on the System updates row because that row looks only at
  pacman's waiting updates (`checkupdates`), not at AUR packages or mise
  tools; both start the same updater.

Some rows also have a button of their own: **Scan now** on the Arch
security and OSV rows runs just that scan, and **Look up now** on the AUR
status row just that lookup (each asks first). **Scan now** on the Infected
files row starts the virus scan of your home folder alone, with progress.
None needs a password.

Bumblebee itself is never installed or updated by a run. Vigil installs
only the version it was reviewed with, from the **Install** button; when a
Vigil update raises that version, the Bumblebee row offers **Install
Bumblebee** with the new one.

The panel shows a progress bar with the current step (and, during a virus
scan, how many files are done), and each line of the Last run list updates as
its step finishes. Results stay on a card in the panel until you close it.

While a check runs, **Stop** under the progress bar ends it:
the step it is on stops, what already finished is kept, and the rest is
tried again the next day (or when you press the button again). Stop also
ends a run left going after the shell restarted.

Only one run (a check, Run all, or the Infected files row's scan) goes at a
time. Pressing a button while one runs
(on another monitor, or one still going after the shell restarted) starts
nothing new: the panel shows "Already running" with that run's progress
instead. Fixes, Clean all and installs show their own progress bar while
they run, so you can close the panel and come back to it.

Under each run button is when that check next runs by itself, with the time
in your computer's timezone when it is today or tomorrow: each runs a day or
a week after the last one, to the minute, or at Vigil's next background
check if it is already due. Each button resets its own countdown. Turning
off `autoUpdate` leaves both to the buttons ("by hand only").

None of these runs cleans anything. Care is separate from security: its rows
never count toward the score or notify you, and Vigil only measures them;
cleaning happens only when you press a button on the Care tab.

Below that, **Last run** says when Vigil last read your settings and files,
then lists each scan and lookup under the run that does it (Daily check,
Weekly check, Run all only), in the order it runs them, with when it last
ran. There are no lines for the lists themselves: each scan fetched its own
right before it ran. The rows in each tab are ordered the same way: the
always-current rows first, then the rows of the daily check and the weekly
check. Anything that failed, is out of date, or used a list that was over
two days old when it ran is shown in your theme's red (Omarchy's `urgent` color, which
every theme sets from its own `red`).

Work done outside Vigil counts too: signatures updated by ClamAV's own
update service or in ClamUI count as current, and the virus scan line
includes scans you ran in ClamUI (marked "by ClamUI"). The Arch security,
OSV and AUR lines can only show Vigil's own runs, since those tools keep no
record when run by hand.

### Every finding

Open any finding and it says, in this order: **Why it matters** (a sentence
or two), **How to fix**, and **Details**. Below that are its buttons:

- **Fix** (named for what it does) when Vigil can do it, or **Learn more** /
  **Show …** when it can't: a guide in your browser (the Arch Wiki, a
  plugin's page) or a read-only command whose output the panel shows.
- **Undo Vigil's change** once Vigil has changed something for it.
- **Mark as reviewed** on anything that is not passing, when you don't want
  that fix: a port you opened on purpose, an unencrypted desktop that never
  leaves home. It stays in its tab, dimmed, no longer lowers the score or
  notifies, and stays reviewed through every later check and scan. **Show
  again** undoes it. Four of the lists you tick (below) are the exception:
  leftover app data, orphaned packages, foreign packages and other plugins
  are reviewed item by item with **Keep**, so those rows have no Mark as
  reviewed (Infected files keeps it).

For findings that report evidence rather than a setting (open ports, startup
files, file changes, exposed secrets, infected files, known-malicious or
known-compromised packages, vulnerable packages, system-wide preload,
abandoned AUR packages, user accounts, shell history files, old network
services), a review covers what was found at the time. If
something new turns up, the finding comes back.

### Lists you tick

Six rows list what they found one item per line (most on two lines: what
it is, then more about it), with a small box to tick: leftover app data,
orphaned packages and big cache folders (Care), infected files (Detect), and
foreign packages and other Omarchy plugins (Supply). Tick the items you want to act on (the box,
or the item's text), then use the buttons under the list, which say how many
they act on.

| List | Remove | Keep |
|---|---|---|
| Leftover app data | **Trash selected**, or **Trash all** with nothing ticked: to the trash, never deleted | **Keep selected** / **Keep all** |
| Orphaned packages | **Remove selected**, or **Remove all** with nothing ticked: `pacman -Rns`, behind your password | **Keep selected** / **Keep all** |
| AUR and other foreign packages | **Remove selected** only: `pacman -Rns`, behind your password | **Keep selected** / **Keep all** |
| Other Omarchy plugins | **Remove selected** only: Omarchy's own `omarchy-plugin-remove`, as you | **Keep selected** / **Keep all** |
| Big cache folders (1 GB or more) | **Clear selected** only: deletes everything inside each ticked folder, for good | **Ignore selected** / **Ignore all** |
| Infected files | **False positive selected** only: Vigil stops counting that exact file | (none) |

Removing, trashing and marking a false positive always ask first, on a card
at the top of the panel that lists each item; a card that removes packages
or plugins also shows the exact command it runs (see
[How actions run](#how-actions-run)). There is no "all" for what you
cannot easily get back (foreign packages, plugins) or must be sure of
(false positives).

**Keep** (called **Ignore** on the big cache folders, where it stops pointing the
folder out) leaves an item as it is and stops counting it: settings you want
back if you install an app again, an orphan you still use, an AUR package or
plugin you have looked at and trust. Keeping only changes Vigil's own notes,
so it does not ask. A kept item, and a file marked as a false positive,
shows a lock in place of its box and moves to the bottom of the list;
**click the lock** to count it (or list it) again. A plugin you kept is counted again by
itself if its code gains a new capability, and a false positive if the file
changes.

### Undo

Every file Vigil writes in `/etc` can be undone from the finding it fixed:
open it and choose **Undo Vigil's change**. Cleanups and added passphrases
have nothing to put back. Turning the firewall on, switching old network
services off and turning on ClamAV's update service change system services
and have no Undo button; [Removing](#removing) has the commands that
reverse them. The undo asks for your password
like the fix did, and it deletes the file only if its first line is still Vigil's
marker; a file someone has edited since is left alone. For Avahi, undo puts
back the original file Vigil saved before its first change. The Chromium
policy is removed only while its bytes are exactly what Vigil wrote, and the
telemetry block in `/etc/hosts` is taken out by its markers, leaving every
other line. The variables in `~/.config/uwsm/env.d` are removed from their
own rows (no password); the rounded weather location cannot be undone.

### Settings

Set these on Vigil's entry in `~/.config/omarchy/shell.json`, or in the bar
settings:

| Key | Default | Meaning |
|---|---|---|
| `refreshMinutes` | `30` | How often to read your settings and files again in the background (5 to 1440). |
| `notifyOnNew` | `true` | One desktop notification when a check that was fine starts failing or warning (checks with no weight never notify). It only says how many. Also one a day, in fixed words, while ClamAV's virus signatures are over two days old. (A weekly check or Run all you start also says when it finishes, whatever this is set to.) |
| `autoUpdate` | `true` | Run the daily check and the weekly check by themselves (see [When Vigil runs what](#when-vigil-runs-what) and [Where the data comes from](#where-the-data-comes-from)). Off means they run only when you press a Run, Scan now or Look up now button. |

## What Vigil checks

Each check has a weight from 0 to 3. A failure costs 10 points per weight
point and a warning 2, so a confirmed problem moves the score a lot and an
improvement only a little; checks with weight 0 are information and never
lower it. A finding you mark as reviewed stops counting.

### Harden

| Check | Finding when | Fix |
|---|---|---|
| Firewall | ufw is not installed, switched off, or lets incoming connections in | Turns ufw on (asks for your password) |
| Kernel self-protection | Any of nine kernel switches is off (`kptr_restrict`, `dmesg_restrict`, `ptrace_scope`, unprivileged BPF, the `fs.protected_*` set, `suid_dumpable`) | Writes `/etc/sysctl.d/90-omarchy-vigil.conf` (asks for your password) |
| Network stack | ICMP redirects, source routing, reverse-path filtering, SYN cookies or broadcast pings are set loosely | Same file as above |
| LLMNR | systemd-resolved answers LLMNR, which attackers on shared Wi-Fi abuse | `/etc/systemd/resolved.conf.d/90-omarchy-vigil-llmnr.conf` (asks for your password) |
| SSH server | sshd runs and allows passwords or root login | `/etc/ssh/sshd_config.d/00-omarchy-vigil.conf`, only if you have an authorized key, validated with `sshd -t` (asks for your password) |
| SSH keys | `~/.ssh` or a private key is readable by others, or a key has no passphrase | Makes them private (no root); **Add a passphrase** runs `ssh-keygen -p` for each key without one (in a terminal, since you type the passphrase; no root) |
| GnuPG folder | `~/.gnupg` is readable by others | Makes it private (no root) |
| Automatic screen lock | Stay Awake is on, the lock is off, or it takes over 15 minutes | Turns Stay Awake off; sets `idle.lock` to 300 seconds, Omarchy's default, with Omarchy's own `omarchy-shell-config` helper (no root) |
| Disk encryption | The system disk is not on LUKS | Cannot be added after install; Arch Wiki guide |
| Restart after updates | A newer kernel is installed than the one running | **Restart now** (after a confirmation) |
| Secure Boot | Shown for information only; Vigil does not suggest setting it up, since a mistake can stop the computer starting | |
| Wrong-password lockout | `pam_faillock` is missing or never locks | Arch Wiki guide |
| FireWire | The machine has a FireWire port and its driver can load (information; shown only on machines with the port) | `/etc/modprobe.d/90-omarchy-vigil-firewire.conf` (asks for your password) |
| Passwordless sudo | Omarchy's passwordless sudo is on | `omarchy-sudo-passwordless --disable` (terminal) |
| Docker group | You are in the docker group, which equals root | `omarchy-remove-security-sudoless-docker` (terminal) |
| Security key or fingerprint | sudo takes only a password (information). On a computer with a fingerprint reader (found by Omarchy's own `omarchy-hw-fingerprint`, which only reads `/sys`), it offers fingerprint instead | `omarchy-setup-security-fingerprint` or `omarchy-setup-security-fido2` (terminal); once set up, **Stop using…** runs Omarchy's `omarchy-remove-security-fingerprint` or `-fido2` |
| System updates | A newer Omarchy is out (warning; found by `checkupdates`, the same check Omarchy's own update indicator uses, in each daily check), or no full update in 14 days (warning) or 30 days (failure). Also says how many updates are waiting | `omarchy-update` (terminal) |
| Flatpak updates | Omarchy's updater does not update Flatpak apps. Vigil checks them each day; waiting updates are a warning. **Daily check** and **Run all** install updates for your own apps without a password | `flatpak update` (no terminal) |
| Open Wi-Fi that joins by itself | Saved networks without a password (or with old WEP) that your laptop joins by itself, so anyone can imitate them. WPA, WPA3 and Enhanced Open never count (warning) | **Stop auto-joining**: `nmcli` sets them to join only when you choose; Undo puts back exactly those |
| Firmware updates | The weekly check asks fwupd, which gets firmware from LVFS where hardware makers publish it. A waiting update that fixes a known security hole (a CVE, or marked high or critical) is a warning; other firmware updates are information | `omarchy-update-firmware` (terminal) |
| Open network ports | Programs listen on the network with no firewall in front | **Show listeners** runs `ss -tulpn` as root and shows the result in the panel (read-only) |
| Exposed secrets | API keys and tokens in the formats their providers publish (AWS, GitHub, GitLab, Slack, Stripe, Anthropic, OpenAI, Google, private keys) in your shell histories, or credential files such as `~/.aws/credentials`, `~/.netrc` and `~/.git-credentials` readable by other accounts. Shows the file and line, never the secret. | Makes credential files private (no root); **Remove those lines** deletes the lines with keys from your shell history (no root); revoke leaked keys with their provider |
| Secrets in your git repositories | gitleaks, in the weekly check, finds keys and tokens in the files of the git repositories below your home folder (the working tree, not the history; at most 100 repositories, four folders deep). Shown as repository, file, line and rule, never the secret (warning) | Revoke the key with its provider, then take it out of the file. A safe line, such as a test value, can carry a `gitleaks:allow` comment. A repository's own `.gitleaks.toml` is ignored on purpose, so a repository cannot switch its own findings off |

### Detect

| Check | What it does |
|---|---|
| Startup files | Reads shell startup files, autostart entries, user services and timers, uwsm and environment.d files, Hyprland config and your crontab, and flags lines that download and run code, decode hidden code, or open reverse shells. These are leads to read, not proof. |
| System-wide preload | Fails if `/etc/ld.so.preload` loads anything; rootkits use it to hide. |
| Rootkit scan (rkhunter) | Run all runs rkhunter as root (Omarchy asks for your password): known rootkits, hidden files, and system programs that changed. Warnings for a program that is byte for byte what an installed package shipped (checked against pacman's own records) are explained and do not count; the others are a warning to read, not proof. The first run records the current system programs as rkhunter's baseline. Its signatures are old, so it mainly finds old, well-known rootkits; the changed-program check is the useful part. Never in the background, and it makes no network call |
| Changes to important files | Records a fingerprint of accounts, PAM, SSH settings, `~/.ssh/authorized_keys`, system and user services, autostart entries and your startup files, and lists anything changed, added or removed since. Changes that match exactly what a package update shipped (from pacman's own records) are not reported, nor are system users that packages add. Changes to who can log in or what runs at start are warnings; changes to your own shell files are information. **Accept these changes** records the current state as the new normal. |
| Infected files (ClamAV) | ClamAV scans your home folder in each weekly check, and every day what is new in Downloads, as you, each time right after Vigil checks the virus signatures are there and how old they are. Infected files are listed one per line (the file, then what ClamAV found, in the theme's red for known malware and yellow for a guess by ClamAV's heuristic rules), with **Open ClamUI to quarantine them**. If ClamAV is wrong about a file you know is safe (a game's DLL, say), tick it and press **False positive selected**: Vigil records that exact file's SHA-256 and stops counting it, and reports it again if the file ever changes. Clicking its lock counts it again. ClamAV's heuristic detections (`Heuristics.…`) count too, shown in yellow. Its "potentially unwanted" rules stay off, as ClamAV ships them; if you turned them on in ClamUI, their `PUA.…` detections are yellow too. |
| Duplicate scan schedules | Finds other apps that repeat what Vigil already runs: a scheduled scan in ClamUI whose folders are all inside your home folder, and the older Bumblebee plugin (once Vigil runs Bumblebee itself). **Let Vigil run them** turns those off the way you would in each app: ClamUI's timer and its own Scheduled scans setting, and `omarchy plugin disable` for the plugin. Undo turns back on exactly what it turned off. A ClamUI schedule that also covers folders outside your home folder (Arch or Flatpak ClamUI: they share one timer, so ClamUI is then left alone entirely), and the Security Scan plugin (it runs AUR-Malware, which Vigil does not), are listed but left on. Nothing is offered when Vigil's background checks are off. |
| ClamUI | Vigil hands virus scanning to [ClamUI](https://github.com/linx-systems/clamui): file and folder scans, scheduled scans, USB drive scans and quarantine. When it is installed, **Open ClamUI** starts it; otherwise Vigil installs it from the Arch repositories (`clamui`), so the Omarchy update keeps it current, and it uses the system's ClamAV. A ClamUI you installed from Flathub keeps working. |
| Virus signatures (ClamAV) | Whether ClamAV is installed and its signatures keep themselves up to date (they count as current for two days; older, the row turns to a warning). Vigil turns on ClamAV's own update service (`clamav-freshclam`) when it installs ClamAV or ClamUI, and offers to if it is off. |

### Supply

| Check | What it does |
|---|---|
| Known-compromised packages (Bumblebee) | Runs Perplexity's [Bumblebee](https://github.com/perplexityai/bumblebee) in each daily check. It checks your packages, lockfiles, editor and browser extensions and MCP configs against the compromised releases Perplexity tracks, and reads files only. **Install** downloads Bumblebee 0.1.2, the version this Vigil was reviewed with, installs it only if it matches the SHA-256 written in Vigil's code, and puts it in `/usr/local/bin/bumblebee` (owned by root). Vigil never fetches a newer Bumblebee by itself, and no run installs or updates it: a newer version arrives with a Vigil update, and the row then offers **Install Bumblebee** with it. Each daily check downloads Perplexity's newest list right before the scan. |
| Known-malicious packages (OSV) | Runs [osv-scanner](https://github.com/google/osv-scanner) over your AUR build folders (`~/.cache/yay`, `~/.cache/paru`) and global npm, Bun, mise and pnpm installs, against [OSV.dev](https://osv.dev), which includes the [OpenSSF Malicious Packages](https://github.com/ossf/malicious-packages) reports. This is how the 2026 Atomic Arch AUR attack (which pulled malware in with `bun install`) and the npm worms that run on Bun are caught. Only confirmed malware (`MAL-` entries) counts; ordinary vulnerabilities in development dependencies are left out. |
| Known vulnerabilities (arch-audit) | Runs arch-audit in each daily check (the row offers **Install** if it is missing). It uses the Arch security team's tracker and lists only the problems an update fixes. **Scan now** runs it straight away. |
| AUR package status | Asks the AUR about your AUR packages: which have left the AUR, which are now in the official repositories, and which have no maintainer (most 2026 AUR attacks took over such orphans). Information only, since packages leave the AUR for ordinary reasons too. |
| AUR and other foreign packages | Lists packages that did not come from the official repositories, each with its version, size, description and what the last AUR lookup said about it. Tick what you no longer use and press **Remove selected**; **Keep** what you trust (see [Lists you tick](#lists-you-tick)). |
| Other Omarchy plugins | Lists your third-party plugins and what each one's code can do: network access, root through sudo or pkexec, shell commands, downloading and running code, compiled binaries. Vigil reads the files as text and never loads or runs them. Each capability is colored by how much it deserves a look, in the theme's own colors: red for root, downloading and running code, and compiled binaries; yellow for network access, shell commands, and a linked development install Vigil did not scan; green when none were found. Tick a plugin you no longer use and press **Remove selected**; **Keep** one you have looked at and trust. |
| AI model files that run code | Model files in the pickle format (`.pt`, `.pth`, `.ckpt`, `.bin`, `.pkl`) run whatever code they carry when a program loads them. The weekly check reads their instructions with Python's own pickle disassembler (which never runs anything) and lists files that import something able to run programs: `os`, `subprocess`, `eval`, `socket`, and the like (warning). It looks in your home folder and in `~/.cache/huggingface` and `~/.cache/torch`, skipping `node_modules`, Python environments and links. Safetensors and GGUF files are plain data and are not read |

### Privacy

| Check | Finding when | Fix |
|---|---|---|
| DNS fallback servers | systemd-resolved falls back to Google, Cloudflare or Quad9 | `/etc/systemd/resolved.conf.d/90-omarchy-vigil-fallback.conf` (asks for your password) |
| Connectivity check | NetworkManager pings a server every few minutes (information) | `/etc/NetworkManager/conf.d/90-omarchy-vigil-connectivity.conf` (asks for your password) |
| Hardware address on Wi-Fi | Networks see your real MAC address | `/etc/NetworkManager/conf.d/90-omarchy-vigil-mac.conf` (asks for your password) |
| Network announcements | Avahi announces this machine's name | Sets `disable-publishing=yes` in `/etc/avahi/avahi-daemon.conf` (asks for your password). Refuses unless the setting appears exactly once under `[publish]`, keeps the first original as `avahi-daemon.conf.vigil-backup`, and restores the previous file if Avahi does not restart. |
| Telemetry opt-out variables | `DO_NOT_TRACK` and similar are not set (information), or Vigil's list has grown since you set it | Writes `~/.config/uwsm/env.d/90-omarchy-vigil-opt-out` (no root): `DO_NOT_TRACK`, and the switches of .NET, Next.js, Deno, Terraform, vcpkg, onnxruntime, gh (`GH_TELEMETRY`), mise and others. Applies from your next login |
| AI agent telemetry | Claude Code, Codex, Gemini CLI, opencode or pi is on this computer (Omarchy installs them) and `DISABLE_TELEMETRY` is not set for Claude Code (warning), or for the others only (information) | Writes `~/.config/uwsm/env.d/90-omarchy-vigil-agents` (no root): `DISABLE_TELEMETRY` and `DISABLE_ERROR_REPORTING` for Claude Code, `PI_TELEMETRY=0` for pi, `GEMINI_TELEMETRY_ENABLED=false` for Gemini CLI. Settings you made in `~/.claude/settings.json` count. Codex has no variable; its crash-report address is in the hosts block below |
| Telemetry hosts | Some of fourteen telemetry, analytics and crash-report addresses (Claude Code's Datadog and Sentry, Codex's Sentry, 1Password's telemetry and Sentry, Brave's usage ping and P3A, Mozilla's telemetry, Google Analytics) can still be reached (warning) | Adds one marked block to `/etc/hosts` that sends each to `0.0.0.0` and `::`, keeping every other line (asks for your password). Pages that use Google Analytics stop reporting your visits to it. Undo takes the block out; addresses you or another tool blocked by hand count as blocked |
| Chromium privacy policy | Chromium is installed and these are not set in a policy file: search suggestions, page preloading, Google error pages and spelling service, usage reports, the Privacy Sandbox ad features, password leak lookups, background running, and the Cast device search (warning) | Writes `/etc/chromium/policies/managed/90-omarchy-vigil.json` (asks for your password). Casting from Chromium stops working; Safe Browsing, updates and cookies are left as they are. Undo removes the file while it is unchanged |
| Weather location | Omarchy's weather panel stores the place you chose to about a metre and sends it to wttr.in and open-meteo.com with every refresh (warning; shown only when a place is stored) | **Round it**: moves the stored point to the centre of a square 1.5 miles (2.4 km) across, which is all a forecast needs (no root). It cannot be undone: the precise figures are not kept |

Vigil does all of this itself, once, when you press a button: no watchdog, no
listener, no service. If a later update puts a setting back, the row turns to
a warning the next time Vigil reads your files, and the same button sets it
again. Earlier versions only detected the separate
[Black Ops](https://github.com/brightwalker25/omarchy-black-ops) plugin and
left telemetry blocking to it. If you still run it, Vigil recognises what it
set (an address blocked in `/etc/hosts` or a variable in `~/.config/uwsm/env.d`
counts, whichever tool wrote it), and the two do not fight over files: use one
of them, and switch Black Ops off before you rely on Vigil's rows.

### Care

Care is upkeep, not security, so it is kept apart from everything else:
its rows never count toward the score or notify you, and neither the daily
check nor the weekly check cleans anything. Vigil only measures (whenever it reads your settings and files);
it cleans when you press a button here. The tab shows its rows in two
groups:

**In Clean all**: cleanups that are safe to do in one go.

| Row | What it measures | Cleanup |
|---|---|---|
| Thumbnail cache | Previews the file manager made of your pictures and videos | Cleared without root |
| Trash | Files you deleted in the file manager | Emptied without root, directly in its standard folders |
| Application caches | Files apps keep in `~/.cache` to start faster | **Clear unused** (no root); and for folders of 1 GB or more, tick them and **Clear selected**, or **Ignore** them; see below |
| Package cache | Packages pacman downloaded | `paccache -rk2` and `paccache -ruk0` (asks for your password), see below |
| System logs | The system journal; under 500 MB it counts as tidy | Over 500 MB: `journalctl --vacuum-time=2weeks` (asks for your password) |

**Not in Clean all**, at the bottom of the tab under its own heading:
removing packages, or settings you may want back, deserves a look, so you
tick what to remove or keep (see [Lists you tick](#lists-you-tick)).

| Row | What it measures | Cleanup |
|---|---|---|
| Orphaned packages | Packages installed as dependencies of something you have since removed, each with its version, size and description | Tick items, then **Remove** them (`pacman -Rns`, asks for your password) or **Keep** them |
| Leftover app data | Settings and data that apps you removed left in your home folder | Tick items, then **Trash** or **Keep** them (no root), see below |

**Application caches.** A file in `~/.cache` counts as unused when no app
has read or changed it for a week, whether or not that app is open now.
Apps you use every day touch their caches each time they start, so on a
busy computer most of `~/.cache` stays "in use", and closing your apps does
not change that. **Clear unused** appears once more than 1 MB has gone a
week untouched, such as the cache of an app you stopped using. Caches in use
are never deleted by Clear unused or Clean all, since apps would only rebuild them.

**Big cache folders.** To show which caches are getting big, a folder directly
in `~/.cache` that holds **1 GB or more** is also listed on this row as a
checklist, each as its place and total size (`~/.cache/chromium · 1.7 GB`),
biggest first. This list changes nothing about Clear unused or Clean all: they
clear files nothing has used for a week in every cache folder, big or not, as
before, and need no ticking. What the list adds is your own choice: tick the
folders you want and press **Clear selected** (there is no "all"). That deletes
**everything inside** each ticked folder, in use or not, for good (the folder
itself stays), so close the app first; the card lists each folder and says so.
Each folder is walked for up to 100 seconds, two million entries and 128
folders deep, and anything of a megabyte or more left after that is named in
the answer.
**Ignore** a folder you want to keep: it stays listed, locked at the bottom,
and stops being pointed out in the row's summary (click the lock to list it
again). Links are removed, never followed, and the helper checks again that
each ticked folder is still listed, 1 GB or more, and not ignored. To see what
takes the space: `du -sh ~/.cache/* | sort -h`.

**Leftover app data.** When you remove an app, its settings and data stay
in your home folder. A file's age says little here (you set an app up once
and it reads those settings for years), so Vigil asks a different question:
does anything on this computer still use it? A folder counts as used when
its name matches an installed package (or what a package provides), a
program, a desktop entry or its window class, an installed Flatpak,
Omarchy's own configs, a running program, or a line in your startup files
(`~/.bashrc` naming `~/.cargo/bin`, say). Folders whose names do not look
like their app, such as `~/.mozilla` for Firefox or `~/.config/Code` for
VS Code, are matched too. Shared folders no single app owns (`autostart`,
`systemd`, `gtk-3.0`, `fontconfig`, `pipewire`, `hypr`, `omarchy`, ...)
and folders that hold your keys or shell (`~/.ssh`, `~/.gnupg`,
`~/.oh-my-zsh`, `~/.nvm`, ...) are never listed, and neither are links,
which usually come from a dotfile manager.

| Where | Listed when |
|---|---|
| `~/.config`, `~/.local/share`, `~/.local/state`, and the folders in `~` that start with a dot | nothing uses it, and nothing in it has changed for 30 days |
| `~/.var/app` | its Flatpak app is no longer installed (`flatpak uninstall` keeps your data unless you ask) |
| `~/.local/bin` | it is a link to a program that is gone |
| `~/.local/share/applications` | the launcher starts a program that is gone, or a Flatpak that is no longer installed |

The list is information: it never lowers the score or notifies. Each item
reads as where it is and how big, then why it is listed and since when. Open
the row and tick the items you want to act on (see
[Lists you tick](#lists-you-tick)).

- **Trash** asks first, on a card at the top of the panel that lists each
  item, then moves them to the trash, the same trash your file manager
  uses. Nothing is deleted: open the trash in your file manager and choose
  **Restore** to put one back. Emptying the trash deletes it for good, and
  so do **Empty** on the Trash row and **Clean all**, which empty the trash
  too.
- **Keep** leaves them where they are: settings you want back if you
  install the app again, or something Vigil cannot know you use. They move
  to the bottom of the list with a lock and are no longer counted; click
  the lock to count one again.

Just before it moves anything, Vigil looks again: an item that something
now uses, or that you kept, stays where it is. Leftovers are never part of
Clean all.

**Package cache.** Vigil keeps the two newest versions of each installed
package, so a bad update can be rolled back, and frees older versions and
every version of packages you have removed. The row shows how much a dry run
of `paccache` says can be freed; "all of it kept for rollback" means there
is nothing to free.

**Clean all** (or `C` on the Care tab) does every cleanup on offer in one go,
after one confirmation that lists each item and its size: thumbnails, the
trash and unused caches right away, then the package cache and the journal,
for which Omarchy asks for your password once. Orphaned packages and
leftover app data are never part of it: removing packages can take away
something you use, and leftovers can be settings you want back, so both
stay separate steps you review item by item.

### Accounts, history and old services

Checks a newcomer benefits from that need no extra tools:

| Check | Finding when | Fix |
|---|---|---|
| User accounts | Another account has root's powers, accounts share one ID (both from `/etc/passwd`), or an account has no password (from the last Run all, which reads `/etc/shadow` as root after your password and keeps only the account names) | How to fix and the Arch Wiki guide; Vigil does not change accounts itself, since a mistake can lock you out |
| Shell history | A history file (`~/.bash_history`, `~/.zsh_history`, `~/.histfile`, fish) is a link or not a normal file, a way attackers hide what they typed | Guide (MITRE ATT&CK); mark as reviewed if you did it yourself |
| Old network services | A telnet, rsh, rlogin, FTP, TFTP, NIS or xinetd service is switched on | **Switch them off**: `systemctl disable --now` on each one that is on (asks for your password) |

Earlier versions ran Lynis for these. Vigil now does them itself: Lynis
looks for Debian package names that never match on Arch, and most of its
hundreds of other tests are written for servers.

## How actions run

The panel always says which kind an action is:

1. **No fix**: Vigil explains and leaves it to you.
2. **Helper actions** run `bin/vigil fix <action>` as you, never as root:
   making `~/.ssh`, `~/.gnupg` and credential files private, the opt-out
   file, accepting file changes, removing leaked keys from shell history,
   the thumbnail cache, the trash, unused application caches, big cache
   folders you tick, and the Scan now and Look up now buttons. (The one
   exception is **Scan now** on Infected files, which the panel runs like a
   weekly scan, with progress, because a scan of the whole home folder takes
   a long time; the same action is still `vigil fix virus-scan-now`.) Each
   is a fixed action name; nothing from a file or from the report reaches
   it. Keeping an item of any list, moving
   leftover app data to the trash and marking false positives work the same
   way, named only by ids the helper made (`vigil keep|unkeep <id>...`,
   `vigil trash <id>...`, `vigil ignore|unignore <id>...`); Mark as
   reviewed and Show again are `vigil dismiss|undismiss <check>`.
3. **Script actions** run one of the constant scripts in
   [`Model.js`](Model.js) without a terminal. The panel first shows what it
   will do in plain words, with the exact commands under the **Run** button
   (every step that runs as root in full, as it will run), and waits for
   **Run**. Scripts that need root run through `pkexec`, so Omarchy's own
   password dialog asks; cancelling it changes nothing. The panel then shows
   what the script printed. Only tools that ask you questions as they go
   still open a terminal: Omarchy's updater (also behind the Omarchy Update button), its FIDO2 key
   and fingerprint setup, passwordless sudo, the docker group, and
   choosing an SSH passphrase. Packages Vigil installs come straight from
   the Arch repositories with `pacman --needed --noconfirm`; Bumblebee, the
   one exception, comes from Perplexity's release, checked against the
   SHA-256 in Vigil's code.
4. **Remove from a list.** **Remove** on orphaned or foreign packages runs
   one constant script through `pkexec`, with the list's name and the
   package names after it; the card shows that command line and the script
   in full. Package names are the one thing from a list that reaches a
   command, and only names Arch allows (letters, digits and `@._+-`, not
   starting with `-` or `.`) get that far. As root the script checks each
   name again, and that pacman still lists it as orphaned or foreign, and
   removes nothing unless every one passes; then it runs
   `pacman -Rns --noconfirm -- <names>`. **Remove** on plugins runs
   Omarchy's own `omarchy-plugin-remove <id> --yes` once per plugin, as
   you, after checking each id the way that tool does (never Vigil itself).
5. **Open an app or a guide**: **Open ClamUI** starts ClamUI directly, and
   **Learn more** opens a page with `xdg-open`, each with a fixed command and
   no shell. Pages come only from a fixed list in `Model.js` (Arch Wiki,
   MITRE ATT&CK, Perplexity's Bumblebee, gitleaks, rkhunter, safetensors); nothing in the report becomes a URL.
   They change nothing, so they do not ask first.
6. **Mark as reviewed**, **Show again**, **Keep** and the lock (which
   counts a kept item or a false positive again) only change Vigil's own
   notes, so they do not ask first either. **Trash** asks first, and moves
   each item with one rename into the trash; it never deletes anything.
   **False positive selected** asks first too.

Vigil never runs a file from its own folder as root (root only ever runs
`/usr/bin/bash` with one of the fixed scripts, after your password (script
actions through `/usr/bin/env HOME=…`, so the script knows your home
folder); the
package removal script also gets the package names you ticked), never
adds a sudoers rule, polkit policy, service or timer, and never changes a
setting on its own.

## What it costs your computer

Vigil runs no service, timer or daemon of its own. While you are logged in:

- **Reading your settings and files every 30 minutes** (the `refreshMinutes` setting), from
  one monitor's copy only, at lowered priority. It reads files only, and every
  folder walk has a time and size limit (a few seconds; the survey for
  leftover app data has four), so even a huge home folder cannot make it
  slow. Other monitors check only when you open their
  panel.
- **The background runs** (the daily check, and the weekly check once a
  week) at low priority; the virus scans themselves run
  at the lowest (`nice -n 19`). A background run starts at most once every
  six hours, whatever happens to it, so a run that keeps failing can never
  restart in a loop.
- **One run at a time**, across monitors and shell restarts.
- **Every step has a deadline**: the virus scan of your home folder 3 hours,
  the Downloads scan, Flatpak updates and the rootkit scan 30 minutes, the
  secrets scan of your git repositories 30 minutes in all (5 per repository),
  the AI model file scan 20 minutes, the other steps
  between 15 seconds and 10 minutes, and the panel stops anything that runs
  past its own backstop. A step that fails is retried the next day; a virus
  scan that runs out of time waits a week instead of repeating hours of
  work every day.
- **Nothing polls** except while a scan started elsewhere is running, when
  the panel asks every 4 seconds for its progress (a file read).

## Where the data comes from

Vigil keeps no threat lists of its own.

| What | Maintained by | How it stays current |
|---|---|---|
| Virus signatures | ClamAV (Cisco Talos) | ClamAV's own `clamav-freshclam` service, several times a day; checked right before every virus scan; updated by **Run all** only when over two days old |
| Known vulnerabilities in Arch packages | Arch Linux security team | arch-audit downloads the tracker as it scans, in each daily check |
| Known-malicious npm, Bun and other packages | OSV.dev and the OpenSSF Malicious Packages project | osv-scanner asks OSV.dev as it scans, in each daily check |
| AUR package status | the AUR | AUR lookup, in each daily check |
| Known-compromised packages, extensions and MCP configs | Perplexity (Bumblebee's `threat_intel` catalogs) | downloaded from Perplexity's repository right before each Bumblebee scan, in each daily check |
| Waiting system and Omarchy updates | Arch Linux and Omarchy package repositories | `checkupdates`, once a day (and **Daily check**) |
| Waiting Flatpak updates | Flathub, or whichever Flatpak remotes you added | `flatpak remote-ls --updates`, once a day (and **Daily check**) |
| Firmware updates | LVFS (hardware makers publish there) | fwupd, once a week (and **Weekly check**) |
| Keys and tokens in git repositories | gitleaks's default rules, which follow the providers' published key formats | gitleaks (from the Arch repositories, so the Omarchy update keeps it current), once a week; no network |
| Rootkits and changed system programs | rkhunter's own data (old) and pacman's records of what each package shipped | rkhunter (Arch repositories), in **Run all** only; no network |
| Pickle model files that run code | Python's own pickle disassembler and a short list of dangerous imports in Vigil's code | read locally, once a week; no network |

There is no maintained feed of malicious AUR packages; Arch's own list for
the 2026 attacks was a one-off document. Vigil covers the AUR through what
the malware pulls in (OSV), through ClamAV signatures, and through the AUR's
own package status.

## Privacy

Vigil has no telemetry and no server of its own. Its checks read local files
and ask `systemctl`, `pacman`, `journalctl` and `crontab` about local state.
With `autoUpdate` on (the default), the background checks also contact the
sources they check against, the same way the tools do when you run them by
hand:

- **Daily**: `checkupdates` downloads the package lists from your Arch and
  Omarchy mirrors (into Vigil's own folder, not the system's), and
  `flatpak remote-ls` asks your Flatpak remotes (usually Flathub) which apps
  have updates. `arch-audit` downloads the Arch security tracker from
  security.archlinux.org; `osv-scanner` sends the names and versions of the
  packages in your AUR build folders and global npm, Bun, mise and pnpm installs to
  OSV.dev (run by Google), never file contents; Vigil asks
  aur.archlinux.org about your installed AUR packages by name, as yay and
  paru do on every update; and it downloads Bumblebee's lists from
  Perplexity's repository on GitHub (a few MB).
- **Weekly**: `fwupdmgr refresh` downloads firmware information from LVFS
  (fwupd.org). The secrets scan of your git repositories (gitleaks) and the
  AI model file scan read local files and make no network calls.

Each happens only once the tool is installed, and turning `autoUpdate` off
limits it to when you press a Run, Scan now or Look up now button. Actions you start yourself also use
the network and say so first: installing a tool (Arch mirrors; Bumblebee
from Perplexity's release on GitHub), ClamAV's signatures (ClamAV's
servers), Flatpak and firmware updates, and `omarchy-update` (also behind the Omarchy Update button, which says
so on its card). The rootkit scan in Run all
reads local files and makes no network call.

## Files Vigil writes

| Path | Written by | When |
|---|---|---|
| `~/.local/state/omarchy-vigil/state.json` | the helper, mode 0600 | every time Vigil reads your settings and files: which findings you have already seen and which you marked as reviewed (as keyed fingerprints), and one score per day for 30 days |
| `~/.local/state/omarchy-vigil/baseline.json` | the helper, mode 0600 | first check, and when you accept file changes: fingerprints (not contents) of the watched files |
| `~/.local/state/omarchy-vigil/arch-audit.json` | the helper, mode 0600 | each daily check: package names and severities an update fixes |
| `~/.local/state/omarchy-vigil/osv.json` | the helper, mode 0600 | each daily check: known-malicious packages found, if any, and which folders were checked |
| `~/.local/state/omarchy-vigil/aur.json` | the helper, mode 0600 | each daily check: names of AUR packages that left the AUR, moved to the official repositories, or have no maintainer |
| `~/.local/state/omarchy-vigil/virus-scan.json` | the helper, mode 0600 | in each weekly check and on Scan now: names of infected files found in your home folder, if any, and how many files were scanned |
| `~/.local/state/omarchy-vigil/no-password.txt` | the helper, mode 0600 | each Run all: names of accounts with an empty password, usually none |
| `~/.local/state/omarchy-vigil/full-scan.json` | the helper, mode 0600 | after each weekly check, automatic or not: its date and time, for the countdown |
| `~/.local/state/omarchy-vigil/bumblebee-catalog.json` | the helper, mode 0600 | each daily check, right before the Bumblebee scan: Perplexity's list of compromised packages |
| `~/.local/state/omarchy-vigil/bumblebee-list.json` | the helper, mode 0600 | each daily check: when Bumblebee's list was last downloaded, so a scan that had to use an older one can say so |
| `~/.local/state/omarchy-vigil/signatures.json` | the helper, mode 0600 | when Vigil updates ClamAV's signatures: the date and time |
| `~/.local/state/omarchy-vigil/bumblebee.json` | the helper, mode 0600 | each daily check: the compromised packages Bumblebee found, if any |
| `~/.local/state/omarchy-vigil/progress.json` | the helper, mode 0600 | during a run: its current step, so a panel that did not start it can show it |
| `~/.local/state/omarchy-vigil/run.json` | the helper, mode 0600 | during a run: the process IDs and start times of the run and the program it is running, so **Stop** can end it |
| `~/.local/state/omarchy-vigil/handed-over/` | the Let Vigil run them action, mode 0700 | one empty marker per schedule it turned off, so Undo turns back on exactly those |
| `~/.config/clamui/settings.json` (or ClamUI's Flatpak copy) | the Let Vigil run them action, and its Undo | only `scheduled_scans_enabled`, with everything else in the file kept |
| `~/.local/state/omarchy-vigil/pending-updates.json` | the helper, mode 0600 | each daily check: how many updates are waiting, and the Omarchy version if one is |
| `~/.local/state/omarchy-vigil/firmware.json` | the helper, mode 0600 | each weekly check: device names and firmware versions with an update waiting |
| `~/.local/state/omarchy-vigil/gitleaks.json` | the helper, mode 0600 | each weekly check: the repository, file, line and rule of each possible secret, never the secret (gitleaks is asked to redact it). `gitleaks-report.json` is gitleaks's own report, written there while one repository is scanned and deleted right after |
| `~/.local/state/omarchy-vigil/model-scan.json` | the helper, mode 0600 | each weekly check: how many model files were read, and the paths and dangerous imports of those that can run code |
| `~/.local/state/omarchy-vigil/rkhunter.json` | the helper, mode 0600 | each Run all: rkhunter's warnings (text capped), and whether each matches what a package shipped |
| `~/.local/state/omarchy-vigil/downloads-scan.json`, `downloads-scan.list` | the helper, mode 0600 | each daily check: the new files in Downloads it scanned, and any it found infected |
| `~/.local/state/omarchy-vigil/confirm-scan.list` | the helper, mode 0600 | only while a scan double-checks what it found: the paths of those files, deleted when it is done |
| `~/.local/state/omarchy-vigil/flatpak.json` | the helper, mode 0600 | each daily check: Flatpak apps with updates waiting |
| `~/.local/state/omarchy-vigil/wifi-autojoin-off` | the Stop auto-joining action | the networks it changed, so Undo puts back exactly those |
| `~/.local/state/omarchy-vigil/checkup-db/` | `checkupdates`, as you | each daily check: its private copy of the package databases (a few MB) |
| `~/.local/state/omarchy-vigil/id-key` | the helper, mode 0600 | once: 32 random bytes, the key every id Vigil writes is hashed with, so `state.json`, `kept.json` and `ignored-detections.json` cannot be matched to names without it (see below) |
| `~/.local/state/omarchy-vigil/ignored-detections.json` | the helper, mode 0600 | when you mark a virus detection as a false positive: its keyed id, the file's SHA-256, its device, inode, size and times (to notice if it changes) and the date; no path or signature name |
| `~/.local/state/omarchy-vigil/kept.json` | the helper, mode 0600 | when you press Keep on any list: each item's keyed id and the date, nothing that names it. Notes from before keyed ids (including an earlier build's `kept-leftovers.json`) are carried over the next time Vigil reads your settings and files, and the names they held are dropped |
| `~/.bash_history`, `~/.zsh_history`, `~/.histfile`, `~/.local/share/fish/fish_history` | the helper, mode kept (at most 0600) | only when you press Remove those lines on Exposed secrets: the lines with keys are deleted, the rest is kept |
| `~/.local/share/Trash/files/`, `~/.local/share/Trash/info/` | the helper | only when you move a leftover to the trash: the item itself, and a `.trashinfo` file with where it came from, as every file manager writes |
| `~/.local/state/omarchy-vigil/busy/` | the helper, mode 0700, empty | once: the folder Vigil locks so only one run goes at a time |
| `/usr/local/bin/bumblebee` | the Install or Install Bumblebee button, as root | only when you press one: Perplexity's program, version 0.1.2, checked against the SHA-256 in Vigil's code |
| `~/.config/uwsm/env.d/90-omarchy-vigil-opt-out` | the helper | only when you set the opt-out variables |
| `~/.config/uwsm/env.d/90-omarchy-vigil-agents` | the helper | only when you switch off AI agent telemetry |
| `~/.local/state/omarchy/settings/weather.json` | the helper | only when you press Round it: `latitude` and `longitude` only, everything else kept |
| `/etc/hosts` | a script action, as root | only when you block the telemetry addresses: one block between `# BEGIN omarchy-vigil telemetry` and `# END omarchy-vigil telemetry`; Undo takes out exactly that block |
| `/etc/chromium/policies/managed/90-omarchy-vigil.json` | a script action, as root | only when you set Chromium's privacy policy |
| `/var/lib/rkhunter/db/rkhunter.dat` | rkhunter, as root | the first Run all with rkhunter installed: its baseline of your system programs (`rkhunter --propupd`); rkhunter's own log is switched off |
| `/etc/sysctl.d/90-omarchy-vigil.conf` | a script action, as root | only when you apply the kernel settings |
| `/etc/systemd/resolved.conf.d/90-omarchy-vigil-llmnr.conf` | a script action, as root | only when you turn LLMNR off |
| `/etc/systemd/resolved.conf.d/90-omarchy-vigil-fallback.conf` | a script action, as root | only when you remove the DNS fallback |
| `/etc/ssh/sshd_config.d/00-omarchy-vigil.conf` | a script action, as root | only when you harden SSH |
| `/etc/NetworkManager/conf.d/90-omarchy-vigil-connectivity.conf` | a script action, as root | only when you turn the connectivity check off |
| `/etc/NetworkManager/conf.d/90-omarchy-vigil-mac.conf` | a script action, as root | only when you change the hardware address setting |
| `/etc/modprobe.d/90-omarchy-vigil-firewire.conf` | a script action, as root | only when you turn FireWire off |
| `/etc/modprobe.d/90-omarchy-vigil-protocols.conf` | a script action of an earlier Vigil, as root | only if you turned rarely used network protocols off before 0.12; its Undo still removes it |
| `/etc/avahi/avahi-daemon.conf` (first original kept at `avahi-daemon.conf.vigil-backup`) | a script action, as root | only when you stop Avahi announcements |

Every file in `/etc` that Vigil creates starts with the line
`# Written by Vigil (keasbeexd.vigil). Delete this file to undo.` ClamUI, the scanners and the Omarchy commands Vigil launches keep their own
files; see their documentation.

**Ids are keyed.** What you kept, marked as a false positive or reviewed is
stored as an id: a hash made with this install's own random key (`id-key`),
not a plain hash of a name. Package and plugin names are public, so a plain
hash could be turned back into the name by trying every name; a keyed one
cannot without the key. So these notes, a backup of them, or another account
reading Vigil's command line while it runs, do not reveal what you trust. It
does not stop a program already running as you, which can read the key as
Vigil does (and could change Vigil itself); a kept item therefore always
stays on its list, with a lock. What a scan found (paths, package names) is
kept as is, since the panel shows it. The
[security model](docs/SECURITY-MODEL.md#ids) has the details.

## Removing

```sh
omarchy plugin remove keasbeexd.vigil
```

This removes the plugin folder. Vigil runs no services, timers or daemons of
its own, and a run that is going when you remove it stops within
seconds, so nothing of Vigil's keeps running. These stay behind until you
remove them:

- `~/.local/state/omarchy-vigil/` (seen findings, score history, the file
  baseline and the cached results of the checks). It holds nothing you need
  without Vigil; delete the folder in your file manager to remove it. If you
  used **Let Vigil run them** and want ClamUI's schedule back, press its
  Undo first, and likewise **Undo** on **Stop auto-joining** if you want
  those Wi-Fi networks to join by themselves again.
- The opt-out files, if you set them. Remove them from the Privacy tab before
  uninstalling, or delete `~/.config/uwsm/env.d/90-omarchy-vigil-opt-out` and
  `~/.config/uwsm/env.d/90-omarchy-vigil-agents`. The rounded weather
  location stays rounded.
- Settings you applied in `/etc`. They are your system settings now and keep
  protecting you after Vigil is gone. To undo one, delete its file from the
  table above with `sudo rm <file>`, then reboot (or restart the service
  named in the table). For the telemetry block in `/etc/hosts`, use **Undo**
  on the Telemetry hosts finding, or delete the lines between Vigil's two
  markers, markers included. For Avahi, restore the original with
  `sudo mv /etc/avahi/avahi-daemon.conf.vigil-backup /etc/avahi/avahi-daemon.conf`.
  Easier: use **Undo** on each finding before you remove Vigil.
- Apps and scanners you installed through Vigil. Remove ClamUI with
  `omarchy pkg drop clamui` (or `flatpak uninstall --user io.github.linx_systems.ClamUI`
  if you installed it from Flathub), and `clamav`,
  `arch-audit`, `osv-scanner`, `fwupd`, `gitleaks` or `rkhunter` with
  `omarchy pkg drop <name>` (and `lynis`, if an earlier Vigil installed it).
  rkhunter keeps its baseline in `/var/lib/rkhunter` after it is removed. Remove Bumblebee with
  `sudo rm /usr/local/bin/bumblebee`.
- Services Vigil switched on for you. They belong to their packages and keep
  protecting you without Vigil: ClamAV's update service stops with
  `sudo systemctl disable --now clamav-freshclam.service`, and the firewall,
  if you turned it on here, with `sudo ufw disable`.

## Requirements

- Omarchy with the Quattro shell.
- `/usr/bin/python3` (Omarchy ships it). Standard library only.

## Development

```sh
python3 -m pytest tests -q          # helper checks, fixes, and marketplace rules
node --test tests/*.test.mjs        # Model.js validation and terminal scripts
tests/lint.sh                       # syntax: the helper, Model.js, and Panel.qml where qmllint is
omarchy plugin validate .
```

The QML cannot be run outside Omarchy: it imports `qs.Commons` and `qs.Ui`,
which come from the running shell. Qt 6's `qmllint` (`/usr/lib/qt6/bin/qmllint`
on Omarchy, which `tests/lint.sh` finds; the Qt 5 one in `/usr/bin` cannot read
Qt 6 QML) still checks `Panel.qml` for syntax errors, and reports the shell's
own types as unresolved, which is expected.

The same tests run on every push to `master` and every pull request, in a
locked-down GitHub Actions job (no token permissions, no third-party
actions, dependencies pinned by hash), and `.github/CODEOWNERS` names the
owner as reviewer of every file. Found a security problem? See
[SECURITY.md](SECURITY.md) and report it privately.

## Credits

Vigil is new code, but it stands on the work of the projects it brings
together, all MIT licensed:
[Black Ops](https://github.com/brightwalker25/omarchy-black-ops) (privacy
protections and its research on Omarchy's defaults: Vigil's telemetry
blocking, Chromium policy, weather rounding and opt-out lists are the same
ideas written again as one-time buttons, with no watchdog or listener),
[omarchy-security-scan](https://github.com/brightwalker25/omarchy-security-scan)
(the gitleaks and rkhunter scans, and the idea of checking a model file's
pickle for code),
[Security Scan](https://github.com/Percius04/security-scan) (scheduled
scanners and the startup-file heuristics),
[omarchy-bumblebee](https://github.com/duketopceo/omarchy-bumblebee) (the
idea of bringing Perplexity's Bumblebee to Omarchy), [ClamUI](https://github.com/linx-systems/clamui) (virus scanning
and quarantine), and [SysCare](https://github.com/cytracon/omarchy-syscare)
(cleaning). The reasoning about root in
[omarchy-firewall](https://github.com/illegalstudio/omarchy-firewall) shaped
how Vigil asks for it.

Known-compromised packages are found by Perplexity's
[Bumblebee](https://github.com/perplexityai/bumblebee) (Apache 2.0) and its
`threat_intel` catalogs. Vigil does not ship either: **Install** downloads
the version Vigil pins from Perplexity's release on GitHub, and the catalogs
come from Perplexity's repository.

## License

MIT. See [LICENSE](LICENSE).
