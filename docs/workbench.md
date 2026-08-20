# The editorial workbench — a guide to the interface

> A living document, like its siblings, and provisional throughout. **Edit it as
> the interface moves** — a control that behaves differently from what is written
> here makes the document worse than nothing. Open it from the burger menu's
> **Documents** entry, where the Edit button writes it back.

> Disclaimer: This initial draft was written by Claude based on my briefing - Joro.

This describes the controls: what each one does, and what happens when you use
it. For the judgement behind the decisions — which candidates deserve which
verdict, and why — read **editing.md**, the editorial handbook, which this
document does not repeat. Open it from the same Documents chooser this document
came from.

**Read the overview; consult the rest.** Everything after it is reference
material you can arrive at with a question.

## The workflow

The pipeline harvests candidate dictionary entries from several sources and
scores them. **Every one of them is unreviewed until you say otherwise.** Your
job is to work through them and decide. The loop:

1. **Narrow.** Set filters until the ledger shows a population you can judge —
   typically `Review:unreviewed` plus something that makes the batch coherent.
2. **Judge.** Click a row. The detail pane shows the record and its evidence:
   where it came from, how confident the pipeline is, what the definitions
   corpus says, which related entries exist.
3. **Decide.** **Accept** (`A`) sanctions it and ships it; **Drop** (`X`) emits
   nothing; **Flag** (`F`) records that you looked and did not rule. Correct the
   fields first if they need it — an accept ships your edits, not the
   pipeline's guess.
4. **Commit.** From the burger menu, when you want the work published. Commit
   derives the dictionary from the patch store and commits both together.

Nothing is ever accepted for you. The pipeline proposes and prioritises; every
sanction in the store is one somebody pressed a key for.

### The five concepts everything else assumes

**Record.** One dictionary entry — one natural key, defined exactly in
**editing.md**. A row in the ledger.

**Group.** Records sharing a spelling and variation set, shown as one
expandable row, so records differing only by part of speech or accent sit
together. Filters and page counts are group-denominated.

**Patch.** What your decision writes. The pipeline's output is never edited in
place: your verdicts and field changes live in a separate store
(`data/patches/patches.jsonl`) and are laid over the pipeline's records. This
is why a decision can always be reversed, and why re-running the pipeline never
loses your work.

**Intrinsic versus derived.** You may edit the fields that state what the word
*is*; you may not edit what the pipeline *computed about it*. Confidence,
source and the classifier outcomes are recomputed on every build, so storing
one would freeze a value the pipeline owns. See *Which fields you can edit*.

**Affix.** A bound form — a prefix, suffix or clitic — which publishes to
`readlex-affixes.json` rather than the main dictionary. The code calls the
generalisation `bound_form`. Affix records carry two controls free words do not
(*regular* and *productive*), so the term recurs below.

### What the numbers on screen mean

The masthead counts are scoped to the **current filter**, not the whole corpus
— alongside the one count that is not: patches banked but not yet committed.
Beside them, a presence note names the other editors polling right now, and a
stale note offers to re-pull the list when another editor's verdict has changed
something in view. **The refresh is offered, never taken automatically.**

The footer's remaining-entries figure is meaningful only while the Review
filter is exactly `unreviewed`, and it counts groups, not records.

---

# Reference

## The ledger

### Groups and how they page

The `group_key` is the Latin word lowercased, the Shavian spelling, and the
variation set (the mergers plus the `variant` flag). **Identity only** —
editorial state never partitions a group, so an accepted record and an
unreviewed one sharing those three things are one group.

The daemon computes the partition and the client renders what it is sent, which
has two consequences you will see:

- **A group is served whole.** If any member matches your filter, every member
  arrives — including siblings that do not match. A group is never split across
  a page.
- **Paging counts groups.** The totals under the list, and the page buttons,
  are group-denominated.

A group of two or more renders a header row previewing the member that would
win the export — the one highest in the accent precedence RRP > RSSB > GenAm.
Clicking the header selects the whole group, and the group sorts by its
best-sorted member.

**Flat view**, in the burger menu, turns grouping off: every record becomes its
own row, and a filter returns exactly the records that match with no siblings
pulled in.

### The state glyph

Every column but the member count sorts. `pos` is the CLAWS C5 tag, and its
plain-English gloss is in its tooltip.

The `state` cell carries a glyph: `?` unreviewed, `✓` accepted, `✕` dropped,
`⚑` flagged, `…` a group whose members disagree. An orphaned patch keeps the
glyph of the verdict it recorded and turns **yellow**: the shape says what was
decided, the colour says it no longer resolves. Flags never orphan.

Two review states never appear as their own glyph. `edited` — an accept
carrying field changes — displays as `accepted`; `dirty` — an unaccepted edit —
displays as `unreviewed`. Both are decorations on a verdict rather than
verdicts, which is why a dirty row is invisible here and identifiable only by
the purple border in the detail pane and the `Data:edited` chip.

### Selection

`V`, Cmd/Ctrl-click, Shift-click for a range, Cmd/Ctrl+A for the working set.
With two or more rows selected the detail pane's buttons act on the whole
selection, and a verdict over ten or more rows asks for confirmation first.

### Why the list does not move under you

The filtered list is a materialised working set. A row you have just decided
stays where it is, showing its new content and stamp — it does not vanish
because it no longer matches. The list re-syncs only when the filter re-runs:
the `⟳` refresh button, or any change to the filters.

## Filters

Filters compose as **chips** in the bar. Four are pinned and cannot be removed
— the search box, Review, Data and Novelty. The rest are added from **+ Add
filter**. The `✕` button clears every filter including the search box.

**Within one facet, values are OR-ed. Across facets they are AND-ed.** So
`Data:manual` + `Review:unreviewed` means manual **and** unreviewed, while two
chips ticked inside Data means either.

Two facets — Source and Variations — are multi-valued and their pickers offer
an any/all toggle. **All** means the record's own set contains every value you
selected: multi-source agreement, or a record carrying every selected
variation.

**The facet lists below are current as of 2026-08-20.** They change; when a
chip you see is not described here, the chip is right and this document is
stale.

### Search

One box, matching **word OR Shavian OR IPA**. It is always a regular expression
and always case-insensitive. A pattern that fails to compile matches nothing
and is marked invalid rather than erroring. Type Shavian directly into it;
Cmd/Ctrl+K opens the Shavian keyboard.

### Review — the verdict lifecycle

`unreviewed`, `accepted`, `dropped`, `flagged`, and `mixed`.

A record is in exactly one of the first four. The two decorations fan out on
the wire: ticking `accepted` also matches records in the `edited` state, and
ticking `unreviewed` also matches `dirty` ones.

`mixed` is the one value that asks about a **group** rather than a record: it
selects groups whose members do not all share one verdict.

`manual` and `orphaned` are origins rather than verdicts, and live in Data.

### Data — origin and nature

Non-mutually-exclusive predicates; a record can satisfy several.

| Chip | Matches |
|---|---|
| `manual` | A record a human created — a patch with no anchor. |
| `orphaned` | An anchored patch whose basis record is gone. |
| `generated` | The RRP generator synthesized it — `source` *contains* `generated`, so a wiktionary+generated record counts. |
| `supplement` | Harvested from a supplement source: `source` contains an origin outside the core. A generated+wiktionary record is **both** generated and supplement. |
| `promoted` | The record carries `orig_var`: a pipeline transform rewrote its accent lane. The detail pane shows the same fact as a "was GenAm" pill. |
| `has definition` / `no definition` | Whether the definitions corpus covers the headword. |
| `inflection` | A tier recategorised the record's POS — either tier. |
| `own lemma` | The record is its own lemma. Computed by identity, not stored. |
| `affix` | The bound forms — affixes and clitics alike. See below. |
| `edited` | The record's patch overrides one or more fields. |

**`affix` is one chip**, asked of the same predicate the export uses
(`basis.is_bound_form`), so what it lists is exactly what ships to
`readlex-affixes.json`. Two different tests qualify a record:

- **An affix** carries the attachment hyphen on **both** its spellings —
  leading (`-ness`) or trailing (`anti-`). The POS tag says nothing here, and a
  record hyphenated on one spelling only is malformed data rather than an affix.
- **A clitic** is on the roster in `basis.BOUND_CLITICS`, because clitic status
  cannot be read off a spelling: the pool writes them bare (`'s`, `'ll`) and
  `'em` opens the same way while being a whole word.

