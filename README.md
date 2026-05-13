# Auto Audit

A GitHub Actions workflow that runs `npm audit` and automatically opens PRs for fixable vulnerabilities, and tracks unfixable ones in a GitHub issue.

## How it works

1. Runs `npm audit` and saves the report.
2. Splits every vulnerability into one of four buckets based on `fixAvailable`:

   | Bucket             | Condition                              | Action                                   |
      |--------------------|----------------------------------------|------------------------------------------|
   | `safe`             | `fixAvailable: true`                   | PR via `npm audit fix` (in-range)        |
   | `forceNonBreaking` | object, not SemVer-major               | PR via targeted `npm install` per pkg    |
   | `breaking`         | object, SemVer-major                   | PR via targeted `npm install`, flagged   |
   | `unfixable`        | `fixAvailable: false`                  | Opens/updates a tracking issue           |

3. Each PR uses a stable branch (`auto-audit/fix-safe`, `-fix-force`, `-fix-breaking`), so daily runs update the existing PR in place instead of opening duplicates.
4. The unfixable-vulns issue is labelled `auto-audit-unfixable`, edited in place each run, and closed automatically once all entries gain upstream fixes.

## Layout

```
.github/
  workflows/check-vulnerabilities.yml   # orchestration
  scripts/
    parse-audit.mjs                     # audit.json -> audit-summary.json + step outputs
    open-fix-pr.sh                      # creates/updates the PR for a given bucket
    sync-unfixable-issue.sh             # creates/edits/closes the tracking issue
.npmrc                                  # save-exact=true (no `^` prefixes on new installs)
package.json                            # demo dependencies covering every bucket
```

## Demo dependencies

`package.json` intentionally pins old versions to exercise every bucket:

- `lodash ^4.17.0` -> safe (in-range patch fix)
- `axios 1.15.0` -> non-breaking out-of-range upgrade
- `express 3.0.0` -> breaking (SemVer-major) upgrade
- `request 2.88.0` -> unfixable (deprecated, no patch)
