# Sample Application for Flutter and Supabase

SupabaseとFlutterで開発したサンプル版WEBアプリ。

本リポジトリで行うことは次の通り。

* マジックリンクを使ってSupabase Authでユーザにサインインする
* Supabaseデータベースのデータ保存と取得

# セットアップ

## 新規プロジェクト作成

1. [app.supabase.com](app.supabase.com)にアクセスする
2. 「New Project」をクリック
3. プロジェクトの詳細を入力
4. 新規データベースが出来上がるのをそのまま待機する

## データベースの新規作成

1. 「SQL」をクリック
2. 「User Management Starter」をクリック
3. 「Run」をクリック

▼使用するSQLファイル

```sql
-- Create a table for public "profiles"
create table profiles (
  id uuid references auth.users not null,
  updated_at timestamp with time zone,
  username text unique,
  avatar_url text,
  website text,

  primary key (id),
  unique(username),
  constraint username_length check (char_length(username) >= 3)
);

alter table profiles enable row level security;

create policy "Public profiles are viewable by everyone."
  on profiles for select
  using ( true );

create policy "Users can insert their own profile."
  on profiles for insert
  with check ( auth.uid() = id );

create policy "Users can update own profile."
  on profiles for update
  using ( auth.uid() = id );

-- Set up Realtime!
begin;
  drop publication if exists supabase_realtime;
  create publication supabase_realtime;
commit;
alter publication supabase_realtime add table profiles;

-- Set up Storage!
insert into storage.buckets (id, name, public)
values ('avatars', 'avatars', true);

create policy "Anyone can upload an avatar."
  on storage.objects for insert
  with check ( bucket_id = 'avatars' );
```

## APIキーの取得

1. 「Setting」をクリック
2. サイドバーにある「API」をクリック
3. APIのURLと「anon」のキーが発行される