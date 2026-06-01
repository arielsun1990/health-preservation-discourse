# Pre-submission action guide: Table 2 reconciliation + H1 linguistic baseline

Two things to finish before Scientific Reports. Part 1 is a quick arithmetic fix that an editor or reviewer *will* notice. Part 2 is the substantive control that decides whether H1's psychological reading survives review.

---

## Part 1 — Reconcile Table 2 and the H1 statistics

**The problem.** Table 2 ("X-Yin-Y-Yang") lists 19 semantic frames whose token counts sum to **29**, but the text states **n = 31**. Because the yin-first total = Table 1 (10) + Table 2, this propagates straight into the headline test.

**Why it matters.** The binomial result changes depending on the true count:

| Yin-first / total | Two-sided exact binomial p | Wilson 95% CI | Cohen's g |
|---|---|---|---|
| 41 / 63 (as written now) | **0.0226** | [0.528, 0.757] | 0.151 |
| 40 / 62 | 0.0300 | [0.521, 0.753] | 0.145 |
| 39 / 61 (if Table 2 = 29) | **0.0396** | [0.514, 0.748] | 0.139 |

Still significant in every scenario, but the 39/61 case sits close to the .05 line — so get the count right.

**Steps.**
1. Re-export the `X-Yin-Y-Yang` concordance from AntConc (or re-run your regex) and recount tokens per frame.
2. Find the missing 2 tokens: either two frames were dropped from the table, or two counts are understated. Fix whichever it is.
3. Recompute the yin-first total = 10 + (corrected Table 2).
4. Re-run the exact binomial test on the corrected `k / n` (snippet below).
5. Propagate the corrected numbers to **all four places**: Table 2 caption, the Results sentence ("41 tokens … 22 tokens; … p = .022"), Figure 4, and the Abstract.

```python
from scipy.stats import binomtest
k, n = 41, 63          # <-- replace with your corrected counts
r = binomtest(k, n, 0.5, alternative="two-sided")
ci = r.proportion_ci(method="wilson")
print(f"prop={k/n:.4f}  p={r.pvalue:.4f}  Wilson95=[{ci.low:.3f}, {ci.high:.3f}]  g={abs(k/n-0.5):.3f}")
```

---

## Part 2 — The H1 linguistic baseline control

**Why reviewers will demand it.** In Chinese the binome itself is conventionally yin-first (阴阳, not 阳阴), and the ordering of coordinate/binomial expressions is partly governed by phonology, tone, frequency, and markedness — independent of meaning (Cooper & Ross 1975; Benor & Levy 2006). So a yin-first surface bias might be *language*, not *cultural psychology*. You need a baseline that isolates the "excess" attributable to a cultural-evaluative preference. Run at least one of the two controls below; Control A is fastest, Control B is stronger.

### Control A — within-corpus generalisation test (no external data needed)

**Logic.** If the bias reflects a *cultural-evaluative* preference (the storing / inward / containing pole leads), it should generalise beyond yin/yang to other medical antonym pairs. If only yin–yang shows it, the effect is probably a yin–yang-specific lexical freeze.

1. Pre-specify (to avoid circularity) which pole is the "storing/inward" pole for a set of paired opposites, e.g.: 虚/实 (deficiency/excess), 寒/热 (cold/hot), 表/里 → 里/表 (interior/exterior), 升/降 → 降/升 (descend/ascend), 收/散 (gather/disperse), 补/泻 (tonify/drain), 沉/浮 (sink/float), 静/动 (quiet/active), 内/外 (inner/outer), 阖/开 (close/open). Decide the predicted "leads-first" member **before** counting.
2. For each pair, count orderings in productive four-character / coordinate constructions (same template logic as yin–yang; exclude any frozen binome).
3. Test whether the predicted (storing) pole leads more often than chance **across pairs** — a sign test / binomial over the set of pairs, plus a pooled binomial over all tokens.
4. **Interpretation:** storing-pole-leads generalises across pairs → supports the cultural-evaluative reading and rebuts the "yin–yang freeze" objection. Only yin–yang shows it → report H1 as a lexical-specific pattern, not a general schema.

### Control B — cross-corpus baseline (stronger, needs a reference corpus)

**Logic.** The frozen binome sets a baseline yin-first rate in general Chinese. The cultural hypothesis predicts the *Compendium's productive constructions exceed that baseline*.

1. Choose a contemporaneous, **non-medical** reference corpus (candidates: Kanseki Repository / Kanripo, ctext.org Ming-era philosophical or literary texts, or a neutral-genre Siku subset). Match register/era as closely as feasible.
2. Extract the same four templates with the same regex (patterns below).
3. Compute the yin-first proportion in each corpus.
4. Compare with a 2×2 test (Fisher's exact or two-proportion z); report the odds ratio.
5. **Interpretation:** Compendium rate significantly *higher* than baseline → that excess is the cultural/medical signal (strengthens H1). Rate ~equal → the asymmetry is general-language convention, and H1's psychological reading should be dropped or heavily qualified.

```python
import re
from scipy.stats import fisher_exact

# four templates as raw-text regexes (4-char windows); de-duplicate and verify hits manually
pat = {
    "yin_first":  [r"阴(.)阳(.)", r"(.)阴(.)阳"],   # Yin-X-Yang-Y, X-Yin-Y-Yang
    "yang_first": [r"阳(.)阴(.)", r"(.)阳(.)阴"],   # Yang-X-Yin-Y, X-Yang-Y-Yin
}
def counts(text):
    yf = sum(len(re.findall(p, text)) for p in pat["yin_first"])
    af = sum(len(re.findall(p, text)) for p in pat["yang_first"])
    return yf, af

comp_yf, comp_af = counts(open("compendium.txt", encoding="utf-8").read())
base_yf, base_af = counts(open("reference_corpus.txt", encoding="utf-8").read())

odds, p = fisher_exact([[comp_yf, comp_af], [base_yf, base_af]])
print(f"Compendium {comp_yf}/{comp_yf+comp_af}, baseline {base_yf}/{base_yf+base_af}, OR={odds:.2f}, p={p:.4f}")
```
*(Regex hits over raw text will over-generate — keep your AntConc-verified, manually-checked token list as the real input and use the regex only to pull candidates from the reference corpus, then verify those by hand too.)*

### Also address the phonological/tonal alternative in the text
阴 (yīn, tone 1, 平) vs 阳 (yáng, tone 2, 阳平). Add one or two sentences noting that Chinese binome ordering is partly tone-governed, and state whether yin-first conforms to the general tonal-ordering tendency. If it does, name that as a competing explanation you cannot fully exclude (you already cite Cooper & Ross 1975 and Benor & Levy 2006 for this).

---

## What to write afterward
- **Controls support the cultural reading:** report the baseline comparison in Results (new sub-result under H1), and you can firm up the interpretation a notch.
- **Controls do not support it:** reframe H1 honestly — "a robust ordering asymmetry that a baseline test attributes to linguistic convention rather than cultural preference." This is still a clean, publishable finding, and stating it plainly is exactly what a soundness-based venue like Scientific Reports rewards.
