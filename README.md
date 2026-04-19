# AI PR Review

> Pull request reviews that learn from your team — not from generic internet advice.

**AI PR Review** is a free Azure DevOps extension that automatically reviews every pull request using patterns your team already established. It reads resolved PR comments from past merges, extracts the rules your reviewers actually enforced, and applies them to every new PR — inline, like a human reviewer.

📦 **Install from Marketplace:** [marketplace.visualstudio.com/items?itemName=AISRE.pr-review](https://marketplace.visualstudio.com/items?itemName=AISRE.pr-review)

![AI review comments inline on a pull request](docs/screenshots/01-ai-comments-on-pr.png)

---

## How it works

```
  Past PR (merged)              New PR (opened)
  ─────────────────             ─────────────────
  Reviewer: "Always use         PrReviewLearning@1 runs,
  HikariCP here. See our        matches learned rule →
  #best-practice guide."        posts inline comment:
  Dev fixes, merges.            "⚠ Use HikariCP — your
                                team's rule (4 observations)"
        ↓
  ingest task learns rule,
  stores it in your ADO org
```

1. Add `PrReviewLearning@1` (mode: **review**) to your PR validation pipeline — one task, zero config beyond that.
2. Add the same task (mode: **ingest**) to your post-merge pipeline to learn new rules automatically.
3. Configure your Azure OpenAI endpoint once in Project Settings → PR Review.

That's it. No servers to operate, no data leaving your Azure tenant.

---

## Features

- **Inline comments on every PR** — AI findings appear directly on the changed lines
- **Learns from your history** — rules come from your team's own resolved comments, not internet opinions
- **`#best-practice` shortcut** — tag any comment to immediately create a rule, without waiting for the AI to classify it
- **Project groups** — one rule can apply across a set of related repositories (e.g. all payment-service repos)
- **Three review modes** — full AI review, rule-based only, or disabled — choose per pipeline
- **Four learning modes** — strict (default), tagged-only, conversation, all
- **Rule approval workflow** — non-tagged rules require admin approval in Hub before affecting PRs
- **Rule provenance & audit log** — every rule shows its source PR comments and full change history
- **Feedback-loop protection** — ingest skips AI-posted comments to prevent self-learning
- **API key encrypted at rest** — AES-GCM with per-organization key
- **Your data, your tenant** — BYOK Azure OpenAI, storage in ADO Extension Data Service, zero external dependencies
- **Free** — you pay only for your own Azure OpenAI token usage (~$5–50 / month typical)

![Learned rules panel — team standards captured from PR history](docs/screenshots/02-learned-rules-panel.png)

---

## Requirements

| Requirement | Notes |
|---|---|
| Azure DevOps organization | Cloud or on-prem |
| Azure OpenAI resource | Chat model (`gpt-4o-mini` recommended) + embedding model (`text-embedding-3-small`) |
| Azure Pipelines | For the pipeline task |

---

## Installation

1. Install the extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AISRE.pr-review) *(link active after public release)*.
2. Open **Project Settings → PR Review** and enter your Azure OpenAI endpoint and API key.
3. Add the tasks to your pipelines (see below).

---

## Pipeline setup

### 1. Review pipeline — runs on every PR

Save as `ai-review-pipeline.yml` in your repo root, register it as a pipeline in Azure DevOps, then set it as a **Branch Policy → Build Validation** on `main` (set as **Optional** to give feedback without blocking merge):

```yaml
trigger: none

pr:
  branches:
    include: [main]
  paths:
    exclude:
      - '**/*.md'
      - docs/*

pool:
  vmImage: 'ubuntu-latest'

jobs:
  - job: Review
    timeoutInMinutes: 5
    steps:
      - task: PrReviewLearning@1
        inputs:
          mode: review
```

### 2. Ingest pipeline — learns rules from merged PRs

Save as `ai-ingest-pipeline.yml`. Run **manually** after merging a PR, passing the PR number as a parameter. The plugin will analyze the PR's resolved comments and capture them as rules for all future PRs:

```yaml
trigger: none

parameters:
  - name: pullRequestId
    displayName: 'Pull Request ID to ingest'
    type: string
    default: ''

pool:
  vmImage: 'ubuntu-latest'

jobs:
  - job: IngestRules
    timeoutInMinutes: 10
    steps:
      - bash: |
          echo "##vso[task.setvariable variable=System.PullRequest.PullRequestId]${{ parameters.pullRequestId }}"
        condition: ne('${{ parameters.pullRequestId }}', '')

      - task: PrReviewLearning@1
        inputs:
          mode: ingest
```

