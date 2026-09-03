---
description: Ask the Seamlex team a question — answered from Jira and Confluence where possible, otherwise raised as a tracked question for a formal response.
---

# Ask Seamlex a question

Hand this to the **seamlex-delivery-liaison** agent in question mode, passing `$ARGUMENTS` as the
question.

The agent reads its settings from the plugin's fixed config, the **Configuration** section of
[`commands/hi-seamlex.md`](${CLAUDE_PLUGIN_ROOT}/commands/hi-seamlex.md) — nothing to generate and
nothing to check first. If the Atlassian connection is not up, run `/hi-seamlex setup`.

If `$ARGUMENTS` is empty, ask what they'd like to know.

The agent searches the board and the space first — many questions are already answered — and only files a
tracked question when it genuinely needs a response from the team. If what the customer is describing is
actually new functionality rather than a question, it hands off to **seamlex-product-owner** instead.
