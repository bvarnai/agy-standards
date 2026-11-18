# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Native Antigravity plugin manifest (`plugin.json`) and monochrome mark (`assets/logo.svg`).
- Always-on rule entrypoint (`rules/GEMINI.md`) importing spec-driven, commit convention, and stack-routing fragments (`fragments/`).
- Stack-specific engineering standards for Go (`standards/go.md`), Java (`standards/java.md`), embedded C/C++ (`standards/embedded.md`), and Shell/Bash (`standards/shell.md`).
- Lifecycle skills (`skills/00-spec` through `skills/05-release`) covering specs, issue drafting, implementation, documentation review, pre-release audit, and release tagging.
- Static lint test suite (`tests/lint/*.bats`) validating frontmatter, relative links, rule byte budget, commit templates, and permission declarations.
- Documentation covering engineering philosophy (`docs/philosophy.md`), model/effort recommendations (`docs/models.md`), and an end-to-end walkthrough (`docs/walkthrough.md`).
