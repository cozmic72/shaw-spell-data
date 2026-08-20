# Bound forms and regular inflections

> Disclaimer: This initial draft was written by Claude based on my briefing - Joro.

Two things a consumer of `readlex.json` needs and cannot read off the file: what
`readlex-affixes.json` holds, and how to reproduce the regular inflections the
export leaves out.

The second is the reason this document exists. The export is designed to prune
regular inflected forms because a client can derive them. That is a bargain: the
dictionary stays small on the promise that the rules below are enough to get the
forms back. Everything here is stated so that promise can be kept.

**Pruning is not yet in effect.** As of 2026-08-20 the published `readlex.json`
still carries its regular inflections — the export stage runs but prunes
nothing, because the judgement it reads is not yet stamped on the published
basis. So a form's absence today usually means the dictionary never had it, not
that it was pruned. Write your reader to generate on absence anyway: that is the
contract the file is moving towards, and generating a form that also ships costs
you a duplicate, while failing to generate one costs you the form.

    python3 -c "import json;d=json.load(open('readlex.json'));\
    print(sum(1 for v in d.values() for r in v if r['pos'] in\
    ('NN2','VVZ','VVD','VVN','VVG','AJC','AJS')),'inflected records still shipping')"

The record schema, the accent lanes and the editorial vocabulary belong to
[`editing.md`](editing.md) and [`dictionary.md`](dictionary.md); this document
assumes them and does not restate them.

## `readlex-affixes.json`

Bound forms — anything that is not a free-standing written word — publish here
instead of `readlex.json`, in the **identical schema**. The split is so that a
consumer reading only the main dictionary never meets `-'ll` or `-ness` and
cannot mistake one for a word. Both files are written by the same commit, so
they cannot disagree.

**A form is bound iff its Latin spelling carries a hyphen on one end — and its
Shavian carries it on the same end.** Trailing for a prefix (`anti-`, `𐑨𐑯𐑑𐑦-`),
leading for a suffix (`-ness`, `-𐑯𐑩𐑕`). Clitics are stored hyphenated too
(`-'s`, `-'ll`), so the one test covers them. The hyphen is the only encoding:
it states the attachment side and marks the entry as a fragment to the eye. A
record hyphenated on one side only is malformed, not a licence to complete the
other.

Do not key on the POS tag. `PRE` is ReadLex's own extension with no suffix
counterpart, and a suffix's tag is needed for something else — see below.

    python3 -c "import json;d=json.load(open('readlex-affixes.json'));\
    print(sum(len(v) for v in d.values()),'records')"

### What it holds today

**Every published record is currently a prefix, tagged `PRE`.** As of
2026-08-20 the file publishes 66 records, all trailing-hyphen, all `PRE`, all
`var: RRP`. No suffix and no clitic has yet been published.

This matters for planning against the file: the suffix and clitic machinery is
built and tested, and `editing.md` describes `-ness` and `'ll` as belonging
here, but nothing has been moved yet. **Write your reader to dispatch on the
hyphen's position, not on `PRE`**, and it will keep working when suffixes
arrive. Check the distribution rather than assuming it:

    python3 -c "import json,collections;d=json.load(open('readlex-affixes.json'));\
    print(collections.Counter(r['pos'] for v in d.values() for r in v))"

### `pos` on a suffix, when suffixes ship

A suffix's `pos` names what it **produces**, not what it attaches to: `-ness` is
`NN1` because it makes nouns, `-ish` is `AJ0` because it makes adjectives. A
client wanting to build a noun from an adjective therefore looks for
`NN1`-tagged suffixes. A prefix's `PRE` says only "prefix" and names nothing.

Expect `UNC` to be common among harvested suffixes: where no source states what
a suffix produces, the tag records that silence rather than a guess. **`UNC` is
not a produced POS** — do not read it as one.

### `productive`

The field that would license generation. Ticked, a client may apply the affix to
any stem of the right POS and trust the result; **absent, the record is a
witness only** — it exists so a definition can be shown, and you must not
generate from it. Absence is a positive statement, not a gap.

**It is populated on no record today**, because the value is the owner's
editorial judgement rather than anything the pipeline can measure. Until it
appears, treat every affix as witness-only.

## Reproducing pruned inflections

A record is pruned only when **both** its Latin and its Shavian are derivable
from its lemma. So for the seven tags below:

> **Absence of a record is a positive claim: the regular form is correct, in
> both scripts. Generate it.**

