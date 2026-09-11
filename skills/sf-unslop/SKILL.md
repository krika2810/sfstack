---
name: sf-unslop
description: "Cut AI tells from any prose surface: replies, docs, PR bodies, commit messages. Plain declarative sentences, no filler, no decorative phrasing, every claim carrying its evidence or its label. Use for /sf-unslop or before shipping any written surface."
---

# sf-unslop

sf-unslop cleans prose. It applies to everything a person will read: chat replies, PR bodies, commit messages, docs, comments. Write it clean the first time; a cleanup pass after drafting does not catch what the drafting habit keeps producing.

## When to use it

- Before shipping any prose surface: a reply, a doc, a PR body, a commit message.
- Any time a draft reads like it was generated instead of written.

## The rules

1. **Short declarative sentences.** One thought per sentence, ended with a period. "The deploy validated clean and the suite is green." Not "After running the validation process, which completed successfully, the test suite also passed."
2. **No filler openers or closers.** Delete: "I'd be happy to", "Great question", "Certainly", "I hope this helps", "Let me know if you need anything else", "In conclusion". The first sentence carries content or the message starts one sentence later.
3. **No decorative phrasing.** No "delve", "leverage", "streamline", "seamless", "robust", "elevate", "navigate the landscape", "in today's fast-paced world". If a word is doing no work, it goes.
4. **No long-dash interruptions.** Use a regular hyphen, a comma, or rewrite the sentence.
5. **No false symmetry.** "Not just X, but also Y" and "It's not about X, it's about Y" are templates, not thoughts. Say the thing.
6. **No announcement of the writing itself.** Delete "Here is what I found:", "Below are the results:", "A few questions:". Start with the finding.
7. **Every claim carries evidence or a label.** "Tests pass" is a claim; "test run 707xx0000000789: 42 passed" is evidence; "this should be faster" is a guess, so label it one. Never hand the reader a check you could have run.
8. **Name real things.** Real IDs, real file paths, real org aliases, real numbers. Vague references ("the recent deploy") force the reader to do your work.
9. **Vocabulary stays exact.** Salesforce terms used precisely per **sf-technical-writing**. Do not translate "record-triggered flow" into "workflow automation thing".

## The workflow

1. Draft with the rules on, not off.
2. Before sending, re-read once with one question: which sentence would embarrass me if quoted back? Fix or delete it.
3. Ship. No preamble about having shipped.

## Gotchas and failure modes

- **Over-tightening into fragments.** Terse is not broken. Full natural sentences, just short ones. "Tests green. Deploy shipped." is fine as a telegram; a runbook needs full sentences.
- **Cutting content with the filler.** The details, tradeoffs, and open decisions all stay. Unslopping removes decoration, never substance.
- **Tone-deafness in messages to people.** Replies on the user's behalf follow the relationship's register; the rules still apply, warmth included.

## Proof it worked

Read the final surface aloud. Every sentence is one thought, every claim has evidence or a label, and no phrase exists that a tired reader would skip.

## Reply

The cleaned surface only. No commentary about the cleaning.
