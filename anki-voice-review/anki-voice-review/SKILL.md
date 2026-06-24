---
name: anki-voice-review
description: "Review Anki flashcards across text and voice modes using a 3-phase workflow:   fetch cards in text mode, review in voice mode with automatic rating, then   batch-apply ratings back to Anki in text mode.   ALWAYS trigger whenever the user says \"I want to review my [X] deck\",   \"review my [X] deck\", \"review my [X] cards\", or \"review my [X] flashcards\",   for ANY value of [X] — e.g. \"system design deck\", \"Spanish cards\",   \"Haitian Creole flashcards\" — even if the word \"Anki\" never appears. The   words \"deck\", \"cards\", and \"flashcards\" alone, in the context of   studying/review, always refer to Anki (never slides). Also trigger for:   \"start an Anki review session\", \"review my Anki cards\", \"Anki voice   review\", \"let's review [X]\" where X is a known study topic. Also trigger   for phonetic variants: \"monkey\", \"anky\", \"hanky\", \"honky\", \"inky\", \"enky\" — these likely mean \"Anki\"."
---

# Anki Voice Review Skill

## Overview

This skill bridges the gap between Anki MCP (only available in text mode) and
voice mode review sessions. It caches card data upfront, runs the review
conversationally in voice mode, then applies all ratings in bulk at the end.

---

## Phase 1: Text Mode Setup

**Trigger:** User requests a review session.

**Steps:**

1. Identify the deck name. If the user is in voice mode or the deck name sounds
   like a phonetic mishearing ("monkey", "anky", "hanky", "inky", "ankee"),
   confirm: "Did you mean Anki — and which deck?"

