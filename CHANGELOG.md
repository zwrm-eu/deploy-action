# Changelog

All notable changes to this action are documented here. This project adheres to
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- Initial release: composite action that installs the `zwrm` CLI (pinned version or `latest`,
  checksum-verified) and runs `zwrm deploy`.
- Inputs: `api-token`, `org-id`, `app`, `control-plane-url`, `working-directory`, `context`,
  `force-build`, `cli-version`, `release-base-url`.
- Supports Linux and macOS runners on X64 and ARM64.
