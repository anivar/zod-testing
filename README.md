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
Keeping the guidance in one place is also the only way to stop the two copies
drifting apart as Zod moves — it is at 4.4.3 as of August 2026, and the APIs
worth teaching change with it.
