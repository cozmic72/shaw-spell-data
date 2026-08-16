# Shaw-Spell — spelling principles and editorial policy

> **This is a living document, and provisional throughout.** It describes the
> project's current thinking, not a finished standard; the dictionary and its
> conventions are still evolving, and this text evolves with them.
> [Editors: you can — and should — edit this document as the conventions
> themselves move.]

> Disclaimer: This initial draft was written by Claude based on my briefing - Joro.

## What Shaw-Spell is

Shaw-Spell extends and modernises the [ReadLex](https://readlex.pythonanywhere.com)
(Kingsley Read Lexicon), the Shavian pronunciation dictionary. ReadLex is our
core: its entries arrive intact, and where it covers a word we treat it as
authoritative. On that foundation we do two things:

- **expand the vocabulary**, spelling new words as ReadLex itself would spell
  them; and
- **capture the major mainstream variations in pronunciation** — the spoken
  language as it is evolving — as entries in their own right, alongside the
  canonical ones.

The second goal is where we go beyond ReadLex's remit. ReadLex describes
itself as neither prescriptive nor descriptive but *selective*: one standard
spelling per word, for stability. We keep that selective standard as our
canonical layer, and add a more descriptive layer on top of it, recording the
principal ways pronunciation genuinely differs across the mainstream accents.
As ever with Shavian: nobody is told how to spell. This is a reference, not a
rulebook.

## Compatibility with ReadLex

The exported dictionary, `readlex.json`, is **mostly backwards compatible**
with ReadLex — a consumer of ReadLex can read it the same way. The
differences:

- **More entries.** Expanded vocabulary, and variant spellings for words whose
  pronunciation genuinely differs between accents.
- **A `mergers` field.** Variant entries carry their accent label with a `Var`
  suffix (extending ReadLex's own `RRPVar` convention) plus a list naming the
  vowel mergers the spelling reflects. A consumer that ignores the field sees
  a ReadLex-shaped record.
- **`TrapBath` is no longer a variety of its own.** Those entries now publish
  as variant records carrying the `trap-bath` merger flag.
- **Bound forms live in a separate file.** Prefixes, suffixes and clitics —
  anything that cannot stand alone as a written word — publish to
  `readlex-affixes.json`, in the identical schema, with a hyphen marking the
  attachment side. The main dictionary contains only free-standing words, so
  nothing in it can be mistaken for one.
- **Regular inflections are not listed.** Consumers generate regular plurals
  and verb forms themselves; the dictionary lists only irregular ones, and the
  presence of an irregular form is the signal not to generate a regular one.

## How words are spelled

We adopt ReadLex's spelling principles wholesale for the canonical layer,
including its most consequential choice: spellings are based on **rhotic
Received Pronunciation (RRP)** — RP with every R pronounced. Because RRP keeps
every vowel and consonant distinction that the major accents make between
them, one RRP spelling can stand for all of them.

That gives the dictionary its shape:

- **An entry with only an RRP spelling covers every accent.** Most words need
  nothing more.
- **Where an accent genuinely differs, it gets its own entry**, labelled with
  that accent (General American, Canadian, Australian, and so on).
- **Absence is meaningful.** If a word has no General American entry, that is
  not a gap: it means the RRP spelling already covers American speech for that
  word.

## Accents, mergers, and variants

Much of the real variation between mainstream accents comes down to **vowel
mergers**: pairs of vowels that one accent keeps distinct and another
pronounces the same. Rather than treating every merged pronunciation as a
separate accent, we record it as the base accent plus a flag naming the
merger its spelling reflects. Three mergers are currently modelled:

- **trap–bath** — BATH words spoken with the short TRAP vowel rather than
  PALM: *basket* 𐑚𐑭𐑕𐑒𐑦𐑑 or 𐑚𐑨𐑕𐑒𐑦𐑑.
- **cot–caught** — the LOT and THOUGHT vowels collapsed into one,
  characteristic of North American speech.
- **lot–palm** (father–bother) — the LOT and PALM vowels collapsed, so
  *father* and *bother* rhyme: American *pond* 𐑐𐑭𐑯𐑛 beside canonical 𐑐𐑪𐑯𐑛.

The governing rule: **the variety that distinguishes the vowels is the
default; the one that merges them carries the flag.** RRP distinguishes
everything, so the canonical layer never needs a flag.

A few properties of the system worth knowing:

- **Flags are detected, never invented.** A merger flag is applied only where
  the merged spelling is an exact vowel-for-vowel counterpart of an attested
  unmerged spelling of the same word. We record variation the sources show;
  we do not generate it.
- **Before /r/, the same mergers surface on the r-coloured letters** (𐑹, 𐑸)
  rather than the plain vowels — the same sound change, written the way
  Shavian writes vowels before R.
- **One accent can hold both forms.** Merged and unmerged pronunciations are
  both mainstream in American English, so a word may carry more than one
  American entry, each right for its speakers.
- **The merged forms are optional by construction.** A reader or tool that
  wants no merged spellings can ignore flagged entries entirely and fall back
  on the canonical spelling, which always exists.
- Variation that no named merger accounts for is marked as a plain variant
  rather than forced into a category.

[Editors: the merger system is very much still work in progress. Which mergers
are named, the direction conventions, and how the flags publish are all still
being settled — expect this section to change, and check the decision log
before relying on details.]

## What gets an entry

An entry in the main dictionary is a free-standing written word. Beyond that:

- **Phrases and hyphenated words** are included when they carry a meaning of
  their own — when the whole says something the parts do not.
- **Bound forms** — prefixes like *anti-*, suffixes like *-ness*, clitics like
  *'ll* — are real dictionary material, glosses and all, but live in
  `readlex-affixes.json` rather than the main file.
- **Inflections**: only irregular ones (*sheep* as its own plural, *went*,
  *better*). Regular inflections are derivable and deliberately not listed.

## How entries are reviewed

Most candidate entries are produced by an automated pipeline from sources such
as WordNet and Wiktionary. **Nothing that pipeline produces is published
without human review**: every generated spelling, harvested variant and merger
tag is a candidate until an editor examines it and accepts, corrects, or
rejects it — no confidence score or batch rule substitutes for that judgement.
The one exception is the ReadLex core itself, which arrives already trusted.

Every editorial decision is recorded, attributed and reversible, and the
published dictionary is always exactly the result of those decisions applied
to the source material — the two cannot drift apart.

---

## Open questions for review (not part of the published document)

1. **Merger direction before publication.** The *pond* and *basket* examples
   follow the direction the code and your rulings pin; `docs/dialect-mergers.md`
   still states the opposite direction in places. Flagging again only because
   this document quotes a direction publicly.
2. **Second use of the editor-aside device.** I also converted the
   disclaimer's edit invitation into a square-bracket parenthetical, so the
   document speaks to end users throughout and to editors only inside
   brackets — two uses total. Say the word if the device should be reserved
   for the merger note alone.
