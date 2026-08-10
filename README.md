# TKI Trust Admin

Prototype tool for administering **TKI, a pooled minors trust with 31 beneficiary
accounts**, where BrookHaven is agent for trustee. Owner: **Will**. Technical owner: **the
CTO**. Destined to be absorbed into [Haven](https://github.com/BrookHavenTax/haven).

---

## Will — start here

Three steps. That's the whole setup.

**1.** Make an empty folder called **`tki-trust-admin`** anywhere you like — Documents is fine.

**2.** Open **Claude Code** in that folder.

**3.** Open **[PROMPT.md](PROMPT.md)** → click the **Raw** button → select all (`Cmd+A`) →
copy (`Cmd+C`) → paste into Claude Code as your first message.

That file contains nothing but the prompt, so you can copy the whole thing without picking
out a section. It is long. Paste all of it anyway, in one go.

**That is genuinely everything you have to do.** No Terminal, no installing, no restarting,
no second prompt. It sets up your Mac, connects the project, writes its own safety rules,
and then gets to work. If something needs an administrator password it will route around it
and flag it for the CTO rather than stopping you — so **never type an admin password, API
key, or token into the chat.** Those come from the CTO.

Setup will take a few minutes and scroll past a lot of text. You can ignore all of it. The
part meant for you starts when it begins asking about the trust.

**That's the real work, and it's the part only you can do.** Your job here is checking its
understanding of the trust, not its code. You're the only person on this project who knows
whether "the addendum is expressly incorporated" has been read correctly, and that's worth
more than any technical review.

**Three things to never do:** give it a real file or scan (the records contain minors'
Social Security numbers), point it at the shared archive folder, or let it talk you into a
workaround when it says it needs a database. Databases, deployment, the Google Gemini API
key, Dropbox and QuickBooks are all the CTO's — just say "the CTO handles that" and move on.

The tool does read scanned documents, using Google Gemini — but in this phase it only ever
reads **practice scans it generated itself**, never a real file from the archive.

Useful things to say: *"Explain that in plain English."* · *"That's wrong — in the master
trust, X actually means Y."* · *"Stop, let's finish what we started before adding
anything."* · *"Run the tests and show me they pass."* · *"What did you assume here that I
haven't confirmed?"* — ask that last one at the end of every session.

**Coming back later?** You never paste this again. Just open Claude Code in the folder and
say *"read CLAUDE.md and .claude/handoff.md, then tell me where we left off."*
