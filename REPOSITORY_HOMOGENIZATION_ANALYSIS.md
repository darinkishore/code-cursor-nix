# Repository Homogenization Analysis

> Comprehensive comparison and standardization proposal for three AI tool Nix flakes

**Repositories analyzed:**
- `code-cursor-nix` (jacopone) - Cursor AI editor
- `antigravity-nix` (jacopone) - Google Antigravity IDE
- `claude-code-nix` (sadjow) - Claude Code CLI

---

## Executive Summary

All three repositories serve the same fundamental purpose: **auto-updating Nix packages for AI coding tools**, providing faster updates than upstream nixpkgs. However, they exhibit inconsistencies in:

1. **Update frequency** (3x/week vs daily vs hourly)
2. **README structure** and detail level
3. **Flake.nix conventions** (overlay support, version tracking)
4. **Feature highlighting** (unique value propositions)
5. **Release management** (tags/releases vs direct commits)

**Recommendation**: Adopt a unified template that preserves each tool's unique features while maintaining consistent structure, documentation patterns, and user experience.

---

## Current State Comparison

### 1. Update Frequency

| Repository | Schedule | Justification |
|------------|----------|---------------|
| **code-cursor-nix** | 3x/week (Mon/Wed/Fri 9:00 UTC) | Balance between freshness and API courtesy |
| **antigravity-nix** | Daily (2:00 AM UTC) | Catch updates quickly |
| **claude-code-nix** | Hourly | Maximum freshness |

**Issue**: Inconsistent update philosophy across repos maintained by same person.

**Recommendation**:
- **Standard**: 3x/week (Mon/Wed/Fri) for all repos
- **Rationale**:
  - Strikes balance between currency and respectful API usage
  - Updates typically available within 48 hours of release
  - Reduces CI costs and GitHub Actions noise
  - Aligns with code-cursor-nix's proven pattern

### 2. README Structure Comparison

#### code-cursor-nix (BEST - most comprehensive)
```markdown
✅ Title with tagline
✅ Status badges
✅ Feature list with emojis
✅ Unique value proposition section (Browser Automation)
✅ Multiple installation methods (4 variants)
✅ Usage examples
✅ Version management with releases
✅ Detailed "How It Works" section
✅ Comparison table (vs alternatives)
✅ "Why This Matters" section
✅ Troubleshooting (missing)
✅ Project structure
✅ Contributing guidelines
✅ License clarity
✅ Related projects
✅ Maintainers
```

#### antigravity-nix (GOOD - solid basics)
```markdown
✅ Title with tagline
❌ Status badges (missing)
✅ Feature list
❌ Unique value proposition section (generic FHS mention)
✅ Multiple installation methods (4 variants)
✅ Usage examples
❌ Version management explanation (no releases section)
✅ "How It Works" section
❌ Comparison table (missing)
✅ Troubleshooting section
✅ Project structure
✅ Contributing guidelines
✅ License clarity
✅ Related projects
❌ Maintainers section (implicit)
```

#### claude-code-nix (GOOD - clear value prop)
```markdown
✅ Title with tagline
❌ Status badges (assumed present but not in summary)
✅ Feature list
✅ Strong unique value proposition (vs npm/nixpkgs)
✅ Multiple installation methods
✅ Usage examples (implied)
✅ Technical implementation details
✅ Known issues section
❌ Comparison table (narrative instead)
❌ Troubleshooting section
❌ Project structure
✅ Contributing guidelines
✅ License clarity
❌ Related projects
✅ Maintainers (implicit)
```

### 3. Flake.nix Structure Comparison

#### code-cursor-nix
```nix
packages = {
  cursor = ...;              # Named package
  default = ...cursor;       # Points to named
};

# ❌ No overlay
# ❌ No version field in outputs
# ❌ No apps section
devShells.default = ...;     # ✅ Dev shell present
```

#### antigravity-nix
```nix
packages = {
  default = ...;             # Direct default
  google-antigravity = ...;  # Named package
};

apps = {
  default = { ... };         # ✅ Apps section
};

overlays.default = ...;      # ✅ Overlay support
version = "1.11.2-...";      # ✅ Version tracking

devShells.default = ...;     # ✅ Dev shell present
```

#### claude-code-nix (inferred from description)
```nix
packages.default = ...;      # Standard pattern
overlays.default = ...;      # ✅ Overlay support (mentioned)
# ✅ Cachix integration
# ✅ Node.js 22 LTS bundled
```

**Best practices synthesis:**
- ✅ Both `default` and named packages
- ✅ Apps section for `nix run` ergonomics
- ✅ Overlay for NixOS integration
- ✅ Version field for tracking
- ✅ devShells for contributors

### 4. Unique Features by Repository

