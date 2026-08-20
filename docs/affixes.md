# Bound forms and regular inflections

> Disclaimer: This initial draft was written by Claude based on my briefing - Joro.
>
> The contract below is stated as it will hold; the published data does not yet
> populate all of it, and the commands below report what is actually there on
> the day you run them.

**A consumer should not have to hardcode English morphology.** Between the two
published files and the conventions below, a client has what it needs to spell
an inflected form it has never seen: the affix file carries the bound forms and
which POS each produces, this document carries the conditions for attaching
them, and the main lexicon carries the forms no rule can produce. Everything
that follows is shaped by that aim — including the resolution contract, which
looks in the lexicon first because a miss there is itself the signal that the
regular rule applies.

The split between the files runs along a single line: what a client can derive,
and what it cannot. Regular inflections may be absent from the main lexicon,
because the conventions below are enough to get them back. **Irregular
inflections are stored in the main lexicon**, always, because nothing derives
them. That is what makes checking first informative rather than merely empty.

`readlex.json` holds free-standing written words; bound forms — anything that is
not one — publish to `readlex-affixes.json` in the **identical schema**, so a
consumer reading only the main dictionary never meets `-'ll` or `-ness` and
cannot mistake one for a word. Both files are written by the same commit, so
they cannot disagree.

The record schema, the accent lanes and the editorial vocabulary belong to
[`editing.md`](editing.md) and [`dictionary.md`](dictionary.md); this document
assumes them and does not restate them.

## The contract

To resolve a form:

1. **Check whether an explicit entry for the word exists in the main lexicon
   first**, under the POS you want. A record found is the answer; stop.
2. **On a miss, find the affix whose `pos` matches the POS you want.** A
   suffix's `pos` is what it **produces**.
3. **Apply it if the affix is `productive`.** An affix without `productive` is
   in the file so its definition can be shown; it is a witness rather than a
   licence to generate.
4. **Fuse per the phonotactic conventions below**, in both scripts.

Step one is what makes the rest safe, and the pruner is deliberately cautious to
keep it so. A record is dropped only when **both** spellings are derivable from
the lemma, because this dictionary is bidirectional and pruning asserts that a
client going either way can regenerate the form; one regular in Shavian but not
in Latin still ships. So **absence of a record is a positive claim: the regular
form is correct, in both scripts. Generate it.**

## `readlex-affixes.json`

Because the schema is shared with the main lexicon, an affix record needs some
way to say which side it attaches to, and the hyphen is it. **A form is bound
iff its Latin spelling carries a hyphen on one end — and its Shavian carries it
on the same end.** Trailing for a prefix (`anti-`, `𐑨𐑯𐑑𐑦-`), leading for a
suffix (`-ness`, `-𐑯𐑩𐑕`). Clitics are stored hyphenated too (`-'s`, `-'ll`), so
the one test covers them. The hyphen is the only encoding: it states the
attachment side and marks the entry as a fragment to the eye.

That is why dispatch goes on the hyphen's position together with the produced
POS, rather than on the POS tag alone. **A suffix's `pos` names what it makes,
not what it attaches to**, which is the orientation a generating client wants:
`-ness` is `NN1` because it makes nouns, `-ish` is `AJ0` because it makes
adjectives. A client wanting to build a noun from an adjective therefore looks
for `NN1`-tagged suffixes.

`productive` is the licence to generate: which affixes a client may still apply
to a stem the dictionary has never paired them with. Ticked, a client may apply
the affix to any stem of the right POS and trust the result. **Absent is a
positive statement, not a gap** — the affix is recorded for what it explains
about existing words, not for making new ones.

    python3 -c "import json,collections;d=json.load(open('readlex-affixes.json'));\
    print(sum(len(v) for v in d.values()),'records',\
    collections.Counter(r['pos'] for v in d.values() for r in v),\
    sum(1 for v in d.values() for r in v if r.get('productive')),'productive')"

## Phonotactic conventions

How the affix fuses to the stem — the part the two files cannot state entry by
entry, and what the pruning bargain rests on. Seven tags take regular
inflections: `NN2`, `VVZ`, `VVD`, `VVN`, `VVG`, `AJC`, `AJS`.

### Shavian

`-s` — `NN2`, `VVZ`:

| Stem ends | Suffix | Example |
|---|---|---|
| sibilant (𐑕 𐑟 𐑖 𐑠 𐑗 𐑡) | `𐑩𐑟` | *boxes* 𐑚𐑪𐑒𐑕𐑩𐑟 |
| other voiceless obstruent (𐑐 𐑑 𐑒 𐑓 𐑔) | `𐑕` | *cats* 𐑒𐑨𐑑𐑕 |
| elsewhere | `𐑟` | *dogs* 𐑛𐑪𐑜𐑟 |

