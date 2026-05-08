# Contributing

Thanks for your interest in contributing to Agentic Engineering for Apache Kafka. This repository is maintained by [Lenses.io](https://lenses.io) and we welcome contributions from the community: new skills, subagents, hooks, bug fixes, doc improvements and prompt-engineering tweaks are all in scope.

## Ways to contribute

A simple heuristic: if you have caught yourself coaching an AI agent through the same Kafka problem more than twice, that is a skill waiting to be written. Open an issue or send the PR.

The skills serve three rough engineer profiles, and contributions are welcome across all of them:

- **Data engineers:** schema compatibility, pipeline reliability, data quality guarantees, lineage, drift detection.
- **Backend developers:** produce/consume correctly without getting buried in Kafka internals; idempotency, retries, graceful shutdown, error handling, DLQs.
- **Streaming developers:** state stores, windowing, joins, exactly-once processing, Kafka Streams and ksqlDB workflows.

Concrete things to contribute:

- **Add a new Kafka skill.** Good candidates include Kafka Streams analysis, ksqlDB, MirrorMaker setup and audit, deeper Schema Registry workflows (compatibility matrices, evolution playbooks), broker configuration audits, capacity planning, partition rebalance review, ACL audits, quota tuning and tiered storage review.
- **Add support for a different Kafka MCP server.** The skills in this repo are observed against [Lenses MCP](https://github.com/lensesio/lenses-mcp). If you run a different Kafka MCP server, fork the relevant skill, swap the MCP tool calls and submit a PR with the variant. That is exactly the kind of contribution this repo is designed to take.
- **Add a new general-purpose skill or subagent** that complements the Kafka work, for example a release-notes generator, a runbook drafter or a load-test helper.
- **Improve an existing skill.** Tighten triggers, expand `references/`, add test cases, sharpen success criteria, fix bugs.
- **Improve hooks, settings or customisation** in `.claude/`, the `AGENTS.md`/`CLAUDE.md` memory files or the cross-tool support story.
- **Report a bug** by opening an issue with the smallest reproduction you can give us.
- **Improve the docs:** README, TROUBLESHOOTING or skill-level docs.

If you are not sure whether an idea is in scope, please open an issue first to discuss it.

## Repository layout

```
.claude/                    Claude Code skills, subagents, settings and hooks
.cursor/                    Cursor skills and subagents (mirror of .claude/ minus settings)
AGENTS.md                   Memory file consumed by Cursor
CLAUDE.md                   Memory file consumed by Claude Code
README.md                   Project overview and usage
TROUBLESHOOTING.md          Cross-skill troubleshooting
CONTRIBUTING.md             You are here
LICENSE                     MIT licence
```

Every skill and subagent lives in **both** `.cursor/` and `.claude/`. We will not merge a skill that only ships for one editor; the whole point of the repo is parity across the two.

## How a skill is structured

Skills follow the [Anthropic open standard for skills](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf), specifically the progressive-disclosure pattern:

```
<editor>/skills/<skill-name>/
├── SKILL.md                 Top-level instructions, frontmatter, examples, troubleshooting
└── references/              Detail loaded on demand: lookup tables, test cases, etc.
    ├── test-cases.md
    └── …                    Other reference files specific to the skill
```

`SKILL.md` must:

- Start with YAML frontmatter (delimited by `---`) including:
  - `name`: kebab-case, must match the folder name
  - `description`: one to three sentences with explicit trigger phrases (what users would actually say) and at least one negative trigger ("Do NOT use for X"). The description is what the agent matches on, so be specific.
  - `license`: `MIT`
  - `metadata`: `author`, `version` (semver), and `mcp-server` if the skill depends on an MCP server (e.g. `lenses-mcp`)
  - `compatibility`: list of editors and platforms supported
  - `category`: one of `mcp-enhancement`, `workflow-automation`, etc.
  - Architecture `approach` (problem-first or tool-first) and `patterns` (sequential-workflow, iterative-refinement, context-aware-selection, domain-intelligence)
- Stay under ~5,000 words. Move detail to `references/`.
- Include an **Examples** section with concrete invocations.
- Include a **Troubleshooting** section for skill-specific issues. General platform issues belong in the top-level `TROUBLESHOOTING.md`.
- Define success criteria (quantitative and qualitative).
- Use validation gates between workflow steps so the workflow stops or adjusts when a step produces unexpected results.

`references/test-cases.md` must include three layers:

1. **Triggering tests:** lists of queries that *should* and *should not* load the skill.
2. **Functional tests:** Given / When / Then scenarios.
3. **Performance baselines:** tool calls, errors and user corrections with vs without the skill.

If your skill calls into the Lenses MCP server, document the MCP tool names you depend on (case-sensitive) and tolerate missing capabilities gracefully. For example, if Schema Registry is not configured in the target Lenses environment, treat it as a finding, not an error.

## Adding a new skill, step by step

1. **Open an issue** describing the skill, the target audience, and the queries it should match. We may have feedback before you build.
2. **Pick a name** in kebab-case. Kafka skills use the `kafka-` prefix (e.g. `kafka-topic-audit`); general skills do not.
3. **Scaffold** under both `.claude/skills/<name>/` and `.cursor/skills/<name>/` with the structure above. Look at an existing skill (`kafka-topic-audit/` is a good reference) and follow it.
4. **Write the SKILL.md** for both editors. The Claude Code variant may include extra configuration: explicit tool restrictions, model routing, persistent memory, inline bash pre-computation. Keep the user-visible behaviour identical across editors.
5. **Add `references/test-cases.md`** with all three test layers.
6. **Test in both editors** (Cursor and Claude Code) against a real Lenses environment if applicable. Community Edition is the easiest target.
7. **Update README.md** to add a row to the appropriate table.
8. **Update CLAUDE.md and AGENTS.md** if the skill introduces conventions the agent should remember.
9. **Open a PR** following the conventions below.

## Coding conventions

The example project uses Python tooling (`uv` for dependency management, `pytest` for tests, `ruff` for linting and formatting) because the bundled hooks demonstrate that stack. The skills themselves are language-agnostic. If you add language-specific examples, follow the conventions in [README.md → Conventions](README.md#conventions).

## Pull request guidelines

- **One concept per PR.** A new skill is one PR. A skill plus refactor is two PRs.
- **Follow the branch naming convention.** `feature/<name>`, `fix/<name>`, `docs/<name>`, `refactor/<name>`.
- **Write a clear PR description:** what the change does, why, what was tested, and what was not.
- **Keep CLAUDE.md and AGENTS.md in sync** if your change introduces a convention.
- **Run the tests and linters locally** before pushing. The Stop hook runs them automatically inside Claude Code; you can run them yourself with `uv run pytest` and `uv run ruff check`.
- **Reference the related issue** in the PR description.

## Code of conduct

Be kind. We expect contributors to interact respectfully: assume good faith, give specific feedback, accept feedback gracefully and remember that not everyone speaks English as a first language.

## Licence and contribution agreement

This repository is released under the [MIT License](LICENSE). By submitting a pull request you agree that your contribution will be licensed under the same terms.

## Questions

- Open an issue on this repository for skill-specific or roadmap questions.
- Join the Lenses Community Slack via [launchpass.com/lensesio](https://launchpass.com/lensesio) for general Lenses, Kafka or MCP questions.

Thanks for helping make Kafka engineering with AI agents a little less painful.
