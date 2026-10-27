# Supabase keep-alive

Keeps the free Supabase project **EcoGas App** (`rovsbafnldavsewobdoa`) from being
paused for inactivity. A GitHub Actions job reads one row from the `keepalive`
table every 3 days.

## Setup

1. Add these repo secrets (Settings → Secrets and variables → Actions):
   - `SUPABASE_URL` = `https://rovsbafnldavsewobdoa.supabase.co`
   - `SUPABASE_ANON_KEY` = the project's anon key (Supabase → Project Settings → API)
2. Actions tab → **Supabase keep-alive** → **Run workflow** to test. It should print `HTTP 200`.

## Notes

- The `keepalive` table is read-only for the public key (row level security, select only).
- GitHub disables scheduled workflows after 60 days with no commits in the repo.
  If that happens, re-enable it from the Actions tab or push any commit.
