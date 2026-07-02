---
name: huggingface-cli
description: "Use the Hugging Face `hf` CLI safely: auth, public/private Hub access, model/dataset/Space discovery, downloads, uploads, repos, cache, jobs, endpoints, webhooks, and troubleshooting. Load when the task names Hugging Face, HF Hub, `hf`, `huggingface-cli`, models/datasets/Spaces on the Hub, or Hub auth/tokens."
version: 1.0.3
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [huggingface, hf, huggingface-hub, models, datasets, spaces, mlops, cli]
    homepage: https://huggingface.co/docs/huggingface_hub/en/guides/cli
---

# Hugging Face CLI (`hf`)

Use this when interacting with Hugging Face Hub from a terminal: model/dataset/Space discovery, authenticated private/gated access, repo/file management, uploads/downloads, cache cleanup, Spaces, Jobs, Inference Endpoints, webhooks, buckets, and collections.

The modern command is `hf`. `huggingface-cli` may exist as an older/deprecated alias; prefer `hf` and run `hf --help` on the target machine because subcommands evolve quickly.

## What is the Hugging Face Hub?

The Hub is Hugging Face's AI collaboration platform: Git-backed repositories for models, datasets, and Spaces, plus Storage Buckets for mutable object storage. It also includes org/team access, tokens, discussions and PRs, collections, webhooks, Jobs, Inference Endpoints, and security/compliance features. Use the Hub docs for product concepts and the CLI docs for exact command syntax:

- Hub docs: https://huggingface.co/docs/hub/en/index
- CLI docs: https://huggingface.co/docs/huggingface_hub/en/guides/cli

## First moves

1. Check the CLI, version, and real command surface:

```bash
command -v hf || command -v huggingface-cli
hf version
hf --help
```

2. Check auth without exposing tokens:

```bash
hf auth whoami --json
```

If that fails, decide whether auth is actually needed. Public search/info/download usually works logged out. Private repos, gated repos, uploads, repo creation/settings, Spaces secrets, Jobs, Endpoints, and webhooks require a token.

3. For automation, prefer machine-readable output:

```bash
hf models ls --search llama --limit 5 --json
hf datasets info HuggingFaceFW/fineweb --json
hf spaces search "leaderboard" --limit 5 --json
```

Use `--format json` where `--json` is not available; use `-q` only when one-ID-per-line output is enough. Authenticated inventory such as `hf repos ls --json` requires a token.

## Install or update

Prefer an existing `hf`. If it is missing, do not bootstrap by piping remote installers into a shell from inside an agent session. Send the user to the official CLI docs or use an environment-approved package manager/install path they already trust: https://huggingface.co/docs/huggingface_hub/en/guides/cli

After installation, restart the shell if PATH changed and verify:

```bash
hf version
hf --help
```

Use `hf update` for the standalone CLI when available.

## Authentication: logged in, not logged in, and no secrets

Token source: https://huggingface.co/settings/tokens. Use the least privilege token that fits the task: read for private/gated downloads, write for upload/repo changes. Some paid/admin operations need additional account/org permissions.

Safe auth ladder:

```bash
# 1. Safe status check. This prints identity, not the token.
hf auth whoami --json

# 2. If already logged in but wrong token/account:
hf auth list
hf auth switch

# 3. If not logged in and the terminal is interactive:
hf auth login

# 4. If an env token is already available, do not print it:
hf auth login --token "$HF_TOKEN"

# 5. If using raw git/git-lfs against Hub repos, also store in git credential helper:
hf auth login --token "$HF_TOKEN" --add-to-git-credential

# 6. Use a token for one command without storing it:
hf download private-org/private-model --token "$HF_TOKEN"
```

Never run `hf auth token` unless the user explicitly asks to inspect their own token locally. Never print, paste, log, commit, or put a literal token in shell history. Do not use `set -x` around token commands. Do not read Hugging Face token files. If the user is not logged in and no `HF_TOKEN` exists, ask them to run `hf auth login` locally or provide credentials via their normal secret manager/environment, not in chat.

Not logged in is fine for:

```bash
hf models ls --search "text-generation" --limit 10
hf models info gpt2
hf download gpt2 config.json --local-dir ./gpt2
hf datasets ls --search "code" --limit 10
hf spaces search "image generation" --limit 10
```

If auth fails:

- `401` or “not logged in”: login or pass `--token "$HF_TOKEN"`.
- `403` on a gated model/dataset: the user may need to accept the model terms in a browser, or the token lacks permission.
- `404` on a known repo: check `--repo-type` (`model`, `dataset`, `space`) and whether the repo is private.

## Discovery and inspection

Models:

```bash
hf models ls --search "llama" --author meta-llama --limit 10 --json
hf models ls --sort downloads --limit 10
hf models info meta-llama/Llama-3.2-1B-Instruct --json
hf models card google/gemma-4-31B-it
hf models ls meta-llama/Llama-3.2-1B-Instruct -R --tree -h
hf models info Qwen/Qwen3.5-9B --expand downloads,likes,tags,sha --json
```

Datasets:

```bash
hf datasets ls --search "code" --limit 10 --json
hf datasets info HuggingFaceFW/fineweb --json
hf datasets card HuggingFaceFW/fineweb
hf datasets ls HuggingFaceFW/fineweb -R --tree -h
hf datasets parquet cfahlgren1/hub-stats --format json
hf datasets sql "SELECT COUNT(*) AS rows FROM read_parquet('https://huggingface.co/api/datasets/cfahlgren1/hub-stats/parquet/models/train/0.parquet')" --json
```

`hf datasets sql` needs DuckDB available as either the Python package or CLI binary; if it errors, install DuckDB through the user's approved package manager before retrying.

Spaces:

```bash
hf spaces search "generate image" --limit 10 --json
hf spaces info username/space-name --json
hf spaces card mteb/leaderboard
hf spaces hardware
hf spaces logs username/my-space --tail 100
hf spaces logs username/my-space --build --tail 100
```

Repos you can access:

```bash
hf repos ls --limit 30 --json
hf repos ls --namespace my-org --repo-type model --search bert --json
```

## Downloading

Pin revisions when reproducibility matters. Start with `--dry-run`, `--include`, and `--exclude` to avoid accidental multi-GB downloads.

```bash
# Whole model into cache; prints local cache path.
hf download meta-llama/Llama-3.2-1B-Instruct

# Specific files.
hf download meta-llama/Llama-3.2-1B-Instruct config.json tokenizer.json

# Pattern-select weights, skip older formats.
hf download meta-llama/Llama-3.2-1B-Instruct --include "*.safetensors" --exclude "*.bin" --dry-run
hf download meta-llama/Llama-3.2-1B-Instruct --include "*.safetensors" --exclude "*.bin" --local-dir ./models/llama

# Dataset or Space.
hf download HuggingFaceM4/FineVision art/ --repo-type dataset --local-dir ./FineVision
hf download username/my-space --repo-type space --local-dir ./space

# Reproducible revision and custom cache.
hf download org/repo --revision <commit-or-tag> --cache-dir ./hf-cache --local-dir ./repo
```

Use `--token "$HF_TOKEN"` for private/gated repos if not logged in. For very large downloads, prefer patterns and a durable `--local-dir`; reruns reuse cached files.

## Uploading and repo file changes

Use `hf upload` for normal single-commit uploads; use `hf upload-large-folder` for large/resumable directories. For risky changes, use `--create-pr` so humans can review on the Hub.

```bash
# Upload current directory to an existing/new model repo.
hf upload username/my-cool-model . . --commit-message "Upload model"

# Upload one file.
hf upload username/my-cool-model ./models/model.safetensors model.safetensors

# Dataset upload.
hf upload username/my-cool-dataset ./data /train --repo-type dataset --commit-message "Upload train split"

# Include/exclude and delete in the same commit.
hf upload username/my-cool-model . . --include "*.safetensors" --exclude "*.bin" --delete "old/*" --commit-message "Refresh weights"

# Safer contribution path.
hf upload bigcode/the-stack . . --repo-type dataset --create-pr

# Large/resumable folder.
hf upload-large-folder username/my-cool-model ./large_model_dir --include "*.safetensors"
```

Single-object copy works with `hf://` URIs:

```bash
hf cp ./model.safetensors hf://username/my-model/model.safetensors
hf cp hf://username/my-model/config.json ./config.json
hf cp hf://datasets/username/my-dataset/data.csv ./data/
hf cp - hf://buckets/username/my-bucket/config.json
```

Delete remote files with a commit, preferably a PR when destructive:

```bash
hf repos delete-files username/my-model "*.bin" --create-pr --commit-message "Remove legacy bin weights"
```

## Repo management

Default repo type is `model`. Always pass `--repo-type dataset` or `--repo-type space` when not managing a model.

```bash
# Create repos.
hf repos create username/my-model --public --exist-ok
hf repos create username/my-dataset --repo-type dataset --private --exist-ok
hf repos create username/my-space --repo-type space --space-sdk gradio --public --exist-ok

# Settings.
hf repos settings username/my-model --private
hf repos settings username/my-model --public
hf repos settings username/my-model --gated auto
hf repos settings username/my-space --repo-type space --protected

# Branches/tags.
hf repos branch create username/my-model dev
hf repos branch delete username/my-model dev
hf repos tag list username/my-model
hf repos tag create username/my-model v1.0
hf repos tag delete username/my-model v1.0

# Duplicate or move.
hf repos duplicate openai/gdpval --repo-type dataset
hf repos move old-namespace/my-model new-namespace/my-model

# Delete: irreversible. Only do this with explicit user approval.
hf repos delete username/my-model --repo-type model
```

`hf repo ...` is deprecated; use `hf repos ...`.

## Spaces operations

Use variables for non-secret config and secrets for tokens/passwords. Never put secret values in README, committed files, or command output. Prefer env-var names when the CLI supports them.

```bash
# Logs and lifecycle.
hf spaces logs username/my-space --tail 100
hf spaces logs username/my-space --build --tail 100
hf spaces restart username/my-space
hf spaces pause username/my-space

# Settings/hardware.
hf spaces hardware
hf spaces settings username/my-space --hardware t4-medium
hf spaces settings username/my-space --sleep-time 300

# Variables vs secrets.
hf spaces variables add username/my-space -e DEBUG=1
hf spaces variables ls username/my-space
hf spaces variables delete username/my-space DEBUG

hf spaces secrets add username/my-space -s HF_TOKEN      # pass from local env when supported
hf spaces secrets ls username/my-space                   # lists names/metadata, not values
hf spaces secrets delete username/my-space HF_TOKEN

# Development helpers.
hf spaces dev-mode username/my-space
hf spaces hot-reload username/my-space app.py
hf spaces ssh username/my-space
```

## Jobs and Inference Endpoints

Jobs run commands on Hugging Face infrastructure. Verify cost/hardware before launching long jobs.

```bash
hf jobs hardware
hf jobs run python:3.12 python -c 'print("Hello from HF Jobs")'
hf jobs uv --help
hf jobs ps --json
hf jobs inspect <job_id> --json
hf jobs logs <job_id>
hf jobs stats <job_id>
hf jobs cancel <job_id>
hf jobs scheduled --help
```

UV-script jobs use the `run` subcommand under `hf jobs uv`; review the script and inline dependencies before launching because jobs run on Hugging Face infrastructure and can incur cost.

Inference Endpoints are managed services and can incur cost. List/describe before changing; pause/scale-to-zero when idle.

```bash
hf endpoints ls --json
hf endpoints catalog list --json
hf endpoints catalog deploy --repo gpt2 --name my-endpoint
hf endpoints deploy my-endpoint --repo gpt2 --framework pytorch --accelerator cpu --instance-size x4 --instance-type intel-icl --region us-east-1 --vendor aws
hf endpoints describe my-endpoint --json
hf endpoints update my-endpoint --min-replica 2
hf endpoints pause my-endpoint
hf endpoints resume my-endpoint
hf endpoints scale-to-zero my-endpoint
hf endpoints delete my-endpoint       # destructive/cost-affecting; require approval
```

