# claudeskills

Agents and skills for Claude Code. Task Router sits on top: you hand it a task, it works out which specialists own a slice of it, dispatches them in parallel, and drives the result through sign-off before anything ships.

## Install

Clone anywhere, then symlink into your Claude config:

```bash
git clone https://github.com/CarlosDanielS3/claudeskills.git
cd claudeskills

for f in agents/*.agent.md; do ln -sf "$PWD/$f" ~/.claude/agents/; done
for d in skills/*/;    do ln -sf "$PWD/${d%/}" ~/.claude/skills/; done
```

Restart Claude Code. Task Router shows up in the agent list; the skills load on their trigger phrases.

## Agents

Task Router reads a task, sweeps every row below, marks each IN or OUT with a reason, and dispatches the IN rows. Coverage and concurrency are the two things it is built to get right.

| Agent | Owns |
|---|---|
| **Task Router** | Routing, coverage sweep, plan approval loop, implementation handoff |
| **Code Craftsman** | Application code quality: Clean Architecture, DDD, SOLID, GoF patterns, concurrency |
| **Principal Frontend** | Frontend architecture, rendering strategy, state, Core Web Vitals, a11y, CSS, FE tests |
| **Database Manager** | Schema and migrations, ORM access patterns, indexing, query plans, transactions, scaling |
| **Principal Security** | Secure code review, threat modeling, authn/authz, crypto and secrets, supply chain |
| **Principal DevOps** | IaC, cloud architecture, Linux, git workflows, containers and Kubernetes |
| **Principal SRE/Platform** | SLI/SLO design, observability, rollout safety, incidents, capacity, platform engineering |
| **Principal QA** | Test strategy, automation architecture, non-functional testing, release readiness |
| **AWS Cloud Tester** | Live AWS account auditing through read-only CLI calls |
| **Bug Hunter** | Mechanical write-site/read-site tracing and interface-completeness checks |
| **Scope Sanity Checker** | Whether the work should exist at all, measured against the original ask |
| **Product Owner** | Backlog ownership, prioritization, user stories, acceptance criteria |
| **Ticket Creator** | Breaking work into PR-sized tickets with descriptions and acceptance criteria |
| **Team Communicator** | Slack, email, PR and ticket comments that read as human |
| **PR Comment Reviewer** | Watching an open PR for new reviewer comments and triaging each one |

Every agent is a single markdown file. Read it, disagree with it, edit it.

## Skills

| Skill | Use for |
|---|---|
| **stop-slop** | Stripping AI tells out of prose |
| **writing-good-comments** | When a code comment earns its place and what it must never contain |
| **software-engineering-patterns** | SOLID, CLEAN, DDD, DRY, KISS, YAGNI, commit and PR conventions |
| **design-patterns** | Picking and implementing the 22 Gang of Four patterns |
| **code-review-guru** | AWS-focused code review: Well-Architected, IAM least privilege, concurrency |
| **project-analyst** | Reading an unfamiliar codebase before writing anything in it |
| **diagram-generator** | Draw.io and Mermaid architecture diagrams |
| **react-doctor** | React lint, a11y, bundle and architecture triage |
| **refine-and-store** | Grilling a ticket or plan into shape, then persisting the result |
| **mobile-brief** | Phone-sized answers: short summary, then tappable options |

Task Router and its specialists call `stop-slop`, `writing-good-comments`, `software-engineering-patterns`, `design-patterns` and `code-review-guru` by name, so keep those installed if you install Task Router.

## Attribution

`skills/stop-slop` is third-party work by Hardik Pandya, vendored under MIT. Its license sits in the skill directory. Everything else is mine.

## Notes

Examples in these files use placeholder names (`PROJ-123`, `acme/web-app`, generic service names). Swap in your own conventions if you want the output to match your tracker.
