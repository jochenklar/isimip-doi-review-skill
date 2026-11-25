---
name: isimip-doi-review
description: "Use this skill when the user asks to review, check, or verify a DOI record on draft.isimip.org (ISIMIP DOI drafts, e.g. 10.48364/ISIMIP.NNNNNN), or asks to write/create/draft an abstract for an ISIMIP DOI record. Trigger on 'review/check/again' + a draft.isimip.org URL, 'create an abstract for <doi>', or similar requests about ISIMIP dataset metadata, abstract typos, missing links, or related-identifier consistency."
---

# ISIMIP DOI record review

The user maintains ISIMIP4 dataset DOI records on `https://draft.isimip.org/doi/10.48364/ISIMIP.<id>`. Records are **edited live between checks** — the user will repeatedly ask "check again / again / ok now", expecting a re-fetch each time.

## Hard rules

1. **Re-fetch before every check** with `curl -sL`; drafts change between user messages. Save to `/tmp/<id>_n.html` and `diff` against the previous fetch to report exactly what changed. Never trust the previous fetch.
2. **Verify claims in the HTML, never assume.** A previously reported "fixed" item must be re-grepped before being reported as fixed (this error was made once and called out by the user).
3. **Spelling house style:** "dataset" (one word), "harvesting" (not "havesting"), consistent "land use" phrasing.
4. **"data" is always plural:** treat *data* as plural — "the data are", not "the data is"; flag "a data", "each data", "this data", suggestions: "the data are" or singular "a dataset"/"this dataset" as appropriate.
5. **Hyphens and en-dashes used correctly and consistently in the abstract only** (do not apply this rule to related-identifier labels or other metadata):
   - Numeric ranges use an **en-dash (–)**, without spaces: "1950–2020", "0–10 µm" (never "1950-2020" or "1950 – 2020").
   - **Hyphens (-)** connect compound modifiers before a noun: "land-use change", "sub-seasonal", "multi-decadal".
   - Compounds standing alone or after the noun use no hyphen/space per standard usage ("land use in…").
   - Compact arcminute unit forms such as `15arcmin` / `30arcmin` are acceptable; do not flag them.
   - Consistency matters: pick one form and use it everywhere in the abstract (and matching the sibling record where possible); flag every instance of an inconsistent variant.
6. **DOIs in the abstract only for dataset citations:** the abstract carries DOIs **only** for cited datasets (e.g. IsDerivedFrom items like WorldPop, FuturePop, HYDE, GHS-POP, SSP zenodo releases, COMPASS D3.1) — every such dataset mentioned must have its DOI next to the citation. Citations of papers/reports stay author-year only, **without** a DOI in the abstract (flag a paper DOI appearing in the abstract as a violation, and a dataset DOI missing from the abstract as one too). Every cited paper/report must still have a corresponding Related identifiers entry with relation type **`Cited`** (DOI/URL there), even though no DOI appears inline in the abstract; papers explicitly marked "in prep." are exempt and do not need a related identifier. Identical "author (year)" pairs in the abstract are **allowed** when their inline DOIs disambiguate them (e.g. two "Bondarenko et al., 2025" dataset cites with different DOIs); do not require 2025a/2025b suffixes in that case.

## Checklist (run on every review)

Extract the abstract region (`Abstract:` … `Publication year`), strip tags, normalize whitespace, then:

1. **Link coverage:** every URL in the abstract (and `www.isimip.org` inline) must appear in the "Related identifiers" section — compare with trailing slashes stripped. **Exception:** issues/notes on `data.isimip.org` (e.g. changelog issue links) do **not** need a related identifier.
2. **Orphan and citation check (per hard rule 6):** no **dataset** DOI in related identifiers absent from the abstract (dataset = IsDerivedFrom inputs like WorldPop, HYDE, GHS-POP, zenodo SSP/D3.1 releases). Conversely, **every paper/report cited in the abstract must appear in Related identifiers as `Cited`**; its DOI belongs there, not inline in the abstract. Papers cited as "in prep." are exempt and do not need a related identifier. isimip.org / protocol.isimip.org extras are OK.
3. **Related-identifier labels** should match the sibling record's own header (authors, title spelling).
4. **"data" plural + dash check (hard rules 4 & 5):** grep the abstract for `data` (flag singular uses: "data is", "a data", "this data") and scan **only the abstract's** ranges/compounds for correct, consistent en-dash (–) vs. hyphen (-) usage; flag every inconsistent variant, but ignore acceptable compact arcminute forms (`15arcmin`, `30arcmin`). Do not flag dash usage in related-identifier labels or other metadata.
5. Report as: ✅/❌ table or bullet list, concrete fixes, and note items fixed vs. still outstanding. Keep it concise.
