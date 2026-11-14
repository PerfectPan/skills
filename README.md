# skills

Agent skills shared across PerfectPan repositories. Each skill lives in `skills/<name>/` with a `SKILL.md` and the supporting resources it points to.

| Skill | Use |
| --- | --- |
| [code-cleanup](skills/code-cleanup/SKILL.md) | Audit low-value tests and duplicated implementations, then apply evidence-backed cleanup when requested. |
| [release-setup](skills/release-setup/SKILL.md) | Set up or migrate an npm repository's release flow to Changesets with a draft version PR and Trusted Publishing. |

Install one skill with the [Skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add PerfectPan/skills --skill code-cleanup
npx skills add PerfectPan/skills --skill release-setup
```

Add a skill after its practice has been proven in a real repository, and keep it in step with the canonical source it names.