| Feature | code-cursor-nix | antigravity-nix | claude-code-nix |
|---------|----------------|-----------------|-----------------|
| **Standout Feature** | Browser automation (Chrome in FHS) | Overlay integration emphasis | Node.js bundling, Cachix cache |
| **Multi-platform** | ✅ Linux + macOS (x86_64, aarch64) | ✅ (implied) | ✅ Linux + macOS |
| **Release tagging** | ✅ GitHub releases | ❌ No releases | ❌ (not mentioned) |
| **Comparison table** | ✅ vs nixpkgs/others | ❌ | ✅ (narrative) |
| **Auto-merge PRs** | ✅ Enabled | ❌ Manual review required | ✅ (implied) |

---

## Proposed Standardized Structure

### Standard README Template

```markdown
# {tool-name}-nix

> Auto-updating Nix Flake for [Tool Name](URL) - Tagline

[![Update Tool](badge-url)](workflow-url)
[![Cachix Cache](badge-url)](cachix-url) <!-- if applicable -->

## Features

- 🚀 **Auto-updating**: Checks for new releases 3x weekly via GitHub Actions
- 📦 **Multi-platform**: Supports Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- ⚡ **Fast**: New versions available within 48 hours of official release
- 🔐 **Reliable**: Automatic hash verification and build testing
- 🤖 **Automated PRs**: Creates and auto-merges PRs when tests pass
- 🔧 **{Unique Feature}**: {Tool-specific highlight}

## 🎯 {Unique Value Proposition Section}

### The Problem

{Describe pain point this solves, e.g., browser automation on NixOS}

### Our Solution

{Explain technical solution}

### What This Enables

- ✅ Feature 1
- ✅ Feature 2
- ✅ Feature 3

## Installation

### Try without installing

```bash
nix run github:{maintainer}/{repo}
```

### NixOS (Flakes)

Add to your `flake.nix`:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    {repo}.url = "github:{maintainer}/{repo}";
  };

  outputs = { self, nixpkgs, {repo}, ... }: {
    nixosConfigurations.your-hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        {
          environment.systemPackages = [
            {repo}.packages.x86_64-linux.default
            # Or use overlay:
            # {repo}.overlays.default
          ];
        }
      ];
    };
  };
}
```

### Home Manager

```nix
{
  inputs.{repo}.url = "github:{maintainer}/{repo}";

  outputs = { self, nixpkgs, home-manager, {repo}, ... }: {
    homeConfigurations.your-user = home-manager.lib.homeManagerConfiguration {
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
      modules = [
        {
          home.packages = [ {repo}.packages.x86_64-linux.default ];
        }
      ];
    };
  };
}
```

### Using the Overlay

```nix
{
  nixpkgs.overlays = [ inputs.{repo}.overlays.default ];

  environment.systemPackages = with pkgs; [
    {package-name}
  ];
}
```

## Usage

```bash
{command} [options]
```

## Version Management

This flake automatically tracks the latest stable version.

### Using Releases

We provide tagged releases for version stability:

```nix
# Latest release (recommended)
inputs.{repo}.url = "github:{maintainer}/{repo}";

# Specific version
inputs.{repo}.url = "github:{maintainer}/{repo}/v{X.Y.Z}";
```

View all releases: `https://github.com/{maintainer}/{repo}/releases`

### Updating

```bash
nix flake update {repo}
sudo nixos-rebuild switch --flake .
```

## How It Works

1. **3x weekly checks**: GitHub Actions runs Monday, Wednesday, Friday at 9:00 UTC
2. **Version detection**: Queries {tool}'s API/download page
3. **Multi-platform support**: Downloads and verifies hashes for all supported platforms
4. **Automated testing**: Builds on Linux x86_64 (and macOS if applicable)
5. **Pull requests**: Creates PR with updates, auto-merges if tests pass
6. **Release tagging**: Automatically creates GitHub releases for version tracking

## Comparison with Other Approaches

| Method | Update Speed | {Unique Feature} | Reliability | Platforms |
|--------|-------------|------------------|-------------|-----------|
| **{repo}** | 3x weekly | ✅ | Automated testing | Linux, macOS |
| nixpkgs | Days to weeks | Varies | Manual review | Linux, macOS |
| Direct install | Immediate | Manual setup | Self-managed | Varies |

## Troubleshooting

### Issue 1

**Problem**: {Description}

**Solution**: {Steps to resolve}

### Issue 2

{Additional common issues}

## Project Structure

```
{repo}/
├── flake.nix              # Main flake configuration
├── package.nix            # Package derivation
├── scripts/
│   └── update-version.sh  # Auto-update script
├── .github/
│   └── workflows/
│       └── update.yml     # GitHub Actions workflow
└── README.md
```

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test with `nix build`
4. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) for details.

