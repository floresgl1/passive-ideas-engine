# Quiz bot — setup

The asynchronous quiz (`prompts/quiz-probe.md` + `prompts/quiz-collect.md`)
needs three things this repo cannot create for itself: a Discord bot that can
*read* messages, the channel it reads, and the user whose replies count.

Discord webhooks are write-only. `GET` on a webhook URL returns the webhook's
own metadata and never channel history, which is why posting works today with
no bot at all and reading requires one.

Posting still goes through the existing webhook — with `?wait=true` so Discord
returns the created message and the routine can record its ID. The bot token is
needed **only for reading replies.**

## 1. Create the bot

1. https://discord.com/developers/applications → **New Application**.
2. **Bot** → **Add Bot** → **Reset Token**, copy it. This is the only time it is
   shown.
3. Same page, **Privileged Gateway Intents** → enable **MESSAGE CONTENT
   INTENT**. Without it every `content` field comes back as an empty string,
   the poller sees blank replies, and — because a blank is not a pass — it will
   route every answer to the MC probe. Silent and very confusing.
4. **OAuth2 → URL Generator**: scope `bot`, permissions **View Channel** +
   **Read Message History**. Nothing else; the bot never needs to post.
5. Open the generated URL, invite it to the guild, and make sure it can see the
   quiz channel.

## 2. Values already known

Read off the live webhooks — no lookup needed:

| Variable | Value |
|---|---|
| `QUIZ_CHANNEL_ID` | `1533995899980611674` |
| (guild) | `1533175431476543568` |

`QUIZ_WEBHOOK_URL` is already in the trigger environment and stays as-is.

## 3. Values you must supply

| Variable | How to get it |
|---|---|
| `DISCORD_BOT_TOKEN` | Step 1.2 above |
| `QUIZ_USER_ID` | Discord → Settings → Advanced → **Developer Mode** on, then right-click your name → **Copy User ID** |

`QUIZ_USER_ID` is what stops the poller from grading someone else's message —
or its own probe — as your answer.

## 4. Triggers

Two, both fresh-session:

| Trigger | Payload | Cron | Why |
|---|---|---|---|
| Quiz - Probe | `prompts/quiz-probe.md` | `0 15 * * 1,3,5` | Three probes a week, an hour after the daily ideas run. Refuses to post if one is already outstanding, so a miss costs nothing. |
| Quiz - Collect | `prompts/quiz-collect.md` | `0 */2 * * *` | Polls for a reply. Exits silently when `stage` is `"idle"`, which is most runs. |

Both need `QUIZ_WEBHOOK_URL`, `DISCORD_BOT_TOKEN`, `QUIZ_CHANNEL_ID`,
`QUIZ_USER_ID` in their environment.

Cadence is deliberately below the ~4 ideas/day intake rate. It is not meant to
drain the backlog — nothing at one-idea-per-sitting can — it is meant to make
showing up not require remembering.

## 5. Verify before trusting it

```bash
# bot can read the channel — expect HTTP 200 and a JSON array
curl -sS -o /dev/null -w '%{http_code}\n' \
  -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
  "https://discord.com/api/v10/channels/$QUIZ_CHANNEL_ID/messages?limit=1"

# message content is actually populated — if this prints "" the intent is off
curl -sS -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
  "https://discord.com/api/v10/channels/$QUIZ_CHANNEL_ID/messages?limit=1" \
  | python3 -c 'import json,sys; print(repr(json.load(sys.stdin)[0]["content"]))'
```

`401` = bad token. `403` = bot not invited, or missing View Channel / Read
Message History. `200` with empty `content` = Message Content Intent off.

## 6. State

`quiz-state.json` at the repo root is the whole state machine:

```json
{
  "stage": "idle | awaiting_free_response | awaiting_mc",
  "idea_file": "ideas/YYYY-MM-DD.md",
  "idea_heading": "### N. Name — [Category]",
  "probe_text": "...",
  "probe_message_id": "...",
  "mc_message_id": null,
  "posted_at": "UTC ISO-8601",
  "expires_at": "UTC ISO-8601"
}
```

It is committed to `main` because the two phases are different sessions on
different machines and share nothing else. `idea_heading` must be the exact
`###` line: every idea in a file has a byte-identical `- competence: unlabeled`
beneath it, so an approximate anchor labels the wrong idea silently.

An expired probe resets `stage` to `"idle"` and leaves the idea `unlabeled`.
Silence is never scored — it measures attendance, not knowledge.
