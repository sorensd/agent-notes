# Messages the user will paste into a chat app

Status updates, client replies and WhatsApp drafts the user forwards verbatim. The agent's
job is the exact text; the user's job is one copy and one paste.

---

### The message arrives broken, or cannot be copied at all

**Symptom.** The user asks for "the WhatsApp message", gets a nicely formatted reply, and
answers "that is not WhatsApp formatting" or "full broke, couldn't be copy pasted". Three
attempts later it still is not right.

**Cause.** The reply was written as Markdown for the terminal or chat UI: `##` headings,
`**bold**`, `>` quote bars, nested bullets. The UI renders it, so the agent sees a clean
message, but what the user copies is either the rendered text with the structure lost, or
the raw Markdown with symbols WhatsApp does not understand. Quote bars in particular drop
the line breaks on copy, so paragraphs run together.

**Fix.** Put the whole message inside one fenced code block, written in the target app's own
markup, and nothing else in the fence:

- WhatsApp: `*bold*`, `_italic_`, `~strike~`, plain `-` bullets, `1.` numbers, blank lines
  between sections. No `#` headings, no `**`, no `>` quotes, no nested bullets, no tables.
- Slack: `*bold*`, `_italic_`, bullets with `•` or `-`. Same bans.
- Email or SMS: plain text, no markup at all.

The fence is what makes copy work: the UI shows it verbatim and a copy button copies it
verbatim. Mention the app's conventions once ("WhatsApp formatting") above the fence, never
inside it.

Shape of the message itself, when it is a status update:

1. one line of what the message is and the date
2. DONE, grouped by area the reader cares about (receiving, shipping, pay page), each bullet
   "what changed for you", no ticket numbers, no build internals beyond one build stamp
3. RUNNING, numbered, each with "nothing for you to do" or the one thing they must do
4. one closing line: when it is finished

**Why it was hard to find.** The agent's own view of the reply looked correct every time.
The failure is only visible on the user's clipboard, and the user's feedback ("not WhatsApp
formatting", "broke") does not say which symbol broke it. The tell is the request itself:
any message the user will forward to a third person must be delivered as copyable text,
not as a rendered reply.

Related: [delivery-rules.md](delivery-rules.md) (the demo path and the What's New are also
written for the person using the screen, not the person who wrote the code).