An **attested irregular suppresses generation for its own POS slot only.**
*children* ships, so generate no `NN2` for *child* — but a headword with an
irregular plural still takes a regular past tense, and vice versa. The mask is
per (lemma, POS), never per lemma. *fly* is the case to reason from: it carries
irregular *flew* (`VVD`) and *flown* (`VVN`), and a client that let those
suppress the whole headword would lose the perfectly regular *flies*. Apply the
mask per slot and you get all three right.

Every record carries a `lemma` link, so segmenting a derived form is a lookup,
not a parse.

    python3 -c "import json,collections;d=json.load(open('readlex.json'));\
    print(collections.Counter(r['pos'] for v in d.values() for r in v\
    if r['pos'] in ('NN2','VVZ','VVD','VVN','VVG','AJC','AJS')))"

### Shavian

Measured over the published set at 2026-08-20, the conditioning below is
exceptionless within each environment.

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
inserted, the ending is necessarily `𐑟` / `𐑛` — so the `-s` set is three
allomorphs, not four, and there is **no `-𐑩𐑕`** and **no `-𐑩𐑑`**. Composing
"insert a schwa after a sibilant" with "devoice after a voiceless obstruent" as
independent rules yields a fourth cell that does not exist. It looks like a 2×2
grid; it is three cells.

`-𐑩𐑕` *is* a real ending elsewhere — `-ness`, `-ess`, `-ous` (*politeness*
𐑐𐑩𐑤𐑲𐑑𐑯𐑩𐑕, *abacus* 𐑨𐑚𐑩𐑒𐑩𐑕). There the schwa is **lexical**, part of the
suffix itself, so nothing voices what follows. Same shape, different
phenomenon.

The remaining three take one form each, with no allomorphy:

| Tag | Suffix | Example |
|---|---|---|
| `VVG` | `𐑦𐑙` | *taking* 𐑑𐑱𐑒𐑦𐑙 |
| `AJC` | `𐑼` | *brighter* 𐑚𐑮𐑲𐑑𐑼 |
| `AJS` | `𐑩𐑕𐑑` | *brightest* 𐑚𐑮𐑲𐑑𐑩𐑕𐑑 |

Note `AJS` carries its own schwa — `𐑩𐑕𐑑`, not `𐑕𐑑`.

### Latin

Not symmetric with the Shavian, and not deterministic. Applied in this order:

| Rule | Condition | Example |
|---|---|---|
| e-insertion | stem ends `s ss x z sh ch`, before `-s` | *box* → *boxes* |
| `y` → `i` | stem ends consonant + `y`; **blocked before `-ing`** | *carry* → *carried*, *carries*; but *carrying* |
| silent-`e` deletion | stem ends `e`, before a vowel-initial suffix | *hope* → *hoping* |
| consonant doubling | CVC-final **and final syllable stressed** | *permit* → *permitted*; *benefit* → *benefited* |

**Doubling is conditioned on stress, not on letters.** *permit* and *benefit*
end in the same three letter-shapes and behave oppositely. Stress must be read
from the lemma's `ipa`; with no pronunciation available, no doubling is
licensed. `qu` counts as the consonant it is spoken as, so *acquit* is CVC.

**This rule is not exact.** Taken together, the four rules above reproduce about
96% of the published inflected forms (measured 2026-08-20; the command below
re-runs it). Two consequences a client must handle:

- **`-l` stems admit both spellings.** *travelled* and *traveled* are both
  correct, one per variety, and `var` does **not** separate them — both sit
  under plain `RRP`. Treat `-l` doubling as optional in both directions, and
  note the Shavian is the same for both.
- **The residue is real.** Because more than one spelling can be regular, the
  pipeline's own test accepts a **set** of licensed spellings rather than
  electing a favourite, and prunes only when the attested form is in that set.
  *benefited* and *benefitted* both ship for this reason.

So: an absent record licenses you to generate the regular form; it does **not**
guarantee your single guess matches what we would have written. Where a client
needs certainty for a specific stem, generate the set and accept any member.