No stored flag stands beside the hyphen: the respelling *is* the redirect.

⚠ **`edited` here is not the `edited` review state.** This chip means "carries
field overrides", and a dropped or flagged record with overrides qualifies. The
review state `edited` means "accepted, with changes", and is mutually exclusive
with dropped and flagged. A record can satisfy one without the other. The
collision is known and is being addressed separately.

The unticked complements — untiered records, linked lemmas, free words — carry
no chip of their own, because a facet that OR-s positives already says them by
leaving them out.

### Novelty — newness against upstream ReadLex

`new word`, `new spelling`, `new POS`, and `upstream`.

`upstream` is the baseline: a ReadLex-core row. The three `new-*` values
classify a supplement record against that baseline, and an upstream row never
matches them. Novelty is measured against upstream **only**, never against
sanctioned patches — so accepting a record never changes its novelty.

### The rest

**Source**, **POS**, **Var** and **Conf ≥/≤** filter on the values they name;
POS and Var are served by the daemon rather than hardcoded, so they list
exactly what the data holds. The four with a rule you could not guess:

- **Words** — `multi-word` or `single-word`, split on internal whitespace after
  trimming. Hyphens do not split a phrase.
- **Annotation** — a regex over the record's `info` entries: the quality tags,
  the `note:` free text, the `accent: unstated` signature. Deliberately
  separate from the headword search, so a note-carrying record is not a hit for
  an unrelated word.
- **Variations** — the union of the mergers list and the `variant` boolean, the
  same tag set the detail editor edits as toggles. `(none / canonical)` selects
  records carrying no variation at all.
- **Author** — the patch's recorded author. A record with no patch has no
  author and matches nothing.
- **Edited ≤ days back** — ⚠ the name understates it: this matches the last
  patch of **any** kind, a Flag included, not only patches that changed a
  field.

### Saved queries

The rightmost button in the filter bar saves the current filter set under a
name and recalls one from the list. Saved queries live in **this browser's**
local storage: they do not travel to another machine, are not shared with other
editors, and are not part of the patch store.

## The detail editor

The pane edits whatever the ledger has focused — one record, or a whole group.

### Which fields you can edit

Editable: **word, pos, shaw, var, ipa, freq, lemma**, the variation toggles
(mergers and `variant`), and the two conditional checkboxes *regular* and
*productive*. These are the **intrinsic** fields, listed in
`basis.INTRINSIC_FIELDS`, and they are the only keys a patch may carry.

Everything else is **derived** and read-only: `source`, `confidence`, `info`,
`orig_var`, the inflection tier, the classifier outcomes. The
pipeline recomputes them on every build. Storing one in a patch would freeze a
value the pipeline owns, so the editor does not offer them.

`freq` is intrinsic despite looking derived: the corpus stage runs *before* the
overlay, so nothing recomputes it afterwards and a patched frequency is the
last word. `lemma` is intrinsic for the same reason — upstream states it, but
the owner can override which lemma a record belongs to.

### The two conditional checkboxes

Both sit in the glance area after the POS, and both are **hidden** — not
disabled — where the question does not arise, so a control you cannot see is
one that could not apply.

**`regular`** appears only on the inflected C5 tags: `NN2`, `VVD`, `VVG`,
`VVN`, `VVZ`, `AJC`, `AJS`. Ticked means **both the Latin and the Shavian are
derivable from the lemma** — the form the affix table produces, and therefore
one the export prunes. Unticking stores `false` — a refutation, not silence.
The stored field is tri-state, and its third state, absent, is the one the
checkbox cannot express: hiding the control on every other tag is what leaves
it reachable.

**`productive`** appears only on affixes, tested by the attachment hyphen on
the Latin spelling. It is a **licence**: ticked means a consumer may apply this
affix to any stem of the right POS and the result is a word. Unticked is not a
refusal to answer — it says the record is a **witness**, held so the definition
can be shown, and nothing is generated from it.

⚠ **`productive` is yours alone.** No producer proposes a value, and the field
ships **absent on every record** until you tick one. Unticking deletes the
field rather than storing `false`: absence is what says "witness only", so the
two states the checkbox shows are the only two that exist. Unlike `regular` it is
published, because the licence is *for* the consumer: withholding it would
leave shaw-spell unable to tell an affix it may apply from one it may only
display.

