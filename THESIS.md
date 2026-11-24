# AgentTaint: thesis

**In one line:** stop judging whether text looks like an injection; track
where each value came from and refuse to let untrusted data drive sensitive
actions.

## The problem

An agent's context is one stream of text, so a malicious web page and the
user's request look the same to the model. Filters that try to spot
injection-looking text are an arms race that defenders lose.

## The bet

1. **Information-flow control transfers to agents.** Labelling context by
   source and checking labels at sinks (shell commands, URLs, recipients) is
   the classic fix for this class of problem, and CaMeL (Google DeepMind,
   2025) showed it works for agents.
2. **Copied values are the common case.** Exfiltration and redirection
   usually move a literal value (a URL, an address, a secret) from untrusted
   content into an argument, which exact and fuzzy matching can catch without
   a planner/executor redesign.
3. **Conservative propagation is acceptable** when every block comes with the
   full flow that explains it.
4. **A drop-in library and MCP proxy** will be adopted where a research
   prototype isn't.

## Why now

The research result exists but there is no open-source, drop-in
implementation. Agents now combine browsing, email and file access with
side-effecting tools, which is exactly the source-to-sink shape taint
tracking addresses.

## What success looks like

- On AgentDojo-style suites, AgentTaint blocks most exfiltration and
  redirection attacks with a benign-task completion rate close to the
  undefended agent's.
- False positives from fuzzy matching are low enough that teams keep it in
  blocking mode.
- Works as a wrapper for at least two agent frameworks and as an MCP proxy on
  the shared OpenAstra core.

## What would prove us wrong

- Most real attacks pass through model-computed values (summaries,
  paraphrases) that matching can't trace, and the planner/executor split is
  too invasive for teams to adopt.
- Conservative labelling taints nearly everything in realistic tasks, so
  everything is blocked or everything is approved.
- Model providers ship equivalent provenance tracking natively.

## Not a bet on

Content classifiers, or limiting which tools may be called (that's
AgentLock).

## Where it fits

Area VIII, AI Runtime Security. Shares sink policy with
[AgentLock](https://github.com/sadayamuthu/agentlock); static counterpart
is [ThreatDiff](https://github.com/sadayamuthu/threatdiff).