`-ed` — `VVD`, `VVN`:

| Stem ends | Suffix | Example |
|---|---|---|
| 𐑑 or 𐑛 | `𐑩𐑛` | *needed* 𐑯𐑰𐑛𐑩𐑛 |
| any other voiceless obstruent (incl. 𐑕 𐑖 𐑗) | `𐑑` | *rushed* 𐑮𐑳𐑖𐑑 |
| elsewhere | `𐑛` | *paddled* 𐑐𐑨𐑛𐑩𐑤𐑛 |

**The epenthetic schwa voices the following obstruent.** Once a vowel is
inserted, the ending is necessarily `𐑟` / `𐑛` — so each set above is three
allomorphs, and there is **no `-𐑩𐑕`** and **no `-𐑩𐑑`**. Composing "insert a
schwa after a sibilant" with "devoice after a voiceless obstruent" as
independent rules yields a fourth cell that does not exist. It looks like a 2×2
grid; it is three cells.

`-𐑩𐑕` *is* a real ending elsewhere — `-ess`, `-ous` (*politeness* 𐑐𐑩𐑤𐑲𐑑𐑯𐑩𐑕,
*abacus* 𐑨𐑚𐑩𐑒𐑩𐑕). There the schwa is **lexical**, part of the suffix itself,
so nothing voices what follows. Same shape, different phenomenon.

The remaining three take one form each, with no allomorphy:

| Tag | Suffix | Example |
|---|---|---|
| `VVG` | `𐑦𐑙` | *taking* 𐑑𐑱𐑒𐑦𐑙 |
| `AJC` | `𐑼` | *brighter* 𐑚𐑮𐑲𐑑𐑼 |
| `AJS` | `𐑩𐑕𐑑` | *brightest* 𐑚𐑮𐑲𐑑𐑩𐑕𐑑 |

Note `AJS` carries its own schwa — `𐑩𐑕𐑑`, not `𐑕𐑑`.

### Latin

Not symmetric with the Shavian, and **set-valued**: English licenses more than
one regular spelling for some stems, so generate the set and accept any member.
Electing a favourite would call the other spelling irregular, which is a claim
the data does not make.

| Rule | Condition | Example |
|---|---|---|
| e-insertion | stem ends `s ss x z sh ch`, before `-s` | *box* → *boxes* |
| `y` → `i` | stem ends consonant + `y`; **blocked before `-ing`** | *carry* → *carried*, *carries*; but *carrying* |
| silent-`e` deletion | stem ends `e`, before a vowel-initial suffix | *hope* → *hoping* |
| consonant doubling | CVC-final **and final syllable stressed** | *permit* → *permitted*; *benefit* → *benefited* |
| `-l` doubling | stem ends `l`, **unconditionally** | *travel* → *travelled*, *traveled* |

**Doubling is conditioned on stress, not on letters.** *permit* and *benefit*
end in the same three letter-shapes and behave oppositely. Stress must be read
from the lemma's `ipa`; with no pronunciation available, no doubling is
licensed. `qu` counts as the consonant it is spoken as, so *acquit* is CVC.

**`-l` stems are the exception: they double regardless of stress.** *travelled*
and *traveled* are both correct, one per variety, and `var` does **not**
separate them — both sit under plain `RRP`. The Shavian is identical either
way; doubling is a Latin-side phenomenon only.

## What the lexicon answers instead

**Stem alternations are lexical, so they are looked up rather than computed.**
Final `f`→`v` and `th`→`dh` alternate in some plurals and not others, and the
stem's sounds do not predict which: *elf* → *elves* alternates while *belief* →
*beliefs* (𐑚𐑦𐑤𐑰𐑓𐑕) does not, though both stems end in the same 𐑓. *bath* →
*baths* alternates; *month* → *months* does not.

*bath* also shows the alternation is per slot, not per stem: the noun plural is
𐑚𐑭𐑞𐑟 while the verb `VVZ` of the same spelling is 𐑚𐑭𐑔𐑕.

The guess is never needed, and this is the pruning bargain paying out. An
alternating plural is not its stem plus any listed affix, so the pruner classes
it irregular and **it always ships** — the lookup is guaranteed to be there.
Generation happens only where the record is absent, and by construction those
are the non-alternating cases.

**An attested irregular suppresses generation for its own POS slot only.**
*children* ships, so *child* needs no generated `NN2` — but a headword with an
irregular plural still takes a regular past tense, and vice versa. The mask is
per (lemma, POS), never per lemma. *fly* is the case to reason from: it carries
irregular *flew* (`VVD`) and *flown* (`VVN`), and a client that let those
suppress the whole headword would lose the perfectly regular *flies*. Apply the
mask per slot and you get all three right.

Every record carries a `lemma` link, so segmenting a derived form is a lookup,
not a parse.
