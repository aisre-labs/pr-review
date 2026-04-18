# Testing the extension

Three levels of testing are available, from fastest to most realistic.

## Level 1 — Unit tests (already passing, 145/145)

```bash
cd extension
npm install
npm test
```

Mocked HTTP; runs in ~1 second. Already covers: SRE keyword filter, thread grouping,
ADO URL construction, AI JSON parsing, scope classification, cosine similarity, glob
matching, rule deduplication, idempotency hashing.

## Level 2 — Integration smoke test (5 seconds, zero setup)

```bash
cd extension
npm run smoke
```

Exercises the full ingest + review flow end-to-end with mocked ADO REST + Azure OpenAI.
Prints a readable report showing which classes called which mocks and the final posted
comment. See `scripts/smokeTest.ts`.

**What it proves:**
- `mode=ingest`: #best-practice detection → rule extraction → embedding
- `mode=review`: rule matching → AI review → post inline comment → idempotency
- Each class's contract is honored (types match, prompts pass through, responses parse)

**What it doesn't prove:**
- Whether real Azure OpenAI returns sensible responses to the prompts (see Level 3)
- Whether ADO accepts the POST `/threads` body (see Level 4)

## Level 3 — Prompt smoke test with real Azure OpenAI (optional)

Requires: Azure OpenAI deployment with `gpt-4o-mini` + `text-embedding-3-small`.

```bash
export AOAI_ENDPOINT=https://your-org.openai.azure.com
export AOAI_KEY=your-api-key
export AOAI_CHAT=gpt-4o-mini
export AOAI_EMBED=text-embedding-3-small
# (not implemented yet — would duplicate smokeTest.ts but with real AiClient)
```

This would validate:
- Prompts return valid JSON (Azure OpenAI's JSON mode is reliable)
- Rule extraction produces sensible scope classifications
- PR review produces useful comments

## Level 4 — Full E2E in a real Azure DevOps organization

The only way to verify the extension actually installs and a pipeline runs the task.

### Step-by-step

**1. Register a publisher** (free, 5 min)
- Go to https://marketplace.visualstudio.com/manage/publishers/
- Sign in with any Microsoft Account
- Click "+ New Publisher"
- Enter Publisher ID (e.g., `aisre-test`) and Display Name
- **Record the Publisher ID** — it's embedded in every extension version

**2. Update manifests with your publisher ID:**

```bash
cd extension
npm run set-publisher <your-publisher-id>
```

This replaces `REPLACE_WITH_PUBLISHER_ID` in `vss-extension.json`, `task.json`, and `LICENSE`.

**3. Build the .vsix:**

```bash
npm run package
# → dist/<publisher>.pr-review-0.1.0.vsix
```

**4. Get a personal access token** (for upload)
- https://dev.azure.com/<your-org>/_usersSettings/tokens
- Scope: Marketplace (Manage)
- Copy the token

**5. Upload as private (shared with your test ADO org only):**

```bash
export PUBLISHER_PAT=your-pat
# in vss-extension.json ensure "public": false
npx tfx extension publish \
  --vsix dist/*.vsix \
  --share-with <your-ado-org-name> \
  --token $PUBLISHER_PAT
```

**6. Install in test ADO org:**
- Go to https://marketplace.visualstudio.com/manage
- Click your extension → Share → verify target org listed
- In ADO org: Organization Settings → Extensions → Shared → Install

**7. Configure Azure OpenAI in Settings Hub:**
- ADO → Project Settings → PR Review Learning
- Fill Azure OpenAI endpoint + key + deployment names → Save

**8. Add pipeline task to a test repo:**

```yaml
# azure-pipelines.yml
trigger: none
pr:
  branches:
    include: [main]

stages:
  - stage: AIReview
    jobs:
      - job: Review
        pool: { vmImage: ubuntu-latest }
        steps:
          - task: PrReviewLearning@1
            inputs:
              mode: review
              dryRun: true   # start with dry-run for first test
```

**9. Create a test PR** targeting `main`
- Pipeline triggers automatically
- Task logs should show AI review activity
- With `dryRun: true`, no comments posted — check task output
- Flip to `dryRun: false` once you're confident

**10. Test learning mode** (on merged PR):
- After merge, run a separate pipeline with `mode: ingest`
- Check Settings Hub → Project groups to see if rules appeared
  (Note: rules UI not yet built — Phase 9 enhancement; rules are stored in
  Extension Data Service and can be inspected via REST)

### Expected costs during test
- Publisher account: **$0**
- ADO org: **$0** (free tier, unlimited private repos)
- Azure OpenAI per test run: ~$0.01 (a few hundred tokens)
- Total test cost: <$1 for extensive testing

## Troubleshooting

**Task fails with "Extension not configured":**
Open Project Settings → PR Review Learning → Azure OpenAI tab, fill in endpoint + key.

**Task logs show 401 Unauthorized (ADO):**
Pipeline's Build Service identity needs `vso.code_write` scope. This is declared in
`vss-extension.json` and granted automatically on install.

**Task logs show 404 on /threads:**
Either pullRequestId is empty (not a PR build) or the repo/project is wrong. Check
`System.PullRequest.PullRequestId` is set.

**AI returns non-JSON:**
Some model versions handle `response_format: json_object` differently. Currently
tested on `gpt-4o-mini` and `gpt-4o`. If issues, override `deploymentOverride` to a
different model in `aiClient.chat()`.
