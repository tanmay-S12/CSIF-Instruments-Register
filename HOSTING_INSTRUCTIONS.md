# Hosting the instrument register (free, no branding)

Once you've plugged your Google Apps Script URL into `instrument_register.html`
(see GOOGLE_SHEET_SETUP.md), you need somewhere to host that single HTML file so people
can open it via a link. All options below are free and show nothing but your own page —
no Anthropic or Claude branding anywhere.

## Easiest: Netlify Drop
1. Go to https://app.netlify.com/drop
2. Drag `instrument_register.html` onto the page.
3. Netlify gives you a live URL instantly (e.g. `random-name-123.netlify.app`).
4. (Optional) Create a free Netlify account to keep the link permanent and rename it to
   something cleaner (e.g. `asc-instrument-register.netlify.app`).
5. Share that link with your 15+ people.

## Alternative: GitHub Pages
Good if your institute already uses GitHub, or you want a `github.io` link tied to an account.
1. Create a free GitHub account and a new repository.
2. Upload `instrument_register.html` (rename it to `index.html` for a cleaner URL).
3. Go to repo **Settings → Pages**, set source to the main branch, save.
4. GitHub gives you a URL like `https://yourusername.github.io/repo-name/`.

## Alternative: Your institute's own web server
If your department already has a web server or intranet space (common at universities),
just ask IT to drop the HTML file into a public folder — this keeps everything fully
in-house.

## What to send people
Just the link. They don't need any login, app, or account — opening the link in any
browser (desktop or phone) loads the form directly.