{Tool Name} itself is proprietary software by {Vendor}.

## Maintainers

- [@{username}](https://github.com/{username})

## Related Projects

- [{other-repo}](link) - {Description}
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Official Nix packages

## Inspired By

{Credit to similar projects that inspired this work}
```

### Standard flake.nix Structure

```nix
{
  description = "{Tool Name} - {Short description} (Nix package)";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs {
          inherit system;
          config.allowUnfree = true;  # If tool is proprietary
        };
      in
      {
        packages = {
          default = pkgs.callPackage ./package.nix { };
          {tool-name} = pkgs.callPackage ./package.nix { };
        };

        apps = {
          default = {
            type = "app";
            program = "${self.packages.${system}.default}/bin/{command}";
          };
        };

        # Development shell for contributors
        devShells.default = pkgs.mkShell {
          packages = with pkgs; [
            nix
            git
            curl
            jq
            gh  # For GitHub CLI operations
          ];
        };
      }
    ) // {
      # Version tracking for auto-update
      version = "{X.Y.Z}";

      # Overlay for NixOS integration
      overlays.default = final: prev: {
        {tool-name} = final.callPackage ./package.nix { };
      };
    };
}
```

### Standard GitHub Actions Workflow

```yaml
name: Auto-update {Tool Name}

on:
  schedule:
    # Run three times weekly: Monday, Wednesday, Friday at 9:00 UTC
    - cron: '0 9 * * 1,3,5'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  update:
    name: Check for {tool} updates
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install Nix
        uses: DeterminateSystems/nix-installer-action@main
        with:
          extra-conf: |
            experimental-features = nix-command flakes
            accept-flake-config = true

      - name: Setup Nix cache
        uses: DeterminateSystems/magic-nix-cache-action@main

      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Check for updates
        id: update
        run: |
          set +e
          ./scripts/update-version.sh
          UPDATE_EXIT_CODE=$?
          set -e

          if [ $UPDATE_EXIT_CODE -eq 0 ]; then
            if git diff --quiet package.nix; then
              echo "has_updates=false" >> $GITHUB_OUTPUT
              echo "No updates available"
            else
              echo "has_updates=true" >> $GITHUB_OUTPUT
              NEW_VERSION=$(grep -oP 'version = "\K[^"]+' package.nix | head -1)
              echo "new_version=$NEW_VERSION" >> $GITHUB_OUTPUT
              echo "Updates available: version $NEW_VERSION"
            fi
          else
            echo "has_updates=false" >> $GITHUB_OUTPUT
            echo "Update script failed with exit code $UPDATE_EXIT_CODE"
            exit 1
          fi

      - name: Test build (Linux x86_64)
        if: steps.update.outputs.has_updates == 'true'
        env:
          NIXPKGS_ALLOW_UNFREE: "1"
        run: |
          nix build .#default --print-build-logs --impure
          nix flake check --impure

      - name: Create Pull Request
        if: steps.update.outputs.has_updates == 'true'
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "chore: update {tool} to ${{ steps.update.outputs.new_version }}"
          title: "chore: update {tool} to ${{ steps.update.outputs.new_version }}"
          body: |
            ## {Tool Name} Update

            This PR updates {tool} to version **${{ steps.update.outputs.new_version }}**.

            ### Changes
            - Updated version in `package.nix`
            - Updated download URLs for all platforms
            - Updated hashes for all platforms
            - Ran `nix flake update`

            ### Testing
            - ✅ Built successfully on Linux x86_64
            - ✅ Flake checks passed

            ### Changelog
            See: {changelog-url}

            ---
            🤖 This PR was created automatically by the auto-update workflow.
          branch: auto-update/{tool}-${{ steps.update.outputs.new_version }}
          delete-branch: true
          labels: |
            automated
            {tool}-update

      - name: Enable auto-merge
        if: steps.update.outputs.has_updates == 'true'
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          sleep 5
          PR_NUMBER=$(gh pr list --head auto-update/{tool}-${{ steps.update.outputs.new_version }} --json number --jq '.[0].number')

          if [ -n "$PR_NUMBER" ]; then
            echo "Enabling auto-merge for PR #$PR_NUMBER"
            gh pr merge "$PR_NUMBER" --auto --squash
          else
            echo "Could not find PR to enable auto-merge"
          fi

      - name: Create Release Tag
        if: steps.update.outputs.has_updates == 'true'
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NEW_VERSION: ${{ steps.update.outputs.new_version }}
        run: |
          # Wait for PR to be merged
          sleep 30

          # Check if PR was merged
          git pull

          # Create release tag
          git tag -a "v$NEW_VERSION" -m "{Tool Name} v$NEW_VERSION"
          git push origin "v$NEW_VERSION"

          # Create GitHub release
          gh release create "v$NEW_VERSION" \
            --title "{Tool Name} v$NEW_VERSION" \
            --notes "Auto-updated {tool} to version $NEW_VERSION. See {changelog-url} for details."
```

---

## Implementation Recommendations

### Priority 1: Immediate Consistency (Both Repos)

1. **Standardize update frequency**
   - Change `antigravity-nix` from daily to 3x/week
   - Update workflow: `.github/workflows/update.yml`
   - Change cron: `'0 2 * * *'` → `'0 9 * * 1,3,5'`

2. **Add missing flake.nix features**
   - code-cursor-nix: Add overlay, version field, apps section
   - antigravity-nix: Already has these ✅

3. **Enable auto-merge for antigravity-nix**
   - Add auto-merge step to workflow (currently missing)
   - Follow code-cursor-nix pattern

### Priority 2: Documentation Parity

4. **Enhance antigravity-nix README**
   - Add status badges
   - Create unique value proposition section (what makes Antigravity special?)
   - Add comparison table
   - Add "Why This Matters" section
   - Expand "How It Works" with more detail

5. **Add troubleshooting to code-cursor-nix**
   - Create troubleshooting section
   - Document common issues (FHS environment, browser automation, permissions)

### Priority 3: Release Management

6. **Implement GitHub releases**
   - Add release creation to both workflows after PR merge
   - Tag format: `v{version}` (e.g., `v2.0.43`)
   - Auto-generate release notes from changelog URLs

7. **Version tracking in flake.nix**
   - Add `version` field to code-cursor-nix outputs
   - Keep synced with package.nix during updates

### Priority 4: Structural Consistency

8. **Use consistent package naming**
   ```nix
   # Both should expose:
   packages = {
     default = ...;           # Primary package
     {tool-name} = ...;       # Named variant
   };
   ```

9. **Standardize devShell contents**
   ```nix
   devShells.default = pkgs.mkShell {
     packages = with pkgs; [
       nix
       git
       curl
       jq
       gh
     ];
   };
   ```

---

## Concrete Action Items

### For `code-cursor-nix`

- [ ] Add `overlays.default` to flake.nix
- [ ] Add `version` field to outputs
- [ ] Add `apps.default` section
- [ ] Add troubleshooting section to README
- [ ] Add explicit maintainers section
- [ ] Implement automated release tagging in workflow

### For `antigravity-nix`

- [ ] Change update frequency to 3x/week
- [ ] Add status badge to README
- [ ] Create unique value proposition section (What is Antigravity? Why use this flake?)
- [ ] Add comparison table (vs nixpkgs, vs direct install)
- [ ] Add "Why This Matters" section
- [ ] Add auto-merge step to workflow
- [ ] Implement automated release tagging
- [ ] Add explicit maintainers section

### For Both

- [ ] Standardize commit message format: `chore: update {tool} to {version}`
- [ ] Standardize PR titles: Same as commit message
- [ ] Standardize branch naming: `auto-update/{tool}-{version}`
- [ ] Ensure all workflows use DeterminateSystems actions for consistency
- [ ] Cross-reference each other in "Related Projects" sections

---

## Benefits of Homogenization

### For Users
- **Predictable experience**: Same installation patterns across all tools
- **Better discoverability**: Consistent README structure helps find information
- **Version stability**: GitHub releases provide pinning points
- **Clear value propositions**: Understand why to use each flake

### For Maintainers
- **Reduced cognitive load**: Same patterns across repos
- **Easier updates**: Copy-paste workflow improvements
- **Professional consistency**: Portfolio of well-maintained flakes
- **Community trust**: Demonstrates attention to detail

### For Community
- **Reference implementation**: Other auto-updating flakes can follow this pattern
- **Best practices**: Demonstrates modern Nix flake conventions
- **Discoverability**: Consistent structure helps search and recommendation

---

## Timeline Suggestion

**Week 1**: Priority 1 (Immediate consistency)
- Update workflows, add missing flake.nix features

**Week 2**: Priority 2 (Documentation parity)
- Enhance READMEs, add missing sections

**Week 3**: Priority 3 (Release management)
- Implement GitHub releases, version tracking

**Week 4**: Priority 4 (Polish)
- Cross-references, final consistency checks

---

## Conclusion

All three repositories are solid foundational work. The proposed homogenization:

1. **Preserves unique features** (browser automation, Node.js bundling, etc.)
2. **Standardizes common patterns** (structure, documentation, workflows)
3. **Improves user experience** through consistency
4. **Simplifies maintenance** through shared patterns

The resulting "template" can serve as a reference for future auto-updating Nix flakes in the AI tooling space.
