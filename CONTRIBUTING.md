# Contributing to WebBrain

Thanks for your interest in contributing. WebBrain is an open-source AI browser agent, and it gets better every time someone files a thoughtful bug report, opens a pull request, improves the docs, or helps another user. This document describes how to do that effectively.

By participating in this project, you agree to abide by the project's Code of Conduct.

## Ways to Contribute

You don't have to write code to help. Useful contributions include:

Reporting bugs with clear reproduction steps. Suggesting features or improvements, especially ones grounded in a real use case. Improving documentation, including the README, this guide, and inline code comments. Adding or improving translations of the UI and marketing site. Triaging existing issues — reproducing bugs, asking clarifying questions, suggesting labels. Reviewing other people's pull requests. Sharing how you use WebBrain so we can learn what's working and what isn't.

## Reporting Bugs

Before opening a bug report, please search existing issues to see if it's already been reported. If not, open a new issue and include:

What you were trying to do, what actually happened, and what you expected to happen. Your browser (Chrome or Firefox) and version, and the WebBrain version. The LLM provider you were using (llama.cpp, OpenAI, Claude, etc.) and, if relevant, the model name. Steps to reproduce, ideally on a public URL we can test against. Any relevant logs from the browser console or the WebBrain trace viewer. Screenshots if the issue is visual.

For security-sensitive issues, please do **not** open a public issue. Follow the process in `SECURITY.md` or contact the maintainers privately.

## Suggesting Features

Feature suggestions are welcome. Before opening a feature issue, please consider:

Does the feature fit WebBrain's scope as a browser-based AI agent? Could it be implemented as an opt-in setting rather than a default change? Does it have security or privacy implications worth discussing up front?

The best feature requests describe a concrete problem ("when I do X, Y happens, and I wish Z") rather than a solution in search of a problem.

## Development Setup

WebBrain is a monorepo containing both Chrome (Manifest V3) and Firefox (Manifest V2) builds, plus a small marketing site and an LM Studio plugin.

Clone the repository:

```
git clone https://github.com/esokullu/webbrain.git
cd webbrain
```

For Chrome development, open `chrome://extensions/`, enable Developer mode, click "Load unpacked," and select the `src/chrome/` folder. Reload the extension after each change.

For Firefox development, open `about:debugging#/runtime/this-firefox`, click "Load Temporary Add-on," and select `src/firefox/manifest.json`. Note that temporary add-ons are removed when Firefox restarts.

For end-to-end testing with a local LLM, start `llama-server` or `ollama serve` and point WebBrain at the appropriate base URL in the extension's settings page.

The codebase is organized roughly as: `agent/` for the agent loop and tool dispatch, `providers/` for LLM provider adapters, `content/` for content scripts injected into pages, `network/` for fetch and research tools, `trace/` for the trace viewer, and `ui/` for the side panel UI.

## Pull Request Process

Fork the repository and create a branch from `main` with a descriptive name (`fix/pdf-tool-timeout`, `feat/grok-provider`, etc.).

Make your change. Keep the change focused — one logical change per pull request makes review much easier. If you're touching multiple unrelated things, split them into separate PRs.

Test your change manually in both Chrome and Firefox where applicable. Pay particular attention to the differences documented in the README's "Known Issues" section — Firefox lacks several Chrome-only features and your change should not silently break the Firefox build.

Update documentation if your change affects user-visible behavior, adds a new tool, adds a new provider, or changes a default.

Open a pull request with a clear title and description. Explain what the change does, why it's needed, and how you tested it. If the change is user-visible, suggest a one-line entry for the changelog in the README's "What's New" section.

A maintainer will review your PR. Reviews are about the code, not the contributor — please don't take feedback personally, and please offer feedback the same way when reviewing others' work. Lazy consensus applies: one maintainer approval with no objections from other maintainers is enough to merge ordinary changes.

## Coding Conventions

Match the existing style of the file you're editing. The codebase uses plain JavaScript (no TypeScript) and avoids heavy frameworks; please keep new code in the same spirit unless there's a clear reason to do otherwise.

Prefer small, well-named functions over deeply nested logic. Comment the *why* of non-obvious decisions, not the *what* — the code already says what.

For agent tools, follow the existing tool pattern (definition, dispatch, normalized output) and document the new tool in the README's Agent Tools table.

For new LLM providers, extend `BaseLLMProvider` in `src/providers/`, implement `chat()` and optionally `chatStream()`, and register the provider in `src/providers/manager.js`. All providers must normalize to `{ content, toolCalls, usage }`.

## Safety and Defaults

WebBrain takes real actions on real web pages. Any change that affects what the agent will do without explicit user confirmation deserves extra care.

Features that increase the agent's blast radius (auto-acting on more sites, bypassing safety checks, sending more data to external services, etc.) should be opt-in and off by default. Defaults are conservative on purpose — please don't loosen them without discussion.

If your change touches API mutation behavior, paywall handling, profile auto-fill, CAPTCHA handling, or any of the other safety-sensitive areas, please call that out explicitly in the PR description so reviewers know to look closely.

## Translations

WebBrain ships in multiple languages. To add or update a translation, edit the relevant locale file under the extension's `_locales/` directory (Chrome) or the equivalent Firefox location. Please preserve placeholder syntax (`$1`, `{name}`, etc.) exactly.

If you're adding a brand-new language, also add it to the language switcher in the UI and to the language list on the marketing site.

## License

WebBrain is released under the MIT License. By submitting a pull request, you agree that your contribution will be licensed under the same MIT License. You retain the copyright on your contributions; you are not assigning copyright to the project or any other entity.

If WebBrain's licensing or governance changes in the future (for example, by joining a foundation), existing contributions remain under the MIT License under which they were originally submitted, and any change in stewardship will be communicated openly in advance.

## Questions

If you're not sure where to start or whether an idea is in scope, open a GitHub Discussion (or issue, if discussions aren't enabled) and ask. We'd rather have the conversation up front than have you spend a weekend on something that doesn't fit.
