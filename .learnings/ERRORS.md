# Errors

## [ERR-20260725-001] git-ls-remote

**Logged**: 2026-07-25
**Priority**: medium
**Status**: resolved
**Area**: infra

### Summary

GitHub SSH connection on port 22 was closed while checking the remote repository.

### Error

```text
Connection closed by 198.18.0.17 port 22
fatal: Could not read from remote repository.
```

### Context

- Command: `git ls-remote --heads git@github.com:willwang2528/ai-knowledge.git`
- Repository: `/Users/will/knowledge/knowledge-agent`
- The local repository is on `main` with one root commit.

### Suggested Fix

Use GitHub's SSH endpoint on port 443 for remote inspection and push, while preserving the configured `origin` URL requested by the user.

### Metadata

- Reproducible: yes
- Related Files: `.git/config`

### Resolution

- **Resolved**: 2026-07-25
- **Notes**: Verified the `ssh.github.com:443` ED25519 fingerprint against GitHub's official documentation, authenticated successfully as `willwang2528`, and configured the repository-local SSH command to use port 443.

---

## [ERR-20260725-002] shell-verification-quoting

**Logged**: 2026-07-25
**Priority**: low
**Status**: resolved
**Area**: docs

### Summary

Backticks inside a double-quoted `rg` pattern were interpreted by the shell as command substitutions.

### Error

```text
zsh: command not found: discussion.md
zsh: command not found: concepts.md
zsh: command not found: references.md
```

### Context

- The command was only verifying Markdown wording.
- Project files and Git state were not modified by the failed command.

### Suggested Fix

Use single quotes around search patterns containing Markdown backticks.

### Metadata

- Reproducible: yes
- Related Files: `AGENTS.md`

### Resolution

- **Resolved**: 2026-07-25
- **Notes**: Re-ran the check with shell-safe quoting.

---
