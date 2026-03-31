# Learning Log — React Router and Cloudflare Pages

## Focus

This file isolates the routing and deployment part of the January 26 work.

---

## SPA routing

React Router handles navigation on the client, but direct access to a route like `/vectors` still needs the server to return `index.html`.

For Cloudflare Pages, the usual fix is:

```text
/* /index.html 200
```

in `public/_redirects`.

---

## Routing example

```jsx
import { Routes, Route, Link } from 'react-router-dom'

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/vectors" element={<Vectors />} />
</Routes>

<Link to="/vectors">Vectors</Link>
```

---

## Wrangler notes

- `wrangler.toml` can define `pages_build_output_dir = "./dist"`
- `wrangler pages deploy` then uses that output directory

Commands used:

```bash
npm run build
wrangler login
wrangler pages project create paul-craft-react
wrangler pages deploy
```