## Buckets, sync, webhooks, discussions, collections, extensions, skills

Storage buckets and sync:

```bash
hf buckets --help
hf sync ./local-dir hf://buckets/username/my-bucket/path
hf cp hf://buckets/username/my-bucket/config.json -
```

Discussions and PRs:

```bash
hf discussions list username/my-model
hf discussions create username/my-model --title "Fix config" --body "..."
hf discussions diff username/my-model <discussion_or_pr_num>
hf discussions comment username/my-model <num> --body "Looks good"
hf discussions merge username/my-model <pr_num>
```

Webhooks:

```bash
hf webhooks ls --json
hf webhooks create --url https://example.com/hook --watch model:bert-base-uncased --domain repo
hf webhooks info <webhook_id> --json
hf webhooks disable <webhook_id>
hf webhooks enable <webhook_id>
hf webhooks delete <webhook_id>
```

Collections:

```bash
hf collections ls --json
hf collections create "My Models"
hf collections add-item username/my-collection moonshotai/kimi-k2 model
hf collections info username/my-collection-slug --json
hf collections update username/my-collection --title "New Title"
```

Extensions are third-party executables/Python packages. Install only from sources the user trusts:

```bash
hf extensions search
hf extensions install <trusted-public-github-repo>
hf extensions ls
hf extensions exec <name> -- --help
hf extensions remove <name>
```

`hf skills` installs Hugging Face marketplace skills for some AI assistants. Hermes agents normally use `hermes skills install ...`; do not use `hf skills add` for Hermes unless the user explicitly asks for Hugging Face's marketplace skill flow.

## Cache, environment, and troubleshooting

```bash
hf env
hf version
hf cache ls
hf cache verify gpt2
hf cache prune
hf cache rm model/gpt2       # destructive local cache cleanup; confirm target
```

Useful environment variables:

- `HF_TOKEN`: token for noninteractive commands; do not print it.
- `HF_HOME`: root for Hugging Face local state/cache.
- `HF_HUB_CACHE`: Hub cache directory.
- `HF_ENDPOINT`: alternate Hub endpoint/mirror.
- `HF_HUB_ENABLE_HF_TRANSFER=1`: optional faster transfer path when supported/installed.

Common failures:

- Command missing: install/update, restart shell, check PATH.
- `huggingface-cli` works but `hf` does not: update `huggingface_hub` or install standalone CLI; prefer `hf` once available.
- 401/403: auth/token/permission/gated terms issue; verify with `hf auth whoami --json` and browser terms acceptance.
- 404: wrong repo ID, wrong `--repo-type`, private repo without token, or typo in namespace.
- Huge accidental download: cancel, rerun with `--dry-run`, `--include`, `--exclude`, and `--local-dir`.
- Upload rejected: token lacks write permission, repo type wrong, file too large for normal upload; use `upload-large-folder` or Git LFS workflow.
- Space stuck building: inspect `hf spaces logs <space> --build --tail 200`, then run logs.

## Verification checklist

After making changes, verify the actual artifact:

- Auth: `hf auth whoami --json` succeeds or the task is confirmed public/read-only.
- Discovery/download: record repo ID, repo type, revision/sha if reproducibility matters, and local path from `hf download`.
- Upload/repo change: confirm with `hf models info`, `hf datasets info`, `hf spaces info`, `hf repos ls`, or file listing (`hf models ls REPO_ID -R --tree`).
- Spaces: check `hf spaces info`, `hf spaces logs --tail 100`, and public/private visibility as requested.
- Jobs/endpoints: capture job/endpoint ID and status; pause/scale down when the user asked or cost risk is obvious.
- Secrets: ensure no tokens were printed, committed, or included in shared files.
