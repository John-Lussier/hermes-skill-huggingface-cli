# Hermes skill: Hugging Face CLI

A public Hermes skill for using the Hugging Face `hf` CLI safely and effectively: auth, public/private Hub access, model/dataset/Space discovery, downloads, uploads, repos, cache, Jobs, Inference Endpoints, webhooks, and troubleshooting.

No secrets belong in this repo. The skill teaches agents to avoid printing Hugging Face tokens and to use `hf auth login`, `HF_TOKEN`, or `--token "$HF_TOKEN"` safely.

## Install in Hermes

Direct install:

```bash
hermes skills install https://raw.githubusercontent.com/John-Lussier/hermes-skill-huggingface-cli/v1.0.0/skills/huggingface-cli/SKILL.md --category mlops
```

Or add this repo as a tap:

```bash
hermes skills tap add John-Lussier/hermes-skill-huggingface-cli
hermes skills install John-Lussier/hermes-skill-huggingface-cli/huggingface-cli
```

Load explicitly in a session if needed:

```bash
hermes -s huggingface-cli
# or inside Hermes: /skill huggingface-cli
```

## Contents

- `skills/huggingface-cli/SKILL.md` — the installable Hermes skill.

## License

MIT
