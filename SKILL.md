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

## Checklist (run on every review)

Extract the abstract region (`Abstract:` … `Publication year`), strip tags, normalize whitespace, then:

1. **Link coverage:** every URL in the abstract (and `www.isimip.org` inline) must appear in the "Related identifiers" section — compare with trailing slashes stripped.
2. **Orphan check:** no DOI in related identifiers that is absent from the abstract (isimip.org / protocol.isimip.org extras are OK).
3. **Related-identifier labels** should match the sibling record's own header (authors, title spelling).
4. Report as: ✅/❌ table or bullet list, concrete fixes, and note items fixed vs. still outstanding. Keep it concise.
