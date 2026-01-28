# Orklys Project

## GitHub Pages Hosting

This repo uses GitHub Pages to host static HTML pages. Jekyll is disabled via `.nojekyll`.

**Structure:**
```
/
├── .nojekyll          # Disables Jekyll processing
├── index.html         # Directory listing of all pages
└── pages/             # All hosted pages go here
    ├── orklys-com-v1/
    ├── orklys-com-v2/
    ├── project-page-v1/
    └── project-page-v2/
```

**To add a new page:**
1. Create a folder in `pages/` (e.g., `pages/my-new-page/`)
2. Add `index.html` and `style.css` to that folder
3. Update the root `index.html` to link to it
4. Commit and push

**URLs:**
- Directory: `https://<user>.github.io/<repo>/`
- Individual page: `https://<user>.github.io/<repo>/pages/<page-name>/`

## Project Structure

- `projects/` — Working drafts and context documents (not hosted)
  - `orklys-com/` — Main website project
  - `project-page/` — Community project page template
- `pages/` — Published pages for GitHub Pages hosting
- `reference/` — Design system and reference documentation
