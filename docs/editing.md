# Shaw-Spell editor's handbook

> A living document, like its sibling, and provisional throughout. Edit it as
> the conventions move.

> Disclaimer: This initial draft was written by Claude based on my briefing - Joro.

This is the working guide for editors: the workflows, the records, how patches
work, and the judgement calls the workbench puts in front of you. For what the
dictionary is trying to be — the goals, the accent model, what ships and what
does not — read the sibling document, [dictionary.md](dictionary.md), first.
This document does not repeat it.

## Where the candidates come from

Almost everything you review was produced by an automated pipeline. It combines
the ReadLex core with supplements harvested from WordNet, Wiktionary, a curated
names list and a generated lane; deduplicates; tags merger and variant
spellings; reclassifies what the rules allow into RRP; collapses records that
say nothing their parent accent does not; and scores each survivor's
confidence. Every record carries its provenance — `source`, `confidence`, and
an `info` list of annotations the sources supplied (quality tags like
`obsolete` or `dialectal`, free-text source notes behind a `note:` prefix,
`accent: unstated` where no source named an accent). Those annotations are
evidence for your judgement, deliberately carried through to review instead of
being filtered out upstream; they decide nothing on their own.

The one thing to internalise about the pipeline: it is systematic, so its
errors are systematic too.

## Fix it upstream, not record by record

**If you find yourself making the same correction twice, stop.** A repeated
wrong pattern is almost never two coincidences — it is one bug in an importer
or classifier, and it has produced every instance at once.

A real case: 270 records — *orange*, *florida*, *boris*, *dorothy*,
*abhorrence*, *authoritarian* and friends — sat misfiled as unclassified
variants. General American *orange* 𐑸𐑩𐑯𐑡 beside canonical 𐑪𐑮𐑩𐑯𐑡 differs by
exactly the lot–palm merger, but before /r/ the two accents spell the vowel
with a different number of letters, and the merger matcher only compared
letter by letter — so nothing matched and the whole class fell through. A
small change to the correspondence table in `src/tools/dialect_mergers.py`
named all 270 on the next build. Hand-editing them would have taken days,
and would have fixed nothing for the next import: the same bug would have
minted the same misfiled records again.

So when a pattern repeats: flag it rather than fixing it by hand, with two or
three concrete examples. Your individual decisions are not wasted by upstream
fixes — a patch anchors to a record's identity and rides along when the
pipeline recomputes everything beneath it — but two hundred hand-edits that
one code change could replace are.

## What a record is

A record's identity is the five-part natural key: **word, part of speech,
Shavian spelling, accent, lemma**. Consequences you will feel daily:

- **The Shavian spelling is the payload.** A different spelling is a different
  record, never an edit of the same one.
- **Each accent lane is reviewed independently.** Fixing the RRP record does
  not touch the GenAm record.
- **The lemma is the sense group**: it gathers one sense of a word with all
  its spelling variants and inflections, and two records can share a wordform
  yet belong to different lemmas (*axes* under *axe* and under *axis*).
  Because the lemma is part of identity, editing it re-anchors the record's
  patches — do it deliberately.

The fields you can change are the intrinsic ones: `word`, `shaw`, `pos`,
`ipa`, `var`, `mergers`, `variant`, and `freq`. Everything else — `source`,
`confidence`, `info`, the classifier outcomes — is derived provenance: the
pipeline recomputes it on every build, and the editor shows it read-only.

## How patches work

Every decision you make in the workbench is a **patch** in
`data/patches/patches.jsonl`: the record's anchor, your verdict, and only the
fields you changed — a minimal diff over the live basis. Your edit wins over
whatever the pipeline recomputes; deleting a patch is a clean rollback. The
store holds current state, not history: re-deciding a record replaces its
earlier patch.

Two rules are absolute:

- **Nothing self-promotes.** Every pipeline product is a candidate until a
  human editor accepts it. No confidence tier, batch operation or tool accepts
  on your behalf — a high score only prioritises review. The one exception is
  the ReadLex core, which arrives already accepted (except where two upstream
  records contradict each other on one slot; those wait for adjudication).
- **The patch store is written only through the editor.** No script, tool or
  agent writes `patches.jsonl` directly. It is the one artefact that cannot
  be regenerated: it *is* the editorial work.

Your verdicts:

- **Accept** — sanction the record, as-is or with your edits laid over it.
  Accepted records ship.
- **Edit** — a work-in-progress edit, persisted when you navigate away. It is
  visible but ships nothing until an explicit Accept.
- **Drop** — the record emits nothing. It stays visible in the editor so the
  decision can be seen and rolled back.
- **Flag** — looked at, no verdict yet.

These surface in the ledger as the review states: `unreviewed`, `accepted`,
`edited` (accept with edits), `dirty` (an Edit awaiting Accept), `dropped`,
`flagged`, and `orphaned` — a decision whose record upstream has drifted out
from under, kept visible for re-anchoring rather than silently lost. A record
you create from nothing is a *manual* record: manual is an origin, not a
verdict, and it awaits acceptance like anything else.

One guard to know: the editor refuses an Accept when another accepted record
already claims the same word, part of speech and accent with a different
spelling — one canonical spelling per slot. Genuine homographs are separated
by their lemma.

## Making the calls

- **Which goal does the entry serve?** Every entry is either a canonical
  spelling ReadLex would produce, or a genuine mainstream variant worth
  capturing alongside one. If it is neither — a transcription artifact, a
  sub-national form, a duplicate — drop it.
- **Never guess a stress pattern.** The spelling rules need the source IPA's
  stress marks; a polysyllabic candidate without them cannot be spelled. Flag
  it.
- **Never ship contamination.** Nothing outside the Shavian block (plus
  space, hyphen, apostrophe and the naming dot) may appear in a spelling. A
  stray IPA or Latin character makes the record malformed under every accent —
  and is usually an upstream bug; see above.
- **ReadLex is authoritative for what it covers.** Supplements extend it;
  they do not overrule it. Overriding an upstream ReadLex spelling is a
  deliberate, knowing patch — never a side effect of accepting a candidate.
- **Merger flag, variant flag, or nothing?** An exact vowel swap a known
  merger names, on an accent that has that merger, takes the merger flag. A
  divergence no merger names, beside an attested canonical sibling, takes
  `variant`. A difference that *is* the base-accent difference takes no flag
  at all — the accent label already carries it. And always check the IPA
  before believing a merger: two spellings over identical IPA are a spelling
  question, not a sound change.
- **Respect meaningful absence.** No GenAm record means RRP covers it; no
  regular inflection means consumers generate it. Filling either "gap" makes
  the data worse.

Every editor accepts on their own authority: an Accept ships the record, it is
not a proposal awaiting someone else. Every patch records who made it, and the
editor is shown on the record and can be filtered on, so anyone's work can be
read back — including your own.

## Affixes, clitics, and inflections

The main dictionary holds only free-standing written words; the test is
orthographic. *could've* is one written word and stays. *anti-*, *-ness* and
*'ll* are not words: they publish to `readlex-affixes.json`, same schema, with
a hyphen marking the attachment side (𐑨𐑯𐑑𐑦-, -𐑯𐑩𐑕, -'𐑤). Affixes take the
`PRE`/`SUF` tags; a clitic keeps its apostrophe and its real syntactic tag.
Clitic membership cannot be read off the spelling — *'em*, *'tween*, *'twas*
open with the same apostrophe but are reduced whole words and stay in the main
dictionary — so moving a form to the bound-forms roster is an explicit
editorial act.

Only irregular inflections ship (*sheep* as its own plural), and an
irregular's presence is itself the signal not to generate the regular form.
Regular inflections still appear in the workbench, attached to their lemma,
and are pruned at export. Reviewing them matters anyway: a wrongly generated
regular form is exactly the error class this arrangement exists to make
visible.

## Publishing

The Commit action publishes `readlex.json` and `readlex-affixes.json`
together, in one commit alongside the patch store, so the published dictionary
can never disagree with the decisions that produced it. The editor is the sole
publisher of both files, and Commit records the publication locally — pushing
it anywhere is a separate, human act.

---

## Open questions for review (not part of the document)


1. **What is the channel for upstream reports?** "Flag it rather than fixing
   it by hand" needs a destination — the message board, the backlog, or
   something else. The record does not say; the section currently says "flag
   it" without naming where.

2. **The mistagged bare clitics** (`'s`, `'m`, `'ll` carrying implausible
   tags) are recorded as outstanding retagging work. Should editors fix them
   on sight, or leave them for a ruling? Editors will meet them.
