# Shaw-Spell — spelling principles and editorial policy

> **This is a living document, and provisional throughout.** It describes the
> project's current thinking, not a finished standard; the dictionary and its
> conventions are still evolving, and this text evolves with them.
> [Editors: you can — and should — edit this document as the conventions
> themselves move.]
> 
> Disclaimer (JORO): This initial draft was largely written by Claude 
> based on my briefing - I have given it a once over but it's not yet where 
> I want it.  Feel free to chip in!

## What Shaw-Spell is

Shaw-Spell is built on top of the [ReadLex](https://readlex.pythonanywhere.com)
(Kingsley Read Lexicon), the Shavian pronunciation dictionary by [Shavian.info](https://shavian.info).  Shaw-spell takes the latest version of Readlex and on top of that, it:

- **expands the vocabulary**, spelling new words following the same conventions 
  as laid out by Realex's author
- **adds new ponunciations**, to offer Shavian users the option to spell words   
  the way they say them 

The second goal is where we go beyond ReadLex's remit. ReadLex describes
itself as neither prescriptive nor descriptive but *selective*: typically it
has just one standard spelling per word, variant spellings are few and far between. 
Shaw-spell sticks to Readlex's selective standard as its default dictionary entries, but adds many more pronunciations across more mainstream accents.

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
  attachment side. The `readlex.json` export contains only free-standing words.
- **Regular inflections are not listed.** Consumers generate regular plurals
  and verb forms themselves; the dictionary lists only irregular ones, and the
  presence of an irregular form is the signal not to generate a regular one.
  
[Note to editors (JORO): this last point  is not entirely true yet.  This is where we are heading though.]

## How words are spelled

We adopt ReadLex's spelling principles for the default entries of 
words.  Default spellings are always based on **rhotic
Received Pronunciation (RRP)** — RP with every R pronounced.

Following Readlex's structure:

- **An entry with only an RRP spelling is used for every dialect.**
- **Where a major dialect genuinely differs, it gets its own entry**, labelled with
  that accent (General American, Canadian, Australian, and so on).
- In cases where there are multiple variants of a word in one major dialect, 
  we will flag alternatives as 'variations', using the '-Var' suffix for the 
  region, and one or more flags specifying what specific kind of variation it is.


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
- **other** - variations that don't clearly fall under the three mergers
  listed get put into a separate 'other' bucket.

As a rule, **the variety that distinguishes the vowels is the
default; the one that merges them carries the flag.**  

[Editors: the merger system is very much still work in progress. Which mergers
are named, the direction conventions, and how the flags publish are all still
being settled — expect this section to change.]

## What gets an entry

An entry in the main dictionary is a free-standing written word. Beyond that:

- **Phrases and hyphenated words** are included when they carry a meaning of
  their own — when the whole says something the parts do not.
- **Bound forms** — prefixes like *anti-*, suffixes like *-ness*, clitics like
  *'ll* — are real dictionary material, glosses and all, but live in
  `readlex-affixes.json` rather than the main file.
- **Inflections**: only irregular ones (*sheep* as its own plural, *went*,
  *better*). Regular inflections are derivable and deliberately not listed.

## How new dictionary entries are curated

Candidate entries are produced by an automated pipeline from sources such
as WordNet and Wiktionary, or they are entered into the system by hand. 
**Nothing that pipeline produces is published without human review**: every generated spelling, harvested variant and merger
tag is a candidate until an editor examines it and accepts, corrects, or
rejects it — no confidence score or batch rule substitutes for that judgement.
