# Heroku Buildpack: Next.js Standalone Slim

This buildpack **slims your Heroku slug** for a Next.js app built with `output: 'standalone'`.

Result: a **smaller slug size** when deploying a Next.js app to Heroku. For example, in the **[strapi-next-monorepo-starter](https://github.com/notum-cz/strapi-next-monorepo-starter)** repo it reduced the slug size from ~480 MB to ~110 MB (~77% reduction).

It is designed to run **after** `heroku/nodejs`. It keeps only what’s needed at runtime:
- Next.js **standalone server output**
- `.next/static` and `public` assets (if present)
- Heroku runtime/buildpack folders (e.g. `.heroku`, `.profile.d`)

## Installation on Heroku

Add buildpacks **in this order**:

0. (optional) `https://github.com/notum-cz/heroku-buildpack-turbo-prune` (additional buildpack when deploying a Turborepo with pruning)
---
1. `heroku/nodejs`
2. `https://github.com/notum-cz/heroku-buildpack-next-standalone-slim`

## How it works

[Heroku’s Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs) installs dependencies and runs your build (e.g. `next build`/`heroku-postbuild`).

Then this buildpack:

1. Removes everything except the [Next.js standalone output](https://nextjs.org/docs/pages/api-reference/config/next-config-js/output#automatically-copying-traced-files) and runtime-required files:
   - `apps/$APP/.next/standalone` (required)
   - `apps/$APP/.next/static` (optional)
   - `apps/$APP/public` (optional)
   - Heroku runtime/buildpack folders: `.heroku/`, `.profile.d/`
   - `Procfile` and `app.json` if present at the repo root
2. Promotes the standalone output to the build root (like a Docker “runner stage”)
3. Ensures a `Procfile` exists:
   - If you provided one at repo root, it keeps it
   - Otherwise it creates with: `web: node apps/$APP/server.js`

## Requirements

- Next.js app with `output: 'standalone'` enabled (so `server.js` is generated)
- `next build` must run before this buildpack (use `heroku/nodejs` first)
- Your apps live under `apps/<app>` (e.g. `apps/ui`, `apps/strapi`)

## Configuration

### Required config vars

These are set **per Heroku app/dyno** as Heroku config vars:

- `APP` – the app folder name under `apps/` - e.g. `ui`, `strapi`