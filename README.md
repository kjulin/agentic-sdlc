# agentic-sdlc

A personal agentic software development lifecycle, packaged as a Claude Code plugin. Each stage of the lifecycle is a slash command:

1. `/concept` — shape a fuzzy idea into a concept doc
2. `/spec` — turn the concept into an executable spec
3. `/verification-plan` — design how we'll verify the spec
4. `/implement` — build it
5. `/verify` — run the verification plan

Currently early — only `/concept` is wired up, and it's a stub.

## Install

In Claude Code:

```
/plugin marketplace add kjulin/agentic-sdlc
/plugin install agentic-sdlc@agentic-sdlc
```

## License

MIT — see [LICENSE](LICENSE).
