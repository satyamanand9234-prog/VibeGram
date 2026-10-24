create table public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  full_name text not null default 'VibeGram User',
  username text unique not null,
  avatar_url text,
  created_at timestamptz default now()
);

create table public.posts (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  caption text default '',
  media_url text not null,
  media_type text not null check (media_type in ('image','video')),
  created_at timestamptz default now()
);

create table public.likes (
  post_id uuid references public.posts(id) on delete cascade,
  user_id uuid references auth.users(id) on delete cascade,
  created_at timestamptz default now(),
  primary key (post_id, user_id)
);

create table public.comments (
  id uuid primary key default gen_random_uuid(),
  post_id uuid references public.posts(id) on delete cascade,
  user_id uuid references auth.users(id) on delete cascade,
  body text not null check (char_length(body) between 1 and 1000),
  created_at timestamptz default now()
);

alter table public.profiles enable row level security;
alter table public.posts enable row level security;
alter table public.likes enable row level security;
alter table public.comments enable row level security;

create policy "Read profiles"
on public.profiles for select
to anon, authenticated using (true);

create policy "Create own profile"
on public.profiles for insert
to authenticated
with check (auth.uid() = id);

create policy "Update own profile"
on public.profiles for update
to authenticated
using (auth.uid() = id)
with check (auth.uid() = id);

create policy "Read posts"
on public.posts for select
to anon, authenticated using (true);

create policy "Create own posts"
on public.posts for insert
to authenticated
with check (auth.uid() = user_id);

create policy "Read likes"
on public.likes for select
to anon, authenticated using (true);

create policy "Create own likes"
on public.likes for insert
to authenticated
with check (auth.uid() = user_id);

create policy "Remove own likes"
on public.likes for delete
to authenticated
using (auth.uid() = user_id);

create policy "Read comments"
on public.comments for select
to anon, authenticated using (true);

create policy "Create own comments"
on public.comments for insert
to authenticated
with check (auth.uid() = user_id);
