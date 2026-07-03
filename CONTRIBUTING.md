# Contributing

Contributions are welcome.

## Scope

This repository contains a Claude Code skill for generating Feishu/Lark Base exam management systems. Good contributions include:

- improved question-bank parsing rules;
- updated `lark-cli base +...` command examples;
- tested formula patterns;
- troubleshooting notes for Feishu/Lark Base API behavior;
- support for additional question types or normalized schemas.

## Development workflow

1. Update `SKILL.md` for user-facing skill behavior.
2. Update files in `references/` for detailed implementation notes.
3. Keep examples in `references/examples/` small and safe.
4. Do not commit secrets, tokens, private Base URLs, or user data.

## Testing checklist

Before proposing changes, check whether the docs still answer:

- How should a question bank be normalized?
- What resources will be created?
- What commands are used and in what order?
- What must be confirmed before creating or deleting persistent Lark resources?
- How should partial failures be reported?

If you test against a real Feishu/Lark Base, redact private tokens and URLs before opening an issue or pull request.
