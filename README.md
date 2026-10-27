# Veridu for Claude Code

Independent review of the change Claude just made, before you commit. Ask Claude to "verify this change with Veridu": it sends the diff and the files it changed to Veridu, two reviewers from different vendors check them, findings that cannot be grounded in the code are dropped, and Claude gets back an outcome (pass, review or block) and concrete findings with file, line, severity and evidence. Claude fixes the findings that hold up and checks again; each finding keeps the same id across follow-ups, so you can see what was fixed, what persists and what a fix introduced.

Veridu reviews only what is sent. It does not read the rest of your repository, so a pass means "nothing found in what was sent", not "safe to merge". For pull-request review with repository context, use the [GitHub App](https://github.com/apps/veridu-review).

## What the plugin contains

- **An MCP server entry** for Veridu's hosted server at `https://api.veridu.io/mcp` (tools: `verify_change`, `get_change`, `veridu_review`, `veridu_review_status`).
- **A skill, `verify-before-commit`**, that tells Claude how to build the payload, read the result, and run the fix-and-check loop.

Nothing runs on your machine: there are no hooks, scripts or packages.

## Install

```
/plugin install veridu --marketplace michaelw-pr/veridu-plugin
```

Then run `/mcp`, choose **veridu**, and sign in in the browser (GitHub, Google or an emailed code). No API keys.

## Cursor

The same plugin installs in Cursor (`.cursor-plugin/plugin.json`): the Veridu MCP server at `https://api.veridu.io/mcp` and the verify-before-commit skill. Sign-in is OAuth in the browser the first time the server is used; there is no token to paste.

## VS Code (GitHub Copilot)

Veridu is a remote MCP server; nothing to install. Add it to `.vscode/mcp.json` in your project (or your user `mcp.json`):

```json
{
  "servers": {
    "veridu": {
      "type": "http",
      "url": "https://api.veridu.io/mcp"
    }
  }
}
```

Start the server from the file or the MCP servers list and sign in in the browser when asked (GitHub, Google or an emailed code). In Copilot agent mode, ask it to "verify this change with Veridu" before you commit: it calls `verify_change` with the diff and the changed files and gets back an outcome (pass, review or block) and findings with file, line, severity and evidence. To review an existing GitHub pull request, ask it to review the PR with Veridu (`veridu_review`).

## Cost

Free accounts include 20 reviews a month. Each new change uses one; up to two follow-up checks of the same change are free, and resending an identical change returns the earlier result free. Plans: https://veridu.io/pricing

## Privacy

When Claude calls `verify_change`, it sends to `api.veridu.io` the diff of the change and the full content of the files it chose to send. Nothing else from your machine or repository is read. The check runs in an isolated sandbox in AWS us-east-1, and the reviewer prompts go to OpenAI and Anthropic, neither of which uses API traffic for training under their published policies. The diff and files are deleted 7 days after the check finishes; the result (verdict and findings, which can quote lines of your code) is kept until you delete your account. Veridu stores only a hash of the sign-in token. Full policy: https://veridu.io/data. Questions or deletion requests: hello@veridu.io.

## Support

hello@veridu.io · https://veridu.io/docs

Veridu is operated by T3 Interactive LLC.
