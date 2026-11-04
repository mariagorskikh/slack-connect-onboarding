# slack-connect-onboarding

A Claude Code skill that opens a shared Slack Connect channel with a customer, prospect or partner,
adds your team, invites their people by email, and posts a short welcome with real next steps.

The next steps come from your meeting notes, the email thread and what the customer asked for. No
generic filler. Nothing is created, invited or posted until you approve the exact plan.

## What it does

1. Reads your config (channel naming, team, tone, where notes and email live).
2. Finds the latest call notes and the email thread for that company.
3. Pulls out what they asked for, what you promised, and any dates.
4. Checks whether a channel already exists.
5. Shows you the channel name, members, invitees and the exact welcome text.
6. After your yes: creates the private channel, adds your team, sends Slack Connect invites one
   email at a time, posts the welcome, and checks that the invites went out.
7. Reports back with anything it left out for you to decide (price, dates, legal).

## Requirements

- A Slack workspace on a paid plan with Slack Connect enabled. Your admin may need to allow
  external invites.
- A way for Claude to use Slack:
  - a Slack MCP server for searching users and creating channels, and
  - a browser tool (for example Claude in Chrome) for sending Slack Connect invites and, by
    default, posting the message. Most Slack MCP servers cannot invite outside emails.
- Optional: a meeting notes tool (Granola, Fireflies, Otter, Zoom, or Google Docs) and Gmail
  access. Without them, paste the notes and the thread into chat.

## Install

```bash
git clone https://github.com/mariagorskikh/slack-connect-onboarding ~/.claude/skills/slack-connect-onboarding
```

## Configure

```bash
cd ~/.claude/skills/slack-connect-onboarding
cp config.example.yaml config.yaml
```

Edit `config.yaml`:

- `channel.name_pattern` and `channel.our_company`: for example `{our_company}-{customer}` gives
  `acme-northwind`.
- `team.members`: Slack handles or emails of people to add to every channel.
- `team.mention_in_welcome`: who to @mention as the people who can help.
- `welcome.tone_notes`, `welcome.booking_link`, word limits, and an optional facts file.
- `sources.meeting_notes`: `granola`, `fireflies`, `otter`, `zoom`, `google_docs` or `pasted`.
- `sources.email`: `gmail` or `pasted`.
- `posting.method`: `ui` (default) or `mcp`.

If something is missing, Claude will ask.

## Example prompt

```
Add Dana (dana@northwind.com) and Priya (priya@northwind.com) from Northwind to Slack.
We had a call with them on Tuesday.
```

Claude replies with a plan like this and waits:

```
Channel:   #acme-northwind  (private, new)
Add from our team:  Sam Rivera, Lee Park
Invite (Slack Connect):  dana@northwind.com, priya@northwind.com
Welcome message:
Hi Dana and Priya! Welcome! Great talking on Tuesday about moving your nightly route exports off
spreadsheets. Sam will send the API sandbox keys here by Thursday. Once you have them, send over
two or three sample export files and we will map the fields with you next week. Feel free to add
anyone on your data team. @Sam Rivera and @Lee Park are here for any questions. Our next call is
Monday at 10am PT.

Left out for you to decide:  their question about annual pricing
Sources used:  "Northwind intro" call (Tuesday), email "Re: route exports"
```

Say "yes" and it runs.

## How it works

- **Sources, newest wins.** Meeting notes first, then the email thread, then what you say in chat.
- **No invented facts.** Every date, owner and promise in the welcome must come from a source.
  Anything that needs your call is flagged, not guessed.
- **Approval gate.** You see the exact channel, people and words before anything happens.
- **One invite per pass.** Slack can drop a second email typed too fast, so invites go one at a
  time.
- **UI posting by default.** Some Slack MCP servers add a "sent using" footer. Typing in the Slack
  UI avoids it and makes @mentions real.

## Good to know

- Outside people must accept the Slack Connect invite, and their admin may need to approve it.
  They show as pending until then.
- Guests and Slack Connect are different. Guests join your workspace. Slack Connect keeps each
  company in its own workspace. The default is Slack Connect.
- Check for an existing channel first. The skill does this for you.

## License

MIT. See [LICENSE](LICENSE).
