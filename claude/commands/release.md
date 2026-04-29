# /release - Create a GitHub release with generated changelog

Generates a changelog from merged PRs and commits since the last git tag, determines the next semver version, and creates a GitHub release.

## Usage

```
/release            # Auto-detect version bump from changes
/release patch      # Force patch bump
/release minor      # Force minor bump
/release major      # Force major bump
```

Follow the **release skill** for the full procedure.

The version argument (patch, minor, major) is optional. If provided, use it to override auto-detection in Step 5.
