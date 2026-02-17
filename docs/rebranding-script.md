# Rebranding: VIBE FOUNDING to Klaudium Studio

## Problem

The site is built with a Framer template that renders content via React hydration. Directly editing the HTML text (e.g. changing `VIBE FOUNDING` in the markup) does not persist because Framer's JavaScript re-renders the DOM after the initial HTML loads, overwriting any static changes.

## Solution

A runtime JavaScript script was added to every HTML page, placed just before the closing `</body>` tag. The script uses a `MutationObserver` to continuously monitor the DOM for changes and replace all occurrences of the old branding with the new one.

### How the script works

1. **`replaceText(str)`** - A helper function that chains `.replace()` calls to catch all variations:
   - `VIBE FOUNDING STDIO` -> `Klaudium Studio`
   - `VIBE FOUNDING STUDIO` -> `Klaudium Studio`
   - `VIBE FOUNDING` -> `Klaudium Studio`
   - `Klaudium Studio` -> `Klaudium Studio` (catches previous rebrand)
   - `Klaudium` -> `Klaudium Studio` (catches previous rebrand)

2. **`rebrand()`** - Performs the actual replacement across:
   - The page `<title>` tag
   - All `<meta>` tags with matching `content` attributes (og:title, twitter:title, etc.)
   - All text nodes in the `<body>` using a `TreeWalker`

3. **`MutationObserver`** - Watches the entire `<body>` subtree for:
   - `childList` changes (new elements added by Framer hydration)
   - `subtree` changes (nested element updates)
   - `characterData` changes (text content modifications)

   Every time Framer modifies the DOM, the observer triggers `rebrand()` again, ensuring the old branding never appears to the user.

4. **Startup** - The script checks if `document.body` exists:
   - If yes, runs immediately and starts observing
   - If no, waits for `DOMContentLoaded`

### Why MutationObserver was necessary

A simpler approach (running the replacement on `DOMContentLoaded` and `load` events) was tried first but failed. Framer's React hydration happens asynchronously after both events fire, re-rendering all text nodes with the original template content. The `MutationObserver` catches these post-hydration DOM updates and re-applies the branding changes.

## Files modified

The script was added to all 10 HTML pages:

- `index.html`
- `blog/page.html`
- `contact/page.html`
- `privacy-page/page.html`
- `terms-of-use/page.html`
- `coursedemo/page.html`
- `blog/5-must-have-ai-tools-to-streamline-your-business/page.html`
- `blog/ai-vs-manual-work-which-one-saves-more-time-money/page.html`
- `blog/how-ai-is-transforming-workflow-automation-for-businesses/page.html`
- `blog/the-future-of-ai-automation-how-it-s-changing-business-operations/page.html`

(`about/page.html` was skipped as it is an empty file.)

## Updating the branding in the future

To change the brand name again, update the `replaceText()` function in each HTML file. Add a new `.replace()` call for the old name mapping to the new name, and update the existing replacements to target the new name.
