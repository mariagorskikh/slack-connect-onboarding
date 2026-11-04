---
name: slack-connect-onboarding
description: Set up a private shared Slack Connect channel with a customer, prospect or partner, add your internal team, invite their people by email, and post a short, warm welcome with real next steps pulled from meeting notes and the email thread. Use whenever the user says "add them to Slack", "create a shared channel with X", "Slack Connect", "invite them to our Slack", "open a channel with <company>", "they want to join our Slack", or forwards an email where someone asks to join a shared channel.
---

# Slack Connect onboarding

Goal: one private shared channel per outside company, the right people in it, and a first message
that tells them exactly what happens next.

**How this skill runs.** It asks its setup questions once. After that it does the whole job itself:
gathers sources, writes the channel name and welcome, checks its own work, creates the channel,
adds the team, sends the invites, posts, verifies, and reports after. There is no approval step,
except for money.

**Money is the only approval.** Pricing, discounts, credits or anything commercial never goes in
the welcome. It is left out and listed under "Left out for you to decide". If creating the channel
or sending Slack Connect invites would require upgrading the Slack plan (or any other purchase),
stop and ask the user. Do not click any upgrade, trial or billing button.

**Hard stops (not approvals, stops).** Halt that action, finish whatever else is safe, and report:
- An external invite is held for admin approval.
- The workspace is not on a paid plan, or Slack Connect is not available.
- Any Slack warning, limit notice, error banner, CAPTCHA, login, 2FA or re-auth prompt.
- Any payment, billing or upgrade form.
- A fact for the welcome that no source supports. Leave it out, never invent it.

If `autonomy.mode` is `review`, follow the "Review mode" section at the end instead of acting
straight away.

## 0. Load the config (ask once)

Read `config.yaml` in this skill's folder. If it is missing, copy `config.example.yaml` to
`config.yaml`.

On the first run, or when a needed value is blank, ask the setup questions in one short batch:
1. Your company name for the channel pattern (`channel.our_company`, `channel.name_pattern`).
2. Team members to add to every channel, and who to @mention in the welcome.
3. Tone notes, sign off, and a booking link (optional).
4. Where meeting notes live (`granola`, `fireflies`, `otter`, `zoom`, `google_docs`, `pasted`) and
   where email lives (`gmail`, `pasted`).

Write the answers to `config.yaml`, then continue without more questions. Do not guess workspace
details.

Later runs ask nothing, with one exception: if the user named only a company and no outside emails
can be found in the thread, the calendar invite or the notes, ask one short question: "Who should I
invite from <company>?" Keep doing everything else (gathering, drafting, checking for an existing
channel) while you wait.

Values you use:
- `autonomy.mode`, `autonomy.money_needs_approval`
- `channel.name_pattern`, `channel.our_company`, `channel.private`, `channel.max_length`
- `team.members`, `team.mention_in_welcome`
- `welcome.*` (sign off, tone notes, booking link, word limits, facts file)
- `sources.meeting_notes`, `sources.email`
- `invites.type`, `posting.method`

## 1. Gather (read only)

**Who to invite.** Use the outside emails the user gave. If they named only a company, take the
people from the email thread or the calendar invite, and note who you picked and why for the report.

**What they need.** Read these in order. When sources disagree, the newest one wins.
1. Meeting notes for the most recent call with them, from `sources.meeting_notes`:
   - `granola`, `fireflies`, `otter`, `zoom`: use that tool's MCP or connector to find the meeting
     by company name, attendee email or date, then read the notes. If the notes are thin and a
     transcript tool is available, search the transcript for asks and promises.
   - `google_docs`: open the doc the user points to, or search Drive for the company name.
   - `pasted`: use what the user pasted. If nothing was pasted, ask for it in the setup batch.
2. The email thread, from `sources.email`: `gmail` (search by their domain) or `pasted`.
3. Anything else the user says in chat.

From those, write down:
- What they asked for (a doc, an access level, a region, a demo, an integration).
- What your side promised, who promised it, and by when.
- What they said they would do or send.
- Whether a next call is already scheduled.
- Anything commercial (price, discount, credits, contract terms). These go to "Left out for you to
  decide", never into the welcome.

**Already there?** Search Slack for a channel that matches the name pattern for this company. If
one exists, invite into it instead of creating a new one.

## 2. Build the channel name and the welcome

**Channel name.** Fill `channel.name_pattern`. Slack rules: lowercase, no spaces or periods, only
letters, numbers, hyphens and underscores, 80 characters max. Strip "Inc", "Ltd", "LLC" and the
like. Example: pattern `{our_company}-{customer}` with Acme and Northwind Logistics gives
`acme-northwind`.

**Welcome message.** Rules:
- Short. Stay inside `welcome.min_words` and `welcome.max_words` (default 40 to 90).
- Warm, in the user's voice. Follow `welcome.tone_notes`.
- Concrete next steps: who does what, by when. Every step comes from the notes, the thread or the
  user. Never invent a date, a promise or a feature.
- Answer everything they asked for, or say when the answer is coming.
- Nothing about money: no price, discount, credits or plan terms, even if a source has them. List
  them under "Left out for you to decide". The same goes for legal terms and dates you do not have.
- Product facts only from `welcome.facts_file` if one is set, or from what the user tells you.
- If a call is already on the calendar, name the day and time. Otherwise add `welcome.booking_link`
  if one is set.
- @mention the people in `team.mention_in_welcome` once, as the people who can help.
- No filler. Do not write "I hope this finds you well", "excited to partner", "don't hesitate to
  reach out", "seamless", "leverage", "delighted", "at your earliest convenience", "circle back".
- No em dashes or en dashes. Use commas, periods or "to".
- No bullet lists unless there are three or more separate next steps.
- No secrets: no API keys, passwords, or promo codes.

Template:

> Hi {first names}! Welcome! {One line tied to what they said or asked for.} {Next step 1: who,
> what, when.} {Next step 2 if there is one.} {One useful pointer if it helps, like where to start
> or what to send.} {Who can help, with @mentions.} {Booking link or the scheduled call time.}

Fictional example (Acme is your company, Northwind is the customer, call was on Tuesday):

> Hi Dana and Priya! Welcome! Great talking on Tuesday about moving your nightly route exports off
> spreadsheets. Sam will send the API sandbox keys here by Thursday. Once you have them, send over
> two or three sample export files and we will map the fields with you next week. Feel free to add
> anyone on your data team. @Sam Rivera and @Lee Park are here for any questions. Our next call is
> Monday at 10am PT.

## 3. Self-review gate (required before acting)

Check every item. Fix and re-check until all pass. Only then go to step 4.
- Every fact, date, owner and promise in the welcome traces to a named source. Remove anything that
  does not.
- Nothing commercial is in the welcome (price, discount, credits, plan, contract terms).
- Word count is inside `welcome.min_words` to `welcome.max_words`.
- No filler phrases, no em or en dashes, no secrets.
- Every link works and is the real URL (booking link from config, not a redirect).
- Re-read it as the outside person would: is the next step clear, and does it answer what they
  asked?
- Channel name follows the pattern and Slack rules, and you checked for an existing channel.
- Invitee emails come from the user or a source, and match the company's domain or the thread.

## 4. Create the channel and add the team

Slack MCP (if available): create a conversation with the channel name, `is_private` from config,
and the user IDs of `team.members`. Look up user IDs by handle or email with the MCP's user search;
never hardcode them. The person running the MCP is added as the creator.

No MCP: create it in the Slack UI in a browser. Channels, Create a channel, name it, set it to
private, then add the team members.

If an existing channel was found, skip creating and add any missing team members to it.

## 5. Invite the outside people

Most Slack MCP servers cannot send Slack Connect invites to outside emails. Use the Slack UI in a
browser:

1. Open the channel.
2. Open the channel's add people or members dialog.
3. Type one email, press Return, confirm it.
4. When Slack asks, choose that the person is from an external organization (Slack Connect), not
   a guest. Send the invitation.
5. Repeat for the next email. Do one email per pass. Typing a second email before Slack finishes
   checking the first one can drop it.

If `invites.type` is `guest`, invite them as guests instead. The default is `slack_connect`.

Stop here (hard stop or money) if Slack shows an upgrade prompt, a trial offer, a "requires a paid
plan" notice, an admin-approval hold, or any warning. Do not fall back to guests on your own.

## 6. Post the welcome

Post the reviewed text. Order: create channel, add team, send invites, then post. That way the
invite preview and the channel history both show the welcome.

- `posting.method: ui` (default): type the message in the Slack UI message box. Pick each
  @mention from the autocomplete so it becomes a real mention, then send.
- `posting.method: mcp`: post with the Slack MCP send message tool. Some Slack MCP servers add a
  footer like "Sent using <app>" under the message. If you see one, note it in the report.

## 7. Verify and report

Verify:
- The message is posted, mentions resolved, the team is in.
- Each invite shows as pending (members list or the channel's pending invitations). If an invite
  did not go out, retry once, then report it.

Report in this format, after acting:

```
Channel:   #<name>  (private, new | existing)
Added from our team:  <names or handles>
Invited (Slack Connect):  <email 1> (pending), <email 2> (pending)
Welcome message (as posted):
<exact text>

Left out for you to decide:  <pricing, discounts, credits, legal terms, missing dates, or "nothing">
Skipped or stopped:  <anything not done and why, or "nothing">
Sources used:  <meeting title and date>, <email thread subject>
```

If the user keeps a CRM and you have a tool for it, log a note like "Shared Slack channel #<name>
created, invited <emails>" and mention it in the report.

## Review mode

When `autonomy.mode` is `review`, run steps 0 to 3, then show the user this block and stop:

```
Channel:   #<name>  (private, new | existing)
Add from our team:  <names or handles>
Invite (Slack Connect):  <email 1>, <email 2>
Welcome message:
<exact text, exactly as it will be posted>

Left out for you to decide:  <items, or "nothing">
Sources used:  <meeting title and date>, <email thread subject>
```

Wait for an explicit yes ("yes", "go", "post it", "looks good"). If the user edits anything, show
the updated block again and wait again. Approval covers this plan only, not later changes. Then run
steps 4 to 7. In review mode, also ask before writing to a CRM.

## Gotchas

- **Slack Connect needs acceptance.** The outside person gets an email invite. Their workspace (or
  they, if they have none) must accept, and their admin may need to approve. Until then the
  channel shows them as pending. That is normal, not a failure.
- **Paid plan required.** Slack Connect works on paid Slack plans. On a free workspace, the invite
  option may not appear, may fall back to guests, or may show an upgrade prompt. That is a money
  stop: ask the user.
- **Guests vs Slack Connect.** Guests join your workspace and count against your seats and admin
  settings. Slack Connect keeps each company in its own workspace. Pick Slack Connect for
  customers unless config says otherwise.
- **Admin policies.** Your workspace may require admin approval for external invites or block
  certain domains. If the invite is held for approval, stop and report who needs to approve it.
- **Private channel names.** Names cannot be changed by outside members, and renaming later
  changes the name for both sides. The self-review gate checks the name before creating.
- **Existing channel.** Always check first. Two channels with the same customer splits the
  conversation.
- **Mentions.** Typed "@Name" text without picking from autocomplete is plain text and notifies no
  one.
- **MCP footer.** Some Slack MCP servers stamp a "sent using" line on messages. Post through the UI
  if that matters.
- **Do not paste secrets.** Never put API keys, passwords or promo codes into a shared channel.
