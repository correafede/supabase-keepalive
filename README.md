# Supabase keep-alive

Keeps the free Supabase project **EcoGas App** (`rovsbafnldavsewobdoa`) from being
paused for inactivity. A GitHub Actions job runs **every 4 hours** (6 times a day) and:

1. reads one row from the `keepalive` table, and
2. calls the `keepalive_ping()` database function, which writes the current
   time to that row (`last_ping`) and returns it.

A read every 3 days was not enough (paused 2026-11-16), and neither was a read
and write once a day (paused 2026-11-20). Supabase's docs say "a few user
requests to the database each day over the previous week" is typically enough,
and GitHub often delays or skips scheduled runs, hence every 4 hours.

## Setup

1. Add these repo secrets (Settings → Secrets and variables → Actions):
   - `SUPABASE_URL` = `https://rovsbafnldavsewobdoa.supabase.co`
   - `SUPABASE_ANON_KEY` = the project's anon key (Supabase → Project Settings → API)
2. The database needs the `keepalive_ping()` function (migration
   `keepalive_ping_function`, already applied):

   ```sql
   alter table public.keepalive add column if not exists last_ping timestamptz;

   create or replace function public.keepalive_ping()
   returns timestamptz
   language sql
   security definer
   set search_path = ''
   as $$
     update public.keepalive set last_ping = now() where id = 1 returning last_ping;
   $$;

   revoke all on function public.keepalive_ping() from public;
   grant execute on function public.keepalive_ping() to anon, authenticated;
   ```

   The function can only touch that one timestamp, so the public key still
   cannot change anything else (the table itself stays select-only).
3. Actions tab → **Supabase keep-alive** → **Run workflow** to test. Both steps
   should print `HTTP 200`.

## Auto-restore if paused

If the `SUPABASE_ACCESS_TOKEN` secret is set, every run first checks the
project's status through the Supabase Management API. If the project is paused,
it calls `POST /v1/projects/{ref}/restore`, waits until the project is healthy,
and then pings as usual. So a pause lasts at most about 4 hours.

To turn it on:

1. Create a personal access token at
   https://supabase.com/dashboard/account/tokens (name it e.g. `github-keepalive`).
2. Add it as a repo secret named `SUPABASE_ACCESS_TOKEN`:

   ```bash
   gh secret set SUPABASE_ACCESS_TOKEN -R correafede/supabase-keepalive
   ```

   (paste the token when asked).

The token has full access to your Supabase account, so keep it only as a
secret. The workflow never prints it, and secrets are not passed to workflows
triggered by pull requests from forks.

## Notes

- GitHub disables scheduled workflows after 60 days with no commits in the repo.
  To prevent that, the job commits `last-heartbeat.txt` whenever the last commit
  is 25+ days old. If it still gets disabled, re-enable it from the Actions tab.
- The only guaranteed way to avoid pausing is Supabase's Pro plan.