**Why manual trigger?** ADO doesn't set `System.PullRequest.PullRequestId` on CI builds triggered by merge — only on PR builds. Running ingest manually with the PR number is the simplest way to control which PRs the plugin learns from. Run it right after merging a PR where a reviewer's comment should become a rule.

---

## How to register the pipelines in Azure DevOps

After saving the YAML files in your repo, each one needs to be registered as a pipeline in ADO:

1. **Pipelines → New pipeline**
2. **Azure Repos Git** → pick your repo
3. **Existing Azure Pipelines YAML file** → select `/ai-review-pipeline.yml` (or `/ai-ingest-pipeline.yml`)
4. **Save** (don't run yet)
5. Optionally rename (three-dot menu → Rename), e.g. `myrepo-ai-review` and `myrepo-ai-ingest`

### Set review pipeline as Branch Policy (one-time)

So the review runs on every PR targeting `main`:

1. **Project Settings → Repositories → [your repo] → Policies**
2. Under **Branch Policies**, click on `main` (or your target branch)
3. **Build Validation → Add**
4. Build pipeline: select `myrepo-ai-review`
5. Trigger: **Automatic**
6. Policy requirement: **Optional** (recommended — AI review is advisory, not blocking). Pick **Required** only once you trust its precision.
7. **Save**

---

## When does each mode actually run?

### Review — automatic, every PR

Once the Branch Policy is set up:

- Dev opens a PR targeting the protected branch → **review pipeline triggers automatically**
- AI analyzes the diff, applies learned rules, posts inline comments on the PR
- Dev pushes new commits → pipeline re-runs; already-posted comments are not duplicated (idempotent by file+line+rule hash)
- Nothing manual — set it once, forget it.

### Ingest — manual, after merge

The ingest pipeline requires a **pullRequestId parameter**, so it's run on demand:

1. Reviewer leaves a comment on a PR (optionally tagged `#best-practice` to skip AI classification and make it a rule immediately)
2. Author fixes and merges the PR → note the PR number (e.g. `#42`)
3. In ADO: **Pipelines → `myrepo-ai-ingest` → Run pipeline**
4. In the "Run pipeline" dialog, paste the PR number into **Pull Request ID to ingest**
5. **Run**
6. Plugin analyzes the PR's resolved comments and captures them as rules; from the next PR onwards, those rules apply automatically

**Not every merge needs an ingest run** — only the ones where a reviewer caught something worth remembering as a rule.

---

## Task inputs

| Input | Default | Description |
|---|---|---|
| `mode` | — | **Required.** `review` — review a new PR. `ingest` — learn rules from a merged PR. |
| `reviewMode` | `full` | `full` — AI general + rules. `rules-only` — matched rules only (cheaper). `disabled` — skip review. |
| `maxComments` | `20` | Hard cap on comments posted per run. |
| `minConfidence` | `0.5` | Rules below this threshold are ignored. |
| `dryRun` | `false` | Log findings to build output instead of posting to the PR. |

---

## The `#best-practice` tag

If a reviewer writes a comment containing `#best-practice`, the ingest task classifies it as **immediately addressable** — no AI resolution analysis needed. The rule is extracted directly from the comment text and stored with full confidence. This is the fastest path from "we agreed on something in a PR" to "it's automatically enforced on the next PR".

---

## Project Groups — scope rules across related repos

By default, a rule learned in one repository applies **only to that repository**. If your team works across multiple repos that share conventions, you can group them so a rule learned in one applies to all.

### When do you need it?

- You have microservices split across multiple repos that share coding standards (`payment-api`, `payment-worker`, `payment-gateway`)
- A reviewer notes *"we always use HikariCP for DB pooling"* in one of them — without project groups, this rule applies only to that one repo. With a `payments` group, it applies to all three.
- Not needed if you have one repo or each repo has different standards.

### How to configure

1. In ADO: **Project Settings → AI Code Reviewer → Project groups**
2. Click **+ New group**
3. Fill in:
   - **Name** — e.g. `payment-services`
   - **Description** (optional) — e.g. *"All services for the payments domain"*
   - **Repositories** (one per line, format `org/project/repo`):
     ```
     myorg/MyProject/payment-api
     myorg/MyProject/payment-worker
     myorg/MyProject/payment-gateway
     ```
4. **Save**

From the next ingest run onwards, the AI classifier sees your groups and can scope a rule to a group instead of just one repo. Add a new repo to the group later → all group-scoped rules apply to it automatically, no re-ingest needed.

### ⚠ Pipelines are still per-repo

A project group shares **rules** across repos, not pipelines. Each repo in the group still needs:

- Its own `ai-review-pipeline.yml` in the repo root
- Its own pipeline registration in ADO (one-time, via *New pipeline* wizard)
- Its own Branch Policy → Build Validation set on `main`

Same for ingest — each repo needs its own `ai-ingest-pipeline.yml` to run ingest on a merged PR from that repo. The group controls how rules **match**; the pipelines control where reviews **run**.

A practical setup for a 3-repo group: 6 pipelines total (review + ingest × 3 repos), all sharing the same learned rules store via the extension's per-organization data service.

### Scope hierarchy

The AI picks the **narrowest** scope that fits each rule:

| Scope | Matches | Example pattern |
|---|---|---|
| `DIRECTORY` | Files matching a glob | `**/*Repository.java` |
| `PROJECT` | One specific repo | `myorg/myproj/payment-api` |
| `PROJECT_GROUP` | Group of repos (you config) | `payment-services` |
| `LANGUAGE` | Any file of that language | `java`, `typescript` |

You can edit any rule's scope later in the **Learned Rules** tab if AI classified it too narrow or too broad.

---

## Enterprise governance — learning mode, provenance, audit log

### Learning mode — control which comments become rules

Set in **Project Settings → AI Code Reviewer → Azure OpenAI → Learning mode**. Four strictness levels:

| Mode | What becomes a rule | Non-tagged rule scope | Recommended for |
|---|---|---|---|
| **Strict** *(default)* | Tagged + threads marked **Resolved** AND where a commit changed the commented line | Forced `PROJECT` | Everyone — safest |
| **Tagged-only** | Only `#best-practice` / `#best-global-practice` tagged | n/a | Maximum precision |
| **Conversation** | Strict + AI analyzes replies to classify intent | Forced `PROJECT` | Higher recall, some AI risk |
| **All** | Tagged + AI replies + diff-heuristic without Resolved gate | AI-picked (any scope) | Legacy — accepts false positives |

Default is **Strict**. Non-tagged rules require both explicit human signals:
1. Thread marked Resolved in PR UI
2. Commit actually changed the commented line

Non-tagged rules are always `PROJECT` scope — they can't accidentally become cross-repo policies. To make a global rule, use `#best-global-practice` tag explicitly.

### Rule provenance — click Details to see source PRs

Every rule in **Learned Rules** tab has a **Details** button. Expanding shows:

- **Provenance** — the exact PR comments that created or reinforced this rule (reviewer, date, file path, link to PR)
- **Audit log** — chronological history of every change to this rule

### Audit log — who changed what

Every state change is recorded:

- **Actor** — `system:ingest (PR #42)` or `user:<display name>`
- **Action** — created / merged / confidence_changed / active_toggled / edited / deleted
- **Before/after** — only changed fields (compact footprint)
- **Note** — human-readable description

Stored sharded (~100 events per document) in Extension Data Service. Zero external infrastructure.

---

## Privacy & security

| What | Where it goes |
|---|---|
| Source code / diffs | Your Azure tenant only |
| PR comments | Your Azure tenant only |
| Azure OpenAI API key | Encrypted in Extension Data Service (your org) |
| AI prompts & responses | Between your pipeline agent and your Azure OpenAI resource |

**AISRE Labs never sees your code, your diffs, or your PR comments.** The pipeline task runs as a standard Azure Pipelines job authenticated by `$(System.AccessToken)`. All network calls go to `dev.azure.com/{your-org}/…` and `{your-resource}.openai.azure.com/…`.

Full details: [aisre-labs.github.io/pr-review/privacy](https://aisre-labs.github.io/pr-review/privacy)

---

## Cost

| Item | Who pays | Typical cost |
|---|---|---|
| Extension | — | **Free** |
| Extension Data Service (rule storage) | — | **Free** (built into ADO) |
| Pipeline minutes | You | Covered by ADO free tier for most orgs |
| Azure OpenAI tokens | You | $5–50 / month for ~100 PRs/month (`gpt-4o-mini`) |

---

## Support & feedback

- **Bug reports / feature requests:** [GitHub Issues](https://github.com/aisre-labs/pr-review/issues)
- **Security vulnerabilities:** privacy@aisre-labs.dev (do not open a public issue)

---

## License

Proprietary — see [LICENSE](https://github.com/aisre-labs/pr-review/blob/main/LICENSE).  
AI prompt templates are trade secrets of AISRE Labs. Reverse engineering is prohibited.
