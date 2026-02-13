# Orklys Playground

## Workflow

Work happens in `projects/`. Each project has its own folder with context, copy drafts, and website drafts. Only **vetted, live pages** get promoted to `pages/` for hosting.

**Workflow:** `projects/<name>/drafts-website/` → review & approve → copy to `pages/<name>-v1/`

## Scaffolding a New Project

1. Create the project folder in `projects/<name>/` with this structure:
   ```
   projects/<name>/
   ├── context/
   │   ├── [add_context_here]
   ├── drafts-copy/
   │   └── draft-copy-1.md
   └── drafts-website/
       └── draft-website-1/
           ├── index.html
           └── style.css
   ```
2. Add the project to this file under **Project Structure**
3. Do NOT add anything to `pages/` — that only happens when a draft is approved

## Publishing a Page

Only publish vetted pages. When the user approves a draft for publishing:

1. Create a versioned folder in `pages/` (e.g., `pages/<name>-v1/`)
2. Copy the approved `index.html` and `style.css` (and any assets) into it
3. Update the root `index.html` to link to the new page
4. Update the structure listing below
5. Commit and push

## GitHub Pages Hosting

This repo uses GitHub Pages to host static HTML pages. Jekyll is disabled via `.nojekyll`.

**Live pages structure:**

```
/
├── .nojekyll          # Disables Jekyll processing
├── index.html         # Directory listing of all live pages
└── pages/             # Only vetted, live pages go here
    ├── orklys-com-v1/
    ├── orklys-com-v2/
    ├── project-page-v1/
    ├── project-page-v2/
    └── you-are-screwed-campaign-v1/
```

**URLs:**

- Directory: `https://<user>.github.io/<repo>/`
- Individual page: `https://<user>.github.io/<repo>/pages/<page-name>/`

## Project Structure

- `projects/` — Working drafts and context documents (not hosted)
  - `aarhus-demo/` — Aarhus demo project
  - `orklys-com/` — Main website project
  - `project-page/` — Community project page template
  - `you-are-screwed-campaign/` — Campaign project
- `pages/` — Published, vetted pages for GitHub Pages hosting
- `reference/` — Design system and reference documentation
