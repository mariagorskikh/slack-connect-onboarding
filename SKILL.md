---
name: slack-connect-onboarding
description: Set up a private shared Slack Connect channel with a customer, prospect or partner, add your internal team, invite their people by email, and post a short, warm welcome with real next steps pulled from meeting notes and the email thread. Use whenever the user says "add them to Slack", "create a shared channel with X", "Slack Connect", "invite them to our Slack", "open a channel with <company>", "they want to join our Slack", or forwards an email where someone asks to join a shared channel.
---

# Slack Connect onboarding

Goal: one private shared channel per outside company, the right people in it, and a first message
that tells them exactly what happens next. Nothing is created, invited or posted until the user
says yes to the exact plan.

## 0. Load the config

Read `config.yaml` in this skill's folder. If it is missing, read `config.example.yaml` and tell
the user to copy it to `config.yaml` and fill it in. For any value you need that is blank or
missing, ask the user once, in a single short list. Do not guess workspace details.

Values you use:
- `channel.name_pattern`, `channel.private`, `channel.max_length`
- `team.members` (people from your workspace to add to every channel)
- `team.mention_in_welcome` (who to @mention as the people who can help)
- `welcome.*` (sign off, tone notes, booking link, word limits, facts file)
- `sources.meeting_notes` and `sources.email`
- `posting.method` (`ui` or `mcp`)

## 1. Gather (read only)

**Who to invite.** Use the outside emails the user gave. If they named only a company, take the
people from the email thread or the calendar invite, and say who you picked and why.

**What they need.** Read these in order. When sources disagree, the newest one wins.
1. Meeting notes for the most recent call with them, from `sources.meeting_notes`:
   - `granola`, `fireflies`, `otter`, `zoom`: use that tool's MCP or connector to find the meeting
     by company name, attendee email or date, then read the notes. If the notes are thin and a
     transcript tool is available, search the transcript for asks and promises.
   - `google_docs`: open the doc the user points to, or search Drive for the company name.
   - `pasted`: ask the user to paste the notes.
2. The email thread, from `sources.email`: `gmail` (search by their domain) or `pasted`.
3. Anything else the user says in chat.

From those, write down:
- What they asked for (a doc, an access level, a region, a price, a demo, an integration).
- What your side promised, who promised it, and by when.
- What they said they would do or send.
- Whether a next call is already scheduled.

**Already there?** Search Slack for a channel that matches the name pattern for this company. If
one exists, plan to invite into it instead of creating a new one.

## 2. Build the plan and the welcome text

**Channel name.** Fill `channel.name_pattern`. Slack rules: lowercase, no spaces or periods, only
letters, numbers, hyphens and underscores, 80 characters max. Strip "Inc", "Ltd", "LLC" and the
like. Example: pattern `{our_company}-{customer}` with Acme and Northwind Logistics gives
`acme-northwind`.

**Welcome message.** Write it now so the user approves the exact words. Rules:
- Short. Stay inside `welcome.min_words` and `welcome.max_words` (default 40 to 90).
- Warm, in the user's voice. Follow `welcome.tone_notes`.
- Concrete next steps: who does what, by when. Every step comes from the notes, the thread or the
  user. Never invent a date, a price, a promise or a feature.
- Answer everything they asked for, or say when the answer is coming.
- If something needs a decision from the user (price, discount, credits, legal terms, a date you
  do not have), leave it out of the message and flag it in the plan.
- Product facts only from `welcome.facts_file` if one is set, or from what the user tells you.
- If a call is already on the calendar, name the day and time. Otherwise add `welcome.booking_link`
  if one is set.
- @mention the people in `team.mention_in_welcome` once, as the people who can help.
- No filler. Do not write "I hope this finds you well", "excited to partner", "don't hesitate to
  reach out", "seamless", "leverage", "delighted", "at your earliest convenience", "circle back".
- No em dashes or en dashes. Use commas, periods or "to".
- No bullet lists unless there are three or more separate next steps.

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

## 3. Approval gate (required)

Show the user this block and stop:

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
the updated block again and wait again. Approval covers this plan only, not later changes.

## 4. Create the channel and add the team

Slack MCP (if available): create a conversation with the channel name, `is_private` from config,
and the user IDs of `team.members`. Look up user IDs by handle or email with the MCP's user search;
never hardcode them. The person running the MCP is added as the creator.

No MCP: create it in the Slack UI in a browser. Channels, Create a channel, name it, set it to
private, then add the team members.

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

## 6. Post the welcome

Post the exact approved text. Posting order: create channel, add team, send invites, then post.
That way the invite preview and the channel history both show the welcome.

- `posting.method: ui` (default): type the message in the Slack UI message box. Pick each
  @mention from the autocomplete so it becomes a real mention, then send.
- `posting.method: mcp`: post with the Slack MCP send message tool. Some Slack MCP servers add a
  footer like "Sent using <app>" under the message. If you see one and the user does not want it,
  switch to `ui`.

## 7. Verify and report

- Check the channel: the message is posted, mentions resolved, the team is in.
- Check that each invite shows as pending (members list or the channel's pending invitations).
  If an invite did not go out, say so and retry once.
- Tell the user: channel name, who was added, who was invited, the message as posted, and the
  items you left out for them.
- If the user keeps a CRM and has a tool for it, offer to log a note like "Shared Slack channel
  #<name> created, invited <emails>". Do not write to it without a yes.

## Gotchas

- **Slack Connect needs acceptance.** The outside person gets an email invite. Their workspace (or
  they, if they have none) must accept, and their admin may need to approve. Until then the
  channel shows them as pending. That is normal, not a failure.
- **Paid plan required.** Slack Connect works on paid Slack plans. On a free workspace, the invite
  option may not appear or may fall back to guests.
- **Guests vs Slack Connect.** Guests join your workspace and count against your seats and admin
  settings. Slack Connect keeps each company in its own workspace. Pick Slack Connect for
  customers unless told otherwise.
- **Admin policies.** Your workspace may require admin approval for external invites or block
  certain domains. If the invite is held for approval, tell the user who needs to approve it.
- **Private channel names.** Names cannot be changed by outside members, and renaming later
  changes the name for both sides. Get it right before creating.
- **Existing channel.** Always check first. Two channels with the same customer splits the
  conversation.
- **Mentions.** Typed "@Name" text without picking from autocomplete is plain text and notifies no
  one.
- **MCP footer.** Some Slack MCP servers stamp a "sent using" line on messages. Post through the UI
  if that matters.
- **Do not paste secrets.** Never put API keys, passwords or promo codes that are not live into a
  shared channel.
