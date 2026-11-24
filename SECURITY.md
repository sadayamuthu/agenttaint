# Security Policy

## Reporting a vulnerability

Please don't report security problems in a public issue, pull request or
discussion. Use
[**Report a vulnerability**](https://github.com/sadayamuthu/agenttaint/security/advisories/new)
on the repository's Security tab instead. Only the maintainers can see the
report.

Include what you can of:

- the AgentTaint version (`agenttaint version`) or commit, your Go version and OS
- how AgentTaint was used, and its configuration with secrets removed
- steps to reproduce
- what happened and what you expected

You'll get a reply when the report has been read, and again when a fix is
ready. A fix ships as a patch release; the advisory names you unless you'd
rather it didn't.

## Supported versions

AgentTaint is at the design stage and has no releases yet. Once it does,
fixes land in the latest release only, until 1.0.

## What counts

AgentTaint sits on an agent's tool path, labels content by source, and allows, blocks or escalates each tool call by where its arguments came from.

AgentTaint is a security tool, so anything that breaks one of these promises
is a security issue:

- **Conservative propagation.** A value derived from untrusted data that reaches a sink labelled trusted is in scope.
- **Blocks are enforced.** A call the sink policy blocks that runs anyway is in scope.
- **Flow logs go only where you configure.** A path that sends them elsewhere is in scope.

These are by design, and not vulnerabilities:

- In proxy mode AgentTaint sees only what crosses the tool boundary; values the model computes from untrusted text can escape matching. See the open research questions in the README.
- Flow logs contain the labelled content.

A wrong result that isn't covered above is a bug, not a vulnerability. Report
it in a normal issue; see [CONTRIBUTING.md](CONTRIBUTING.md).
