# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `minimax-speech` Agent Skill: cloud text-to-speech (T2A v2) with 332 preset voices across 40+ languages, voice cloning, text-described voice design, inline pause control, word- and sentence-level timestamps, streaming, and async long-form synthesis, plus a full voice catalogue reference.

### Changed
- `minimax-music` skill and API reference rewritten against the live API: six models documented (including free-tier variants and their 3 RPM limit), verified section tags, and corrected request/response schemas.
- README covers all three skills, documents how they compose into a single spot, and reflects the live-API corrections.

### Fixed
- `minimax-video`: corrected the H3 endpoints and schemas against the live API — polling is `GET /v1/query/video_generation` (the previously documented `/v2/video_generation/query` returns 404, which made poll loops spin silently until timeout); the query returns a `file_id` that must be resolved via `/v1/files/retrieve` rather than a `video_url`; `duration`, `resolution` and `ratio` are all required; task status is `Preparing` → `Processing` → `Success`/`Failed`; error payloads are nested under `error`; `adaptive` is a valid ratio; no seed parameter exists.

## [1.0.0] - 2026-08-02

### Added
- `minimax-video` Agent Skill: text-to-video, image-to-video, and multimodal reference-to-video generation with MiniMax H3, including prompt engineering, API orchestration, polling, and ffmpeg post-processing playbooks.
- `minimax-music` Agent Skill: full song, instrumental, and cover-song generation with MiniMax Music 3.0, including lyrics generation, genre/mood libraries, and audio post-processing playbooks.
- Repository README with setup instructions for Claude Code and general agent-harness compatibility.
