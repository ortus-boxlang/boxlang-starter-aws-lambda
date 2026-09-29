# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

----

## [Unreleased]

### Added

- `src/main/bx/handlers/` convention for routed handlers, with a flat (`Products.bx`) and a nested (`api/Test.bx`) example.
- `generateManifest` Gradle task, which scans `handlers/` and generates `manifest.json` so the runtime never scans the filesystem for routable handlers at cold start. Wired into `test`, `runLocal`, `runLocalApi`, `runLocalLegacy`, and `buildLambdaZip` so it can never silently drift out of date.

### Changed

- Bumped `boxlangVersion` to 1.18.0, which includes a security fix restricting URI-routing and `x-bx-function` header dispatch to registered handlers only (previously, any root-level `.bx` file and any of its public methods, including `Application.bx`'s lifecycle callbacks, could be reached this way).

* First release