In a mixed group both boxes start unticked and contribute nothing until you
move one, so an untouched group edit cannot rewrite each member's flags.

### The attachment hyphen homogenises

Editing **word**, **shaw** or **ipa** on an affix rewrites the attachment
hyphen on the other two to match, on blur — so the three spellings cannot drift
apart. The field you edited is the one that decides, and a form hyphenated on
both sides is malformed rather than doubly bound.

Storage stays symmetric this way, and a reader sees the fragment marker on
whichever field they happen to be looking at.

### Field markings

- **Purple border** — the field carries an explicit patch override: this value
  is the owner's, not upstream's. It is joined by a small `edited` tag beside
  the label. The marking never appears on an unoverridden field, even a focused
  one, so a cursor alone can never read as "patched".
- **Green** — unsaved typing in this session. Live editing wins over the purple
  tint in the cascade, so a field you are changing right now looks changed, not
  merely overridden.

### Editing a group

Select several records and the fields speak for all of them:

- A field every member agrees on renders that value normally.
- A field the members disagree on renders **blank** with a `multiple`
  placeholder, and the distinct values listed beside it. It contributes nothing
  to the verdict unless you type into it — so a group verdict cannot silently
  overwrite one member's value with another's.
- The purple override marking uses the **intersection** of the members'
  override sets, not the union: marking it otherwise would let one member's
  edit vouch for values the others never touched.

The note field follows the same rule. A group whose members' Latin words differ
(case-insensitively) gets the editor alone, without the evidence panes.

### The evidence panes

Below the editor, two collapsible panes whose open state persists across
selections and reloads.

**Related entries** gathers every record sharing the Latin word
(case-insensitively) **or** the Shavian spelling **or** the lemma's word and
POS. Each union brings in something the others miss: the shared spelling brings
variant siblings (*estrogen* / *oestrogen*, same Shavian, different word), the
lemma brings the inflections (*abacus* / *abaci*), which share neither. Each
row opens in a modal where it can be decided without leaving the record you are
on.

**Definitions** is a read-only view of the Shavian definitions corpus for the
headword. Of the reference links beside the fields (Wiktionary, Cambridge,
Merriam-Webster, OED, Wordnet), the last is the gloss source itself, so a
definition can be checked against what it was derived from. A missing transliteration or an empty sense list is a coverage gap, and
the pane says so rather than hiding it.

The `✎` button beside a sense corrects **that sense's** transliteration; the
gloss and the synset are stable identity and cannot be edited. The correction
is written to `data/patches/definition-patches.jsonl` — a **separate** store,
never `patches.jsonl`. Definition corrections have no accept, flag or drop: a
correction is an edit, not a sanction.

### The note

A full-width fold below the fields. The notes already in the store are machine
provenance written by the pipeline (`shave_says`, `ml_disagrees`, `r_gap`), not
prose; they are shown as they stand, and anything you type replaces the note on
the patch your next verdict writes.

## Verdicts

**editing.md** says what each one means editorially; this is what each does.

| Button | Key | Writes | Effect |
|---|---|---|---|
| **Accept** | `A` | `op:accept` | Sanctions the record with your edits laid over it. Reviewed, and it ships. |
| **Drop** | `X` | `op:drop` | The record emits nothing. The row stays visible so the decision can be seen and rolled back. |
| **Flag** | `F` | `op:flag` | Looked at, no verdict. A no-op for production output. |
| **Clear** | `C` | deletes the patch | Reverts to the untouched source. No-ops on an unreviewed row. |

### Why there is no Edit button

`E` focuses the Shavian field; an edit you make and then navigate away from is
auto-saved as a **dirty** patch — persisted `op:edit`, recorded but neither
reviewed nor shipped. Only an explicit Accept ships it. **This is the mechanism
behind "never auto-accept": leaving a row can persist your work, but it can
never sanction it.**

"Navigating away" means moving the detail pane to another record — clicking a
row, stepping, or changing the selection. The save targets the record you are
leaving, so it cannot race the one you land on. **Closing the tab is not a
navigation and does not save.** Cmd/Ctrl+Enter saves without moving.

Auto-save is skipped for a group edit — a divergent group has no single record
to save, and its boxes read blank — so group edits commit only through an
explicit verdict.

