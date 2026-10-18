Saved full-page screenshots in [docs/screenshots](/home/camara/Aegis/docs/screenshots). They use a **synthetic demo database**; your project database was not changed.

- [Overview](/home/camara/Aegis/docs/screenshots/overview.png)
- [Assessment history](/home/camara/Aegis/docs/screenshots/assessments.png)
- [Assessment comparison](/home/camara/Aegis/docs/screenshots/compare.png)
- [Assessment detail](/home/camara/Aegis/docs/screenshots/assessment-detail.png)

I also fixed the comparison view so inventory rows display their resource identities. The full test suite reports **105 passed, 1 skipped**; the skip is the opt-in Floci integration test. Git diff checks passed. Nothing was staged or committed.

### LinkedIn post draft

I’ve been building **Project AEGIS**, a local-first, read-only cloud security posture assessment tool.

AEGIS brings together a Python CLI and a local dashboard to discover cloud inventory, run a small set of evidence-based checks, and review persisted assessment history. Findings link back to evidence references, and assessments can be compared and exported as JSON or Markdown.

A core design goal is to show when the evidence is incomplete. AEGIS keeps `PASS`, `FAIL`, `NOT_SUPPORTED`, and `ERROR` distinct, so missing API data is not presented as a clean result.

The dashboard reads saved assessments only. It does not trigger cloud collection or modify cloud resources. The screenshots use synthetic demo data.

This is an early project: its checks cover a limited set of verified API responses, and the local dashboard is not intended for remote deployment.

#CloudSecurity #Python #CyberSecurity #OpenSource #AWS

**Files to publish:** choose the four images above. The current Git changes also include the UI renderer fix; screenshots remain untracked.