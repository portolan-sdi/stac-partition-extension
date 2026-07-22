# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed

- Canonical schema URI moved from
  `https://portolan-sdi.github.io/stac-partition-extension/v1.0.0/schema.json` to
  `https://schemas.portolan-sdi.org/partition/v1.0.0/schema.json`, following the
  Portolan schema-hosting convention (one path segment per schema on
  `schemas.portolan-sdi.org`, as with the profile under `/portolan/`). The schema
  `$id`, the `stac_extensions` declaration it enforces, the README identifier, and
  all examples now use the canonical URI.

## [v1.0.0] - 2026-05-06

### Added

- Initial schema definition with fields: `partition:scheme`, `partition:strategy`,
  `partition:keys`, `partition:file_count`, `partition:glob` (asset-level).
- Example collections: kdtree spatial partitioning, H3 grid partitioning, multi-key
  temporal+spatial partitioning.
- Consumer code examples for DuckDB and PyArrow.
- GitHub Actions workflows for CI validation and GitHub Pages publishing.
