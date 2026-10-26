# FoodieExpress — Base44 sandbox notes

Plain static multi-page site (`index.html`, `restaurant.html`, `cart.html`,
`confirmation.html` + `style.css`, `app.js`). **There is no build step, no
package manager, no backend, and no env vars or secrets.** Do not add a Node or
Python runtime service: nothing here compiles or needs dependencies.

## Running it

```bash
docker compose -f docker-compose.base44.yml up -d
```

One `web` service: stock `nginx:1.27-alpine` serving the repo bind-mounted
read-only at `/usr/share/nginx/html`, published on host port 3000. Because the
server reads the working tree directly, a page refresh always shows the current
source — there is no watcher to wait on and no image to rebuild after edits.

## Non-obvious quirks

- **nginx workers run as root** (`docker/nginx.base44.main.conf`). The sandbox
  checkout directory is mode `700` owned by root, so the stock image's
  unprivileged `nginx` user cannot traverse the bind mount and every request
  returns 403/404. That main config exists only for this; do not copy it to a
  production deployment.
- `docker/nginx.base44.conf` sets `charset utf-8` (the pages are full of emoji)
  and `Cache-Control: no-store` so refreshes never serve stale CSS/JS.
- The pages are separate documents, so client-side routing helpers do not work.
  Verify by loading each page directly; navigate between them with normal links.
- Cart state lives only in `localStorage` under the key `cart`.

## Verifying

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:3000/   # expect 200
for p in index.html restaurant.html cart.html confirmation.html style.css app.js; do
  printf '%-20s %s\n' "$p" "$(curl -s -o /dev/null -w '%{http_code}' http://localhost:3000/$p)"
done
```

All six must return 200. `docker compose -f docker-compose.base44.yml ps` should
show `web` as `(healthy)`.
