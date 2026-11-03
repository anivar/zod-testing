# zod-testing — merged into [zod-skill](https://github.com/anivar/zod-skill)

The testing guidance here now also ships inside the `zod` skill, alongside
schema design, parsing, error handling and inference. New work happens there.

**If you are installing fresh, install [`zod-skill`](https://github.com/anivar/zod-skill)** —
one skill covers writing schemas and testing them, and the testing material is
at `references/testing.md`, `references/testing-api.md` and
`references/testing-anti-patterns.md`.

**If you already have `zod-testing` installed, it keeps working.** The skill is
intact and this repository is not archived; nothing you depend on has been
removed. Switch when it suits you.

Created by **[Anivar Aravind](https://anivar.net)**

---

Merged because testing schemas is not a separate discipline from writing them.
The guide in this repository is generated from the zod skill's testing references
by `.github/check/sync-testing.ts` in anivar/zod-skill. A check in this repository
fails when the two copies differ. Edit the zod-skill copy, then run the sync.
