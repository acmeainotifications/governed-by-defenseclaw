---
title: "Step 7 — Stage 5: Meeting summaries & action items"
---

# Step 7 — Stage 5: Meeting summaries & action items

Pull a meeting's transcript and get a summary plus a clean split of YOUR action items versus the team's.

!!! warning "Transcript, not video, and the AI-summary caveat"
    `webex.list_transcripts` downloads the plain-text transcript the model can read. The Webex AI-generated summary (`meeting:summaries_read`) needs the **Webex AI Assistant** license; without it, summarize the transcript text yourself with the local model. No transcript (transcription was off) → nothing to summarize.

## 1. Summarization prompts

=== "Standard, summary + split actions"

    ```
    "Download the transcript of <meeting-title> and summarize it.
     Then list action items in two groups: mine, and everyone else's."

    # → webex.list_transcripts → download text → local model summarizes
    ```

    ??? note "Expected output"
        ```
        Summary
          Platform sync, 38 min. Reviewed Q3 capacity numbers, agreed to
          start the deck Friday, postponed the vLLM upgrade until next sprint
          pending Jamie's sign-off, oncall handoff settled for next week.

        Sources
          GET /v1/meetingTranscripts/<id> · 38 min, 4 speakers

        Action items
          • Mine: write the Q3 capacity deck draft by Friday EOD
                  sign off on vLLM upgrade plan once Jamie posts it
          • Others': Mo — collect H1 capacity numbers (today)
                     Jamie — post vLLM upgrade plan (Wed)
                     Anna — confirm oncall swap with Dana

        Decisions made
          • Postpone vLLM upgrade by one sprint
          • Start capacity deck Friday

        Open questions
          • Do we need extra GPUs for Q3 surge?

        Suggested next action
          Want me to draft your reply to Mo with the deck outline?
        ```

=== "TL;DR, 5 bullets"

    ```
    "TL;DR of <meeting-title> in 5 bullets. No fluff."
    ```

    ??? note "Expected output"
        ```
        • Q3 capacity numbers in — total looks fine, surge is borderline
        • vLLM upgrade postponed 1 sprint (need Jamie's sign-off)
        • Deck starts Friday — owner: me
        • Oncall handoff next week (Dana → Anna)
        • Open: extra GPUs for Q3 surge?
        ```

=== "Decisions + open questions only"

    ```
    "From <meeting-title>, give: decisions made, open questions, blockers.
     Drop everything else."
    ```

    ??? note "Expected output"
        ```
        Decisions
          1. Postpone vLLM upgrade 1 sprint
          2. Capacity deck owner: me, deadline Friday EOD
          3. Oncall handoff: Dana → Anna next week

        Open
          1. Extra GPUs for Q3 surge — needs cost approval
          2. Customer Acme escalation — no decision, taken to async

        Blockers
          None hit during the meeting.
        ```

## 2. Sanity-check the split (yours vs theirs)

That's where it slips most often early on.

[Continue to Step 8. Stage 6: Drafts →](phase-8.md){ .md-button .md-button--primary }
