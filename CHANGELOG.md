# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v1.0.0] - 2026-05-06

### Added

- Initial schema definition with fields: `partition:scheme`, `partition:strategy`,
  `partition:keys`, `partition:file_count`, `partition:glob` (asset-level).
- Example collections: kdtree spatial partitioning, H3 grid partitioning, multi-key
  temporal+spatial partitioning.
- Consumer code examples for DuckDB and PyArrow.
- GitHub Actions workflows for CI validation and GitHub Pages publishing.
