# Working in this repository

## What this project is

A single-page **static** marketing site for TaibaBuggyAdventure (buggy tours in
Taíba, Ceará). Plain HTML, CSS and JavaScript — no package manager, no build
step, no framework, no backend and no database:

- `index.html` — the whole page (hero, tours, experience, destination, footer)
- `style.css` — all styling, minified into long single lines
- `script.js` — mobile menu toggle + auto-fill of the footer year
- `assets/*.jpg` — `hero.jpg` is used as a CSS background in `.hero`; `dunas.jpg`,
  `praias.jpg` and `sunset.jpg` are the three tour-card images

Booking is handled by plain `https://wa.me/558597204227` deep links. There is no
form, API key or third-party credential anywhere in the project, so no secrets
are required to run it.

## Running it (Base44 sandbox)

```
docker compose -f docker-compose.base44.yml up -d
```

`docker-compose.base44.yml` runs `nginx:alpine` and serves the checkout itself —
the page is read straight off disk from a read-only bind mount, so nothing is
built or copied. Host port `3000` → container port `80`, with `nginx.base44.conf`
mounted as the vhost config.

The files are mounted **one by one** (`index.html`, `style.css`, `script.js`,
`assets/`) rather than mounting `./` as the document root, because the checkout
directory is not world-traversable (`drwx------`) in the sandbox: mounting `./`
gives nginx's unprivileged worker `403 Forbidden` on every request. That means a
**new top-level file (e.g. `sobre.html`, a new image folder) also needs a line in
the `volumes:` list** of `docker-compose.base44.yml`, then
`docker compose -f docker-compose.base44.yml up -d`. Files inside the existing
`assets/` folder need no change.

- **No dependencies to install** and no migration/seed step; a fresh clone boots
  as-is.
- **No live-reload watcher.** Because files are served straight from the bind
  mount, an edit is already on disk and appears on the next full page load — but
  the browser will not refresh by itself. After changing any file here, call
  `reload_preview` (or reload the page) to see it.
- `nginx.base44.conf` sends `Cache-Control: no-store` so an edited stylesheet or
  script is never served from cache, and denies dotfile paths as a safety net.

## Verifying it works

```
curl -fsS http://localhost:3000/ | grep -q TaibaBuggyAdventure
curl -fsSI http://localhost:3000/assets/hero.jpg   # 200 + image/jpeg
```

The compose healthcheck runs the first check inside the container (`wget` + `grep`
on real page content, not just an open port), so `docker compose ps` showing
`healthy` already means the site is served.

The page references Google Fonts over the network; without outbound access it
still renders correctly on the system font fallback.
