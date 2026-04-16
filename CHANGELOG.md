# Changelog

All notable changes to AgentDo will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.2] - 2026-04-16

### Fixed
- `mcp-manifest.json` homepage URL (`.io` → `.me`) — landing is on `agentdo.me`, `agentdo.io` is the dashboard

## [0.1.1] - 2026-04-16

### Added
- Currency switcher for lead price (HUF / EUR / USD) on landing pricing card
- Decimal lead prices supported (step 0.01)
- Pre-prod Basic Auth gate (`PREPROD_BASIC_AUTH` env)
- Ügyfélportál.app webhook integration

### Changed
- Landing page lead price now reads from admin settings live (was hardcoded)
- Landing copy: "developers" → "users (AI agents)" terminology
- Number input attributes now use unescaped EJS tags (HTML5 validation works)

### Fixed
- Landing page did not reflect admin lead price changes
- HTML5 `step`/`min`/`max` attributes were HTML-escaped and silently ignored
