**To:** Priya Anand — VP, Customer Operations, Meridian
**From:** Partner Delivery Team
**Subject:** Re: Honestly not sure this is going to work — it's fixable, and here's the proof

Hi Priya,

Straight answer to your question: **this is fixable, and it wasn't the model.** Here's what we found on T-4471 and what we did about it.

---

**What was actually breaking it**

Not the model's reasoning — the instructions we gave it. The agent's coordinator was explicitly told to hand every ticket to exactly one specialist, even when a ticket raised more than one issue, and to always mark the ticket "resolved" once that one specialist finished.

On T-4471, the customer raised two separate problems: an SSO lockout (40 people couldn't log in) and a $1,200 billing refund. The agent correctly diagnosed *both* — it even found the exact root cause of the SSO issue (an expired login certificate) and confirmed the refund was valid in the audit log. But because it was only allowed to engage one specialist per ticket, it routed to the account specialist, told the customer to go email billing themselves about the refund, and then closed the ticket as fully "resolved" anyway. That's the exact pattern your team saw last week: confidently wrong, and marked done when it wasn't.

**Is it the model?** We tested that directly before touching anything — reran the identical ticket on a significantly larger, more expensive model. Same result: 0 for 5 resolved, at more than double the cost per ticket. A bigger model doesn't help if it's still following an instruction that tells it to drop the second issue on the floor.

**The fix**

One rule, in the coordinator's instructions (not the model, not the specialists' own logic): instead of "route every ticket to exactly one specialist," it now reads "route each distinct issue to its own specialist." When a ticket has two problems, the agent now engages two specialists and folds both resolutions into one reply — instead of quietly dropping whichever issue didn't fit.

**Proof**

| | Before | After |
|---|---|---|
| T-4471 resolved | 0 / 5 runs | 5 / 5 runs |
| Held-out tickets (3 different tickets, different issue types) | — | 9 / 9 runs |
| Cost per ticket | ~$0.12 | ~$0.17 |

The held-out tickets are ones the fix was never tuned against — different customers, different issue combinations, including one single-issue ticket to confirm we didn't break the simple case. All resolved cleanly.

**Why not just pay for a better model?**

We already ran that experiment: the larger model produced the identical failure (0/5 resolved) at $0.27 per ticket — more than 2x today's cost, and still more than 1.5x what the fixed prompt costs now. The issue was never model capability; it was an instruction that capped the agent at one specialist per ticket regardless of how many problems the customer actually raised. Paying more for a bigger model buys you the same bug at a higher price.

**What it would take**

This was a single-sentence change to one file (the coordinator's routing instructions) — no new tools, no model change, no changes to the specialist agents themselves. Recommend two follow-ups before calling this closed:
1. Spot-check a broader sample of live tickets (beyond the 4 we've tested) to confirm the same fix holds at volume.
2. Add this "did we actually resolve everything the customer raised" check to ongoing monitoring, not just this one-off diagnosis, so a regression like this surfaces immediately next time rather than three weeks in.

Happy to walk your leadership through this directly if it'd help — the before/after numbers above are the whole story.

— Partner Delivery Team
