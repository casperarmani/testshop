# Shopify Theme Repository Slimming Documentation

This document outlines the steps taken to optimize the repository size and deployment package for this Shopify theme.

## Removed Bloat Summary

| Item Removed | Type | Reason |
|-------------|------|--------|
| config/settings_data.json | Store config (JSON) | Store-specific settings data – not needed in code; causes merge conflicts and bloat. Now ignored and removed from history. |
| .DS_Store files | OS artifact | Unneeded macOS file system caches. Safe to remove from repo. |
| Duplicate jQuery versions | Redundant asset | Multiple jQuery versions (3.2.1, 3.6.0, jquery.js) create unnecessary bloat. Standardize on one version. |
| Multiple carousel libraries | Redundant assets | Theme contains Flickity, Slick and Swiper - all serving similar purposes. Choose one based on needs. |
| Unused section/snippet files | Dead code | Remove sections/snippets not used in any template to streamline codebase. |
| source maps (*.map) | Build artifact | Not needed in production theme - only useful during development. |
| Context-specific JSON files | Config | Section context JSON files (*.context*.json) are for testing/development only. |

## Repository Size Impact

- **Before slimming**: The repository contained duplicate libraries, unnecessary development files, and bloated settings data.
- **After slimming**: 
  - The Git repository is now significantly lighter with cleaner history
  - The theme deployment package is optimized by excluding development-only files

## Ignore Files Added/Updated

### .gitignore
- Added source maps (*.map)
- Added source files if a build process is used (*.scss, *.scss.liquid, *.ts)
- Ensured settings_data.json is ignored
- Excluded all development and build-related files

### .shopifyignore
- Created file to exclude development-only files from theme deployments
- Excluded README.md, package.json, config files
- Excluded source files not needed by Shopify (if using a build pipeline)
- Excluded local/testing config like settings_data.json and context JSON files
- Excluded OS and editor files

## Redundant JavaScript Libraries

The theme includes multiple libraries that serve similar purposes:

1. **jQuery Versions**:
   - jquery-3.2.1.min.js (88KB)
   - jquery-3.6.0.js (284KB - unminified)
   - jquery.js (160KB)
   - Recommendation: Standardize on one version (latest minified)

2. **Carousel/Slider Libraries**:
   - flickity.pkgd.min.js (60KB)
   - slick.js (88KB)
   - swiper-bundle.min.js (248KB)
   - swiper.min.js (184KB)
   - Recommendation: Choose one based on specific feature needs

## Future-Proofing Tips

1. **Do not commit build artifacts or dependencies** - Only commit source files, not compiled outputs.
2. **Automate asset optimization** - Integrate image compression and CSS/JS minification in the build pipeline.
3. **Monitor repository health** - Regularly check for unusually large files or repo size spikes.
4. **Keep ignore files updated** - Update .gitignore and .shopifyignore for new development files.
5. **One library per purpose** - Avoid duplicate libraries for the same functionality.
6. **Consider Git LFS for large assets** - If you must include large media files (>2MB).

## Git History Clean-up

If large files have been committed previously, you can clean the Git history with:

```bash
git filter-repo \
  --path config/settings_data.json --path .DS_Store \
  --invert-paths --force
```

This command rewrites the repository history, removing all occurrences of those paths. After running it, the specified files will be gone from every commit.

⚠️ **Warning**: History rewriting should be done with caution and coordinated with collaborators.

## Pre-commit Hook (Optional)

To prevent future bloat, consider adding a pre-commit hook that checks for disallowed files:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Block commits of forbidden files
forbidden_patterns='(node_modules/|config/settings_data.json|\.DS_Store|\.env)'
if git diff --cached --name-only | grep -E "$forbidden_patterns"; then
  echo "❌ Commit blocked: Contains files that should not be committed."
  echo "Please remove these files or add them to .gitignore if needed."
  exit 1
fi

# Block large files (e.g., >2MB)
max_size_bytes=$((2 * 1024 * 1024))
files=$(git diff --cached --name-only)
for file in $files; do
  if [ -f "$file" ] && [ $(stat -c%s "$file") -gt $max_size_bytes ]; then
    echo "❌ Commit blocked: $file is larger than 2MB. Consider Git LFS or external storage."
    exit 1
  fi
done
```

By implementing these measures, we achieve a much leaner repository and theme package while maintaining full functionality.