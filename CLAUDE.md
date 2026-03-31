# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**lolcommits** — a Ruby gem that captures a webcam snapshot on every git/hg commit. Supports still images, video, animated GIFs, and a plugin system. Ruby >= 3.1, requires ImageMagick.

## Commands

```bash
bundle install                  # Install dependencies
rake test                       # Run MiniTest unit tests (default task)
bin/cucumber                    # Run Cucumber integration tests (primary test suite)
bin/rubocop --parallel          # Lint (matches CI)
bin/cucumber --profile default  # Run specific feature tags, e.g. @wip
```

Run a single Cucumber scenario by line: `bin/cucumber features/lolcommits.feature:42`
Run a single MiniTest file: `bundle exec ruby test/permissions_test.rb`

## Architecture

### Entry Point & CLI

`bin/lolcommits` -> `lib/lolcommits.rb` -> requires all core modules and `runner.rb`.
The `App` class (in bin/) mixes in `OptparsePlus::Main`, `Lolcommits`, and `Lolcommits::CLI`.

### VCS Backend Pattern

`VCSInfo` provides a unified interface over Git and Mercurial. `backends/` contains:
- `git_info.rb` / `mercurial_info.rb` — extract commit metadata (sha, message, branch, author, date)
- `installation_git.rb` / `installation_mercurial.rb` — hook install/remove

### Capture System

`Runner` orchestrates: pre_capture hooks -> platform capturer -> post_capture hooks -> capture_ready hooks.
`Platform.capturer_class(animate:)` selects the capturer by OS. Each capturer extends `Capturer` in `capturer/`. `CaptureFake` is the test double (copies fixture files instead of using webcam).

Platform tools: `imagesnap`/`videosnap` (macOS, vendored in `vendor/ext/`), `mplayer` (Linux), `CommandCam` (Windows).

### Plugin System

Plugins are gems prefixed `lolcommits-`. `PluginManager` discovers them at startup.
Plugin class naming: gem `lolcommits-loltext` -> class `Lolcommits::Plugin::Loltext`, extending `Plugin::Base`.
Lifecycle hooks: `run_pre_capture`, `run_post_capture`, `run_capture_ready`.
`lolcommits-loltext` is bundled as the default plugin.

### Core Extensions (`lib/core_ext/`)

Monkey-patches for `mercurial-ruby` gem to fix Ruby 3.x compatibility (String#encode, ConfigFile#exists?, Repository.open, Windows Command execution).

## Testing

Cucumber (Aruba) is the primary test suite. Tests use `LOLCOMMITS_CAPTURER=Lolcommits::CaptureFake` and `LOLCOMMITS_FAKE_HOST_OS` env vars to mock platform/capture. Cucumber tags: `@slow_process`, `@fake-interactive-rebase`, `@fake-no-imagemagick`, `@fake-no-ffmpeg`, `@requires_ffmpeg`.

MiniTest has minimal coverage (only `test/permissions_test.rb`). Test fixtures live in `test/assets/`.

## Code Style

RuboCop with target Ruby 3.1. Double quotes enforced in lib/, test/, Gemfile. Ruby 1.9 hash syntax. `%i[]`, `%w[]`, `%r{}`, `%()` delimiters. No hard tabs. `vendor/**/*` excluded.

## Configuration

Lolcommit config per-repo: `~/.lolcommits/<repo-name>/config.yml`. Test mode: `~/.lolcommits/test/`. `LOLCOMMITS_DIR` overrides the base directory.
