# 🚨 FAF Feature Request: Static Site Detection

**Date:** 2026-04-08  
**Issue:** FAF auto-detection fails for static HTML sites  
**Impact:** Manual intervention required for basic web projects  

---

## Problem Statement

When running `bunx faf` in a static HTML site directory, FAF incorrectly detects:
- **type:** `library` (should be `website`)
- **main_language:** `Unknown` (should be `html`)
- **app_type:** Not detected/assigned at all

This happens even with clear static site indicators present:
- `index.html` file
- No backend framework files
- HTML/CSS/JS only structure

## Current Behavior

```bash
# In a directory with index.html
bunx faf
# Result: type: library, main_language: Unknown ❌

bunx faf auto
# Result: No improvement in detection ❌
```

## Requested Enhancement

### 1. Add Static Site Detection Logic

**Detection signals for static sites:**
```javascript
// Priority 1: HTML entry files
if (exists('index.html') || exists('index.htm')) {
  type: 'website'
  main_language: 'html'
  app_type: 'static-site'  // NEW
}

// Priority 2: Common static site patterns
if (exists('404.html') || exists('about.html')) {
  type: 'website'
  suggested_app_type: 'static-site'
}

// Priority 3: Static site generators (but still static output)
if (exists('_site/') || exists('dist/index.html')) {
  type: 'website'
  app_type: 'static-generated'
}
```

### 2. Add app_type Field Options

Suggested `app_type` values for websites:
- `static-site` - Pure HTML/CSS/JS
- `static-generated` - Built from generator (Jekyll, Hugo, etc.)
- `spa` - Single Page Application
- `pwa` - Progressive Web App
- `docs-site` - Documentation site
- `landing-page` - Marketing/landing page

### 3. Improve Stack Detection for Static Sites

When `app_type: static-site` is detected, auto-fill appropriate stack:
```yaml
stack:
  frontend: vanilla-html  # Not "slotignored"
  css_framework: vanilla-css
  ui_library: vanilla-js
  backend: static  # Not "slotignored"
  runtime: browser
  build: none  # or "static" 
  hosting: # Detect from vercel.json, netlify.toml, etc.
```

## Use Cases Affected

1. **FAF Foundation Sites** - foundation.faf.one, skills.faf.one
2. **Documentation Sites** - Many projects use static HTML
3. **Landing Pages** - Marketing sites, product pages
4. **Portfolio Sites** - Developer portfolios
5. **GitHub Pages** - Tons of static sites

## Workaround (Current)

Users must manually use `faf go` to correct the misdetection:
1. Change type from `library` → `website`
2. Change language from `Unknown` → `html`
3. Fill all stack fields manually

## Expected Behavior

```bash
# In a directory with index.html
bunx faf
# Result: type: website, main_language: html ✅
# Result: app_type: static-site ✅

bunx faf auto  
# Result: Detects vanilla stack appropriately ✅
# Result: Score jumps significantly (not stuck at 11%) ✅
```

## Priority

**HIGH** - Static sites are extremely common, and current detection actively misleads users by categorizing them as libraries.

## Additional Context

During creation of skills.faf.one, FAF failed to detect obvious static site patterns even with:
- index.html (1000+ lines)
- package.json with "homepage" field
- No backend code whatsoever
- Clear static site structure

This impacts the "7-letter magic" promise when Boris has to manually correct basic detection errors.

---

**Recommendation:** Add static site detection as a priority before library fallback. Most directories with `index.html` are websites, not libraries.

*Filed during faf-skills-site creation when FAF stubbornly insisted it was a library* 🤦