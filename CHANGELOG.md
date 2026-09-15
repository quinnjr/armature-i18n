# Changelog — `armature-i18n`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

### Changed

- Bumped `icu_locale_core`, `icu_datetime`, `icu_decimal`, `icu_plurals`, and
  `icu_calendar` from `2.2` to `2.3` as part of a workspace-wide dependency
  upgrade. All five stay aligned on the 2.3 line with no duplicate versions in
  the dependency graph (`cargo tree -d` is clean). No migration was needed:
  the `icu` feature these crates gate is declared in `Cargo.toml` but not
  referenced anywhere in this crate's source, so nothing in `src/` depends on
  their API surface.

## [0.5.0] - 2026-08-05

### Changed

- **Requires `armature-core` 0.9 (breaking).** The requirement moved `0.8` →
  `0.9`. `armature-core 0.9.0` itself moves `armature-h1` across a breaking
  0.x boundary; because `armature-core` types appear in this crate's own
  public API, the requirement change is breaking here too and the minor moves
  with it. Under Cargo's 0.x caret rules the 0.8 and 0.9 types are distinct
  and do not unify, so a consumer holding an `armature-core 0.8` type cannot
  pass it to this crate. Part of the `armature-core 0.9.0` release train; see
  `armature-core`'s CHANGELOG for the publish order.

## [0.4.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.

## [0.4.0] - 2026-08-04

### Fixed

- **Behaviour — `parse_accept_language`:** an entry with `q=0` is dropped
  rather than ranked last. RFC 9110 section 12.5.4 defines `q=0` as "not
  acceptable", so such an entry names a locale the client is *refusing*.
  Returning it inverted that: `negotiate_locale` walks the list in order and
  would select the refused locale whenever nothing the client accepted was
  available.
- A quality outside `0..=1` is clamped into it — above `1` becomes `1`, and
  anything negative (including `-inf`) becomes `0` and is therefore refused
  like an explicit `q=0`. A quality that is not a number, or is `NaN`, is
  ignored and behaves as though no `q` had been sent. `str::parse::<f32>`
  accepts `NaN`, `inf` and any magnitude, none of which is a qvalue.
- Accept-Language entries are ordered with `f32::total_cmp`. The previous
  `partial_cmp(..).unwrap_or(Equal)` reported every pair involving a `NaN` as
  equal, which is not transitive, and `sort` is entitled to produce arbitrary
  output from a comparator that is not a total order. `AcceptLanguageEntry`'s
  `PartialEq` is defined through that same ordering, so `Eq`'s contract with
  `Ord` holds.

- **Breaking:** the fallback chain is evaluated per key, not per bundle, so a key present in `fr.json` but missing from `fr-CA.json` no longer skips `fr` entirely; `has` follows the same chain `t` does.
- Plural and bundle lookups no longer allocate per probe.
