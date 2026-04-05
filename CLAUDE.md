# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo (`gha-shared`) provides **reusable GitHub Actions workflows** (callable via `workflow_call`) for KeenMate projects. Other repos reference these workflows to build and deploy Docker images.

## Architecture

All workflows live in `.github/workflows/` and are designed to be called from other repositories.

### Workflows

- **build-store-image.yml** — Builds a Docker image and pushes it to the private registry (`registry.km8.es`). Does NOT deploy.
- **deploy-docker-image.yml** — Calls `build-store-image.yml` to build and push, then SSHs into the deployment host to run `docker compose down/pull/up -d` in `/srv/docker/<deployment-dir>`. Requires additional SSH secrets (`BUILDER_SSH_USERNAME`, `BUILDER_SSH_KEY`, `BUILDER_SSH_PASSPHRASE`).

Both workflows:
- Use `workflow_call` trigger (reusable workflows, not standalone)
- Run on `self-hosted` runners
- Authenticate against `registry.km8.es` using org-level secrets
- Support configurable Docker build context, Dockerfile path, image name/tag
- Pass `MIX_ENV`, `NPM_SCRIPT`, and optionally `NPM_TOKEN`/`NUGET_*` as build args

`deploy-docker-image.yml` reuses `build-store-image.yml` (no duplication) and adds a separate deploy job.

## Conventions

- Workflow files are standard GitHub Actions YAML with `workflow_call` triggers.
- Image tags default to `latest`; callers can override via the `image-tag` input.
- Deployment target directories follow the pattern `/srv/docker/<name>` on the remote host.