To re-measure, with `src/tools` from
[`shaw-spell-editor`](https://github.com/cozmic72/shaw-spell-editor) on the path
— `latin_forms` is the rule as the pipeline applies it:

```python
import json, collections
from inflection import latin_forms
d = json.load(open("readlex.json"))
ipa = {}
for v in d.values():
    for r in v:
        ipa.setdefault((r["Latn"].lower(), r["pos"]), r.get("ipa", ""))
res = collections.Counter()
for v in d.values():
    for r in v:
        lem = r.get("lemma")
        if r["pos"] not in ("NN2","VVZ","VVD","VVN","VVG","AJC","AJS") or not lem:
            continue
        b, f = lem["Latn"].lower(), r["Latn"].lower()
        if not b.isalpha() or not f.isalpha():
            continue
        res[f in latin_forms(b, r["pos"], ipa.get((b, lem["pos"]), ""))] += 1
print(res, res[True] / sum(res.values()))
```

### What not to derive

**Stem alternations are lexical. Look them up; do not compute them.** Final
`f`→`v` and `th`→`dh` alternate in some plurals and not others, and the stem's
sounds do not predict which: *elf* → *elves* alternates while *belief* →
*beliefs* (𐑚𐑦𐑤𐑰𐑓𐑕) does not, though both stems end in the same 𐑓. *bath* →
*baths* alternates; *month* → *months* does not. Roughly a fifth of eligible
stems alternate, so a rule guessing either way is wrong most of the time it
fires.

*bath* also shows the alternation is per slot, not per stem: the noun plural is
𐑚𐑭𐑞𐑟 while the verb `VVZ` of the same spelling is 𐑚𐑭𐑔𐑕.

An inverse recognition rule is worse. "Final 𐑝𐑟 may be an f-stem plural" fires
on several times more records than it should, most of them wrong.

You never need the guess. An alternating plural is not its stem plus any listed
affix, so the pruner classes it irregular and **it always ships** — the lookup
is guaranteed to be there. Generate only where the record is absent, and by
construction those are the non-alternating cases.

```python
import json, collections
d = json.load(open("readlex.json"))
alt = collections.Counter()
for v in d.values():
    for r in v:
        lem = r.get("lemma")
        if r["pos"] != "NN2" or not lem: continue
        ls, s = lem["Shaw"], r["Shaw"]
        if not ls.endswith("\U00010453"): continue          # stem ends f
        if s.startswith(ls[:-1] + "\U0001045D"): alt["v"] += 1
        elif s.startswith(ls): alt["f"] += 1
print(alt)
```

### Worked examples

*abatement* (`NN1`, 𐑩𐑚𐑱𐑑𐑥𐑩𐑯𐑑) — no `NN2` is published, so generate one:

| Tag | Latin | Shavian | Rule |
|---|---|---|---|
| `NN2` | *abatements* | 𐑩𐑚𐑱𐑑𐑥𐑩𐑯𐑑𐑕 | stem ends 𐑑, voiceless → `𐑕`; plain `-s` |

*permit* (`VVI`, 𐑐𐑼𐑥𐑦𐑑, `pəRˈmɪt`) — the full paradigm, and the doubling case:

| Tag | Latin | Shavian | Rule |
|---|---|---|---|
| `VVZ` | *permits* | 𐑐𐑼𐑥𐑦𐑑𐑕 | stem ends 𐑑, voiceless → `𐑕` |
| `VVD`/`VVN` | *permitted* | 𐑐𐑼𐑥𐑦𐑑𐑩𐑛 | stem ends 𐑑 → `𐑩𐑛` |
| `VVG` | *permitting* | 𐑐𐑼𐑥𐑦𐑑𐑦𐑙 | invariant `𐑦𐑙` |

Latin doubles: CVC-final and `ˈ` falls on the last syllable. *benefit*
(`ˈbenɪfɪt`) is the same CVC shape with initial stress, so it does not double —
though the published set attests *benefitted* alongside *benefited*, which is
what the set-valued rule above is for. Note the Shavian is identical either way:
doubling is a Latin-side phenomenon only.

*box* (`NN1`, 𐑚𐑪𐑒𐑕) — the epenthesis case:

| Tag | Latin | Shavian | Rule |
|---|---|---|---|
| `NN2` | *boxes* | 𐑚𐑪𐑒𐑕𐑩𐑟 | stem ends 𐑕, sibilant → `𐑩𐑟`; Latin takes `-es` |

The epenthetic schwa forces `𐑟`; `𐑚𐑪𐑒𐑕𐑩𐑕` is not a possible form.

### Known gaps

- **Doubling's residue is not fully characterised.** The rule above is the one
  the pipeline applies; the forms it misses are shipped rather than pruned, so
  no data is lost — but a client generating from the rule alone will
  occasionally differ from us on a stem where we would have shipped.
- **Stress comes from the lemma's `ipa`.** A lemma with no pronunciation
  licenses no doubling, so a regular doubled form under such a lemma is shipped
  rather than pruned.