### The two guards on Accept

- Shavian cannot be empty, and frequency must be a whole number. Either failure
  raises before anything is written.
- The editor refuses an Accept when another **accepted, canonical** record
  already claims the same word, POS and accent with a different spelling. One
  canonical spelling per slot; genuine homographs are separated by their lemma.
  A record carrying a merger or the `variant` flag is not canonical and is
  exempt — which is why flagging one of a colliding pair as a variant resolves
  the conflict.

### Undoing

**Undo** (`U`) reverses the last decision. Clearing a row whose patch carries
field edits warns first, **naming which word and which fields** would be lost —
because "you will lose some edits" is not something you can weigh.

Dropping a **manual** record deletes its patch outright: it has no basis to
revert to, so a drop and a clear are the same act.

### Creating and cloning

The `+` button opens a blank create form; **Clone** opens it prepopulated from
the focused record — the path for adding a dialect sibling.

Both write a manual patch with no anchor. A new manual record is created
**dirty** — unreviewed, shipping nothing — and takes a verdict like any other
row. Re-deciding it edits that manual patch in place rather than writing an
anchored one, which would orphan the decision.

The form checks distinctness as you type, disabling the verdict buttons on a
collision. It looks only at **live** siblings: a dropped record has vacated its
slot, so authoring the same anchor again is legitimate.

## Keyboard

`?` opens the full list in the app. Beyond the verdict keys above: `J`/`K` or
`↑`/`↓` step, `→`/`+` expand the focused group and `←`/`-` collapse it, `U`
undoes. **Inside a field** Enter accepts, Shift+Enter drops, Cmd/Ctrl+Enter
saves without moving, and Esc leaves edit mode or closes a dialog.

## The burger menu

**Commit** (below), **Flat view** (toggles grouping off), **Messages**,
**Documents**, **Keyboard shortcuts**, **Shavian keyboard**, **Sign out**, and:

**Revert uncommitted changes** — discards every editorial decision since the
last commit, by checking out the committed patch store. It confirms first,
naming the count. **It cannot be undone.**

### Messages

Editors' notes to each other, newest first, in their own store
(`data/messages.jsonl`). They are not patches and are never published. A
message is an atomic append — nothing replaces one. Text is posted and rendered
as plain text, never as markup, because another editor wrote it.

The board opens on every page load.

### Documents

One entry opening a chooser of filenames — the `.md` files in `data/docs/`,
each a plain markdown file with no serialisation around it. A document opens
rendered; the Edit button swaps in a textarea. **This document is one of them,
and this is where you edit it.**

Saving is guarded by a revision hash of the file's bytes, not an mtime. If
someone saved first, **nothing is written**: you are told who and when, and
your text stays in the textarea. It warns; it never locks.

Documents are rendered inside this dialog rather than served as pages, so a
markdown link from one document to another cannot resolve. Name a sibling
document and let the reader pick it from the chooser.

## Commit

Commit publishes. It is in the burger menu, and the trigger carries an
`uncommitted` marker so pending work stays visible while the button is tucked
away.

The dialog offers a checkbox per category — **Patches**, **Messages**,
**Documents**. They exist to *narrow* a commit; the common case is committing
everything pending. Only the ticked categories' stores are staged; the rest of
the working tree is never swept in. The subject line is generated and not
editable; the body is yours.

⚠ **The dictionary publish rides Patches alone.** Ticking Patches derives
`readlex.json` and `readlex-affixes.json` from the on-disk patch store and
commits them alongside it, so the published dictionary can never disagree with
the decisions that produced it. A messages-only commit derives nothing.

The derivation runs **before** git touches anything: if it fails the whole
commit aborts with nothing staged, so the patch store is never committed ahead
of the artifacts it ships with. Publishing also requires the frequency corpus —
the editor runs without it, but a published `readlex.json` must match
production, so its absence fails the commit rather than shipping a
frequency-less file.

Orphaned patches do not block a publish: they are skipped, retained in the
store, and summarised in the log.

**Commit is a local git commit.** Pushing it anywhere is a separate, human act.

---

*Sibling documents, both in the Documents chooser: **editing.md** — the
editorial handbook; **dictionary.md** — what the dictionary is for.*
