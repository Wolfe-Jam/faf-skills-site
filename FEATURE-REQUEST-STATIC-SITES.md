# Feature Request: Add Static Site Detection to `faf auto`

**To submit:** Create issue at https://github.com/Wolfe-Jam/faf-cli/issues

---

## Feature Request Title
Add support for static site detection in `faf auto`

## Use Case
Many static sites using vanilla HTML/CSS/JS or static site generators need .faf context but aren't properly detected. Currently, FAF incorrectly identifies static sites as "library" type with "Unknown" language.

### Real-world example:
While creating skills.faf.one (following the foundation.faf.one pattern), FAF failed to detect:
- Plain `index.html` file (1000+ lines)
- No backend framework indicators
- Clear static website structure

Result: `type: library`, `main_language: Unknown` ❌

## Proposal
Enhance framework detection to recognize:
1. Plain static sites (HTML/CSS/JS only)
2. Static site generators (Jekyll, Hugo, Eleventy, etc.)
3. Static-first frameworks (Astro in static mode)

## Implementation Ideas

### 1. Detection Logic
```javascript
// Plain static sites
if (exists('index.html') || exists('index.htm')) {
  return {
    type: 'website',
    main_language: 'html',
    stack: {
      frontend: 'vanilla-html',
      css_framework: 'vanilla-css',
      ui_library: 'vanilla-js',
      backend: 'static',
      runtime: 'browser'
    }
  };
}

// Static site generators
const staticGenerators = [
  { file: '_config.yml', name: 'jekyll' },
  { file: 'hugo.toml', name: 'hugo' },
  { file: '.eleventy.js', name: 'eleventy' },
  { file: 'astro.config.mjs', name: 'astro' }
];
```

### 2. New Project Types
Add to project type options:
- `static-site` - Plain HTML/CSS/JS
- `static-generated` - Built with SSG
- `docs-site` - Documentation focus
- `landing-page` - Marketing site

### 3. Better Defaults
When detecting static sites, don't use "slotignored" for obvious fields:
- `frontend` → `vanilla-html` or detected framework
- `backend` → `static`
- `runtime` → `browser`
- `hosting` → check for vercel.json, netlify.toml, etc.

## Impact
This affects a large number of projects:
- Portfolio sites
- Documentation sites
- Marketing landing pages
- GitHub Pages projects
- FAF's own ecosystem sites (foundation.faf.one, skills.faf.one)

## Current Workaround
Users must manually run `faf go` and correct multiple misdetections:
1. Change type: library → website
2. Change language: Unknown → html  
3. Fill all stack fields manually

This breaks the "7-letter magic" promise of `bunx faf`.

## Test Cases
```bash
# Test 1: Plain static site
mkdir test-static && cd test-static
echo "<!DOCTYPE html><html><body>Hello</body></html>" > index.html
bunx faf
# Expected: type: website, main_language: html ✅
# Current: type: library, main_language: Unknown ❌

# Test 2: With CSS/JS
mkdir assets && echo "body { color: red; }" > assets/style.css
bunx faf auto
# Expected: Detects vanilla-css ✅
# Current: No improvement ❌
```

## Priority
HIGH - Static sites are extremely common, and the current behavior actively misleads users.

## References
- FAF Version: 3.0
- Encountered during: Creation of skills.faf.one
- Related sites: foundation.faf.one (likely had same issue)

---

*Following SUPPORT.md guidelines for feature requests*