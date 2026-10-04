# clicker-game

A browser-based clicker game. Open `index.html` or serve this folder with `python3 -m http.server 8000`.

## Shared leaderboard setup

The leaderboard is local until Supabase is configured. To share scores between players:

1. Create a Supabase project.
2. In the Supabase SQL Editor, run this once:

```sql
create table public.clicker_leaderboard (
	username_key text primary key,
	username text not null check (char_length(username) between 1 and 20),
	score bigint not null default 0 check (score >= 0),
	rebirths integer not null default 0 check (rebirths >= 0),
	updated_at timestamptz not null default now()
);

alter table public.clicker_leaderboard enable row level security;

create policy "Anyone can read clicker leaderboard"
	on public.clicker_leaderboard for select to anon using (true);
create policy "Anyone can add clicker scores"
	on public.clicker_leaderboard for insert to anon with check (true);
create policy "Anyone can update clicker scores"
	on public.clicker_leaderboard for update to anon using (true) with check (true);

grant select, insert, update on public.clicker_leaderboard to anon;
```

3. In the Supabase project API settings, copy the **Project URL** and **anon/public key**. In `index.html`, replace `YOUR_PROJECT_ID` in `SUPABASE_URL` and replace `YOUR_SUPABASE_ANON_KEY` in `SUPABASE_ANON_KEY` with those values. Never put a `service_role` key in the game.
4. Publish the updated `index.html` again. The leaderboard will upload scores as players play and fetch the shared top 100 when opened.

This setup is intentionally username-only and publicly writable: anyone can claim a username or alter scores, especially because the game includes client-side cheat commands. Do not use personal or sensitive usernames. A cheat-resistant leaderboard needs authenticated accounts and server-side score validation.
