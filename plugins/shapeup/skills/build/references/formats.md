# Item and comment formats

Item types, scope fields, tags and states are written the way the tracker adapter says. The content below is the
same for every tracker.

## Slice states

| State | Meaning | Set when |
|---|---|---|
| `New` | Discovered, not started | You create the slice |
| `Active` | Being worked on; may have a draft pull request | You start working on it |
| `Resolved` | Fully implemented, pull request ready for review | You publish its pull request |
| `Closed` | Approved and merged, or cut | You merge it, or cut it with a comment |

## Slice

- Type: the tracker's slice item, child of the bet
- Scope fields: as the tracker adapter lists
- Title: the slice name in the pitch's language, e.g. `Duplicate detection`
- Tags: `slice`, plus `uphill` or `downhill`, plus `nice-to-have` if it is one
- Description: one or two sentences on what the slice covers

## Question

- Type: the tracker's question item, child of the slice it is about, or of the bet if it is about the bet as a whole
- Scope fields: as the tracker adapter lists
- Title: `Q: What counts as a duplicate?`
- Tags: `question`
- Assignee: the shaper
- State: open while open; `Closed` once answered or once you adopt your default
- Description:

```
Raised in session 3.
Context: Rows can match on email but differ on name.
My default if unanswered: match on email only.
```

Always include your default. When the shaper answers, act on the answer, comment on the question with what you did,
and close it.

## Session comments (on the bet)

```
Session 4 of 10 started
Session 4 of 10 ended
Session 4 taken over
```

## Decision comment (on the item it affects)

```
DECISION (session 3): Duplicate rows keep the most recent entry.
Pitch doesn't specify. Most recent is the least surprising choice for users
re-importing updated data. Cheap to change later.
```

## Session summary (on the bet, before the `ended` comment)

What moved, what's next, open questions. A few lines. If you noticed work that doesn't belong in the bet, add it
last:

```
Outside this bet: write path re-reads setup.json on every count (slow, not needed for this outcome).
```

## Circuit breaker (on the bet)

```
CIRCUIT BREAKER: appetite of 6 sessions spent.
Merged into bet/1958: line crossing, capacity status.
Unfinished: live view (draft PR 812, uphill: rendering lag on three doors).
Learned: ...
```