2. Call `list_decks` first. Do not assume the user's spoken/typed deck name
   matches exactly — match it against the real deck list (e.g. "system
   design" → `0-System Design`). If no close match exists, show the user the
   deck list and ask which one they meant.
If this call fails or times out, see Troubleshooting: MCP Tool Calls Failing below before proceeding.

3. **Get the full set of due card IDs — don't rely on `get_due_cards` alone.**
   `get_due_cards` only returns ONE card per call and won't serve the next
   until the current one is rated, so it cannot be used to pre-load a full
   session in text mode. Instead, use `find_notes` to get all due/new note
   IDs up front:
   - Query: `deck:"<deck name>" (is:due or is:new)`
   - **Always wrap the deck name in double quotes inside the query string.**
     Deck names with spaces or special characters (e.g. `0-System Design`)
     will silently return 0 results without quotes — this is a common
     failure mode, not a sign the deck is empty. If a query with quotes
     still returns 0, then trust that the deck genuinely has nothing due.
   - Note: `is:due` alone excludes new cards. Use `(is:due or is:new)` to
     capture the full due queue (new + learning + review).

4. Call `notes_info` with the note IDs from step 3 to pull Front/Back fields
   for every card in one batch call.

5. **If zero cards are returned:** tell the user the deck has nothing due
   right now and stop. Do not proceed to Phase 2.

6. **Check for blank `Back` fields.** For any card where `Back` is empty,
   flag the count without revealing front text. Format as:

   **⚠️ 3 cards have no answer filled in. ⚠️**

   Then ask: "fill them in now, skip them for this session, or decide
   per-card?" If the user asks which specific cards, reveal the front text
   then — but not before asking. Let the user choose per card. Do not
   assume; do not batch-skip without asking. **This rule takes priority
   over showing card content — the no-preview rule in step 8 applies here
   too, not just after this check.**

7. Build an internal tracking log from the `notes_info` results (excluding
   any cards the user chose to skip). For each remaining card, store:
   - `cardId` (from MCP response — do not invent IDs)
   - `front` (the question)
   - `back` (the correct answer)
   - `userAnswer` (to be filled during review)
   - `rating` (to be filled during review, scale 1–4)

   **This tracking log must be fully built and held in context BEFORE
   switching to voice mode.** Voice mode cannot call Anki MCP tools, so all
   card content needs to already be loaded into memory at this point — not
   fetched on demand during the review loop.

8. Confirm to the user with ONLY a card count and, if any were skipped, a
   skipped count. **Do not print or preview card questions/fronts, topics,
   or subject matter at this stage in any form** — no topic-level summary,
   no category labels, nothing that hints at content. The only acceptable
   output here is a bare number. Cards are revealed one at a time during
   the actual review in Phase 2, not before.

   Format exactly as:
   "X cards ready for review."

9. End Phase 1 with a clear, obvious call-to-action telling the user to
   switch to voice mode now. This line MUST be printed on its own line,
   verbatim, including the emoji — do not paraphrase or drop it:
   "➡️ Switch to voice mode now to begin!"

---

## Troubleshooting: MCP Tool Calls Failing

If any Anki MCP tool call (`list_decks`, `find_notes`, `notes_info`,
`present_card`, `rate_card`, etc.) fails, times out, or returns a
connection error, retry exactly once. After that, do not retry blindly. Tell the user the call failed and walk them through this checklist in order, since each step depends on the one before it:

1. **Is the Anki Desktop app open and running?** The MCP server (whether
   the Anki MCP Add-On or a standalone MCP server) cannot reach
   the collection unless Anki itself is running.
2. **Is the Anki MCP server itself running?** This is a separate process
   from Anki Desktop — either the Anki MCP Add-On listening inside Anki, or a standalone MCP server process. Confirm it started without errors.
3. **Is the computer Anki is running on powered on and connected to the
   internet?** Covers the case where the whole machine is asleep,
   shut down, or offline.
4. **If reviewing remotely: is the cloud tunnel connected?** This applies
   to the AnkiMCP cloud tunnel service, ngrok, or any other custom tunnel.
   Tunnels can silently disconnect or expire — check the tunnel's own
   status/logs, not just Anki.

Ask these as a single checklist rather than guessing which layer failed —
the failure mode (timeout vs. explicit connection refused vs. auth error)
doesn't reliably indicate which layer is down.
Mention that they can seek addition troubleshooting information on the AnkiMCP site: https://ankimcp.ai/docs/installation/web/#troubleshooting

---

## Phase 2: Voice Mode Review Loop

**Trigger:** User switches to voice mode after setup.

**Card presentation flow (repeat for each card):**

1. Read the question (front of card) aloud. No preamble.
2. Wait for the user's spoken answer.
3. Internally compare their answer to the back of the card.
4. Give brief feedback (one sentence max).
5. State the rating verbally using the word (not the number), then
   immediately present the next question.

   | Rating | Word to say |
   |--------|-------------|
   | 1 | "Again" |
   | 2 | "Hard" |
   | 3 | "Good" |
   | 4 | "Easy" |

**Do not:**
- Say the numeric rating (1–4) — say the word instead
- Ask for confirmation before rating
- Add filler phrases like "Great job!" or "Let's move on to the next one"
- Repeat the correct answer in full unless the user asks

**If the user disagrees with a rating:**
They will say something like "that should be a 4" or "I think I got that right."
Update the rating in the tracking log without comment and continue.

**Session end:**
When the user says "done", "end session", "that's it", or "apply ratings",
end the response with the following line printed verbatim — do not
paraphrase, shorten, or drop it. Since this line is spoken aloud in voice
mode, it must be plain text with no emoji or symbols:
"Your ratings aren't saved yet. Switch back to text mode now and tell me to
apply and sync your card ratings!"

Make clear (in your own words, before this line) that ratings are not yet
saved to Anki — they only exist in this session's tracking log until
applied in Phase 3.

---

## Phase 3: Text Mode Batch Rating

**Trigger:** User switches back to text mode and confirms they're done.

**Steps:**

1. Display the session summary — a table of all cards reviewed with ratings:

   | Card ID | Question | Rating |
   |---------|----------|--------|
   | ...     | ...      | ...    |

2. For each card in the log, in order:
   - Call `present_card` with that `cardId` first. This loads the card to
     the top of Anki's review queue — `rate_card` will fail with "not at
     top of queue" if this step is skipped, even though the card was
     already reviewed conversationally in voice mode.
   - Then call `rate_card` with the same `cardId` and the stored `rating`.
   - If `rate_card` still errors with "not at top of queue", call
     `present_card` again for that same card and retry once before moving on.

3. After all cards are processed, confirm to the user how many were
   successfully synced, and call out by Card ID any that failed after the
   retry so they aren't silently dropped.

4. Cards skipped in Phase 1 (blank `Back` fields) and any cards left
   unreviewed because the session ended early remain due in Anki — no
   action needed, they'll surface again next time `find_notes` runs for
   that deck.