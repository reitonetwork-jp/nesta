# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

NESTA is a Next.js 16 + Supabase application, bootstrapped from the official Next.js/Supabase starter template (App Router, `@supabase/ssr`, shadcn/ui, Tailwind CSS).

## Commands

```bash
npm run dev      # start dev server (localhost:3000)
npm run build    # production build
npm run start    # run production build
npm run lint     # eslint (flat config: next/core-web-vitals + next/typescript)
```

There is no test suite configured in this repo.

## Environment

Copy `.env.example` to `.env.local` and set:

```
NEXT_PUBLIC_SUPABASE_URL=<supabase project url>
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=<supabase anon/publishable key>
```

`lib/utils.ts` exports `hasEnvVars`, which several components (`EnvVarWarning`, the proxy) check to detect an unconfigured Supabase project and degrade gracefully rather than crash.

## Architecture

**Auth/session flow (three Supabase client entry points, do not conflate them):**
- `lib/supabase/client.ts` — browser client (`createBrowserClient`), for use in Client Components.
- `lib/supabase/server.ts` — server client (`createServerClient`) that reads/writes cookies via `next/headers`; must be created fresh per request/function call (never module-scoped) because of Fluid compute.
- `lib/supabase/proxy.ts` — `updateSession()`, used by the request proxy to refresh the auth session on every request and redirect unauthenticated users away from non-public routes.

**Request proxy:** `proxy.ts` at the repo root is this project's equivalent of Next.js middleware — it exports `proxy()` (not `middleware()`), which is Next 16's renamed convention, and delegates to `updateSession()`. The `config.matcher` excludes static assets and images. Route protection logic (which paths require auth) lives inside `updateSession` in `lib/supabase/proxy.ts`, not in `proxy.ts` itself.

**Auth pages/flows** live under `app/auth/*` (login, sign-up, forgot-password, update-password, error, sign-up-success) paired with matching form components in `components/*-form.tsx`. Email OTP/link confirmation is handled by the route handler `app/auth/confirm/route.ts`, which calls `supabase.auth.verifyOtp` and redirects to `next` (or `/auth/error`).

**Protected area:** `app/protected/` is gated by the proxy redirect; `app/protected/layout.tsx` renders the shared nav/footer chrome (uses `Suspense` around `AuthButton` since it reads auth state), and `app/protected/page.tsx` fetches the user via `supabase.auth.getClaims()` server-side, redirecting to `/auth/login` on failure.

**UI components:** `components/ui/` holds shadcn/ui primitives (style: "new-york", base color "neutral", icon library "lucide") generated via `components.json`; treat these as generated/vendored and prefer composing over hand-editing. Non-primitive shared components sit directly in `components/`. `components/tutorial/` are template-provided onboarding widgets, not application features.

**Styling:** Tailwind CSS v3 with `tailwindcss-animate`; theme toggling via `next-themes` (`components/theme-switcher.tsx`), wired up in `app/layout.tsx`'s `ThemeProvider`.

**Path alias:** `@/*` maps to the repo root (see `tsconfig.json` / `components.json` aliases).

**Next.js config:** `next.config.ts` enables `cacheComponents: true` (Next 16 caching model) — be aware of this when adding data fetching, as it affects caching/revalidation semantics for Server Components.
# NESTA 開発ルール

## 1. プロジェクト概要

NESTA（ネスタ）は、カテゴリーを限定しないX型の総合SNSです。

コンセプト：

> 好きなことから、人とつながる。

ユーザーは、日常・趣味・ゲーム・音楽・映画・スポーツ・AI・プログラミング・仕事・学習・旅行・ファッション・食事・ペット・雑談など、カテゴリーに関係なく自由に投稿できます。

NESTAは「カテゴリー」よりも「人」を中心にしたSNSを目指します。

---

## 2. 技術スタック

### Frontend

* Next.js
* React
* TypeScript
* Tailwind CSS

### Backend

* Next.js Server Actions
* Next.js Route Handlers
* Supabase

### Database

* Supabase PostgreSQL

### Authentication

* Supabase Auth
* Email / Password
* Google OAuth
* Discord OAuth

### Storage

* Supabase Storage

### Realtime

* Supabase Realtime

### Hosting

* Vercel

### Payment

* Stripe

### AI

* 将来的にLLM APIを利用

### Development

* Git
* GitHub
* Claude Code

---

## 3. 開発方針

NESTAは一度にすべての機能を作らない。

必ず段階的に開発する。

### MVP

1. 独自ユーザー登録
2. 独自ログイン
3. Googleログイン
4. Discordログイン
5. プロフィール
6. 投稿作成
7. 投稿削除
8. タイムライン
9. いいね
10. フォロー
11. コメント
12. 通知
13. ユーザー検索
14. 投稿検索
15. おすすめユーザー
16. レスポンシブUI

### Phase 2

1. 再投稿
2. 引用投稿
3. ブックマーク
4. ハッシュタグ
5. トレンド
6. 高度な検索

### Phase 3

1. DM
2. ブロック
3. ミュート
4. 通報
5. メッセージ通知

### Phase 4

1. NESTA Pro
2. Stripe
3. サブスクリプション
4. 広告
5. 公式アカウント
6. クリエイター機能

### Phase 5

1. AIおすすめ
2. AI検索
3. AI要約
4. 投稿作成支援
5. AIモデレーション

---

# 4. 最重要開発ルール

## 4.1 既存コードを必ず確認する

実装前に必ず、

* ディレクトリ構造
* package.json
* 既存コンポーネント
* Supabase設定
* 認証処理
* DB migration
* RLS
* 型定義

を確認する。

既存機能を壊さない。

---

## 4.2 一度に大量実装しない

ユーザーから指定されたPhase・機能だけを実装する。

勝手に将来機能を実装しない。

特に、

* DM
* 課金
* 広告
* AI
* 再投稿
* トレンド

などをMVPに勝手に追加しない。

---

## 4.3 実装前に計画する

複数ファイルに影響する変更の場合、

1. 現状確認
2. 変更内容
3. 影響範囲
4. DB変更の有無
5. セキュリティへの影響

を確認してから実装する。

---

# 5. Supabaseルール

## 5.1 Supabase SSR

Next.jsではSupabaseのSSR構成を使用する。

基本的に、

* `@supabase/supabase-js`
* `@supabase/ssr`

を使用する。

古いSupabase Auth Helpersは使用しない。

Supabase公式でもAuth Helpersから`@supabase/ssr`への移行が推奨されている。

---

## 5.2 Clientを分離する

基本構成：

```text
lib/
└── supabase/
    ├── client.ts
    ├── server.ts
    └── proxy.ts
```

### client.ts

Browser / Client Components用。

### server.ts

Server Components、Server Actions、Route Handlers用。

### proxy.ts

認証セッションのCookie更新・同期用。

---

## 5.3 Environment Variables

使用する環境変数：

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
```

`.env.local`から読み込む。

秘密情報をコードに直接記述しない。

---

## 5.4 Environment Variablesの安全性

絶対にソースコードへ直接記述しない：

* Supabase Secret Key
* Service Role Key
* Database Password
* Stripe Secret Key
* OAuth Client Secret
* AI API Secret Key

ログにも秘密情報を出力しない。

---

# 6. Supabase Auth

認証はSupabase Authを使用する。

MVPでは：

* Email / Password
* Google
* Discord

を実装する。

---

## 6.1 認証情報とプロフィールを分離する

Supabase Authのユーザー情報とNESTAのプロフィール情報を混同しない。

認証：

```text
auth.users
```

NESTAプロフィール：

```text
public.profiles
```

を使用する。

---

## 6.2 profiles

基本構造：

```text
profiles
├── id
├── username
├── display_name
├── bio
├── avatar_url
├── website_url
├── is_private
├── created_at
└── updated_at
```

`profiles.id`は`auth.users.id`と対応させる。

---

# 7. Databaseルール

DB変更は必ずmigrationとして管理する。

直接本番DBだけを書き換えない。

例：

```text
supabase/
└── migrations/
    ├── 001_create_profiles.sql
    ├── 002_create_posts.sql
    └── ...
```

migrationはGitにコミットする。

---

# 8. Row Level Security

SupabaseのテーブルではRLSを基本的に有効化する。

RLSなしで本番公開しない。

基本原則：

### profiles

ユーザーは自分のプロフィールを編集できる。

### posts

ユーザーは自分の投稿を削除できる。

### likes

ユーザーは自分のいいねだけ作成・削除できる。

### follows

ユーザーは自分のフォロー関係を作成・削除できる。

### notifications

ユーザー本人だけが自分の通知を閲覧できる。

### bookmarks

ユーザー本人だけが自分のブックマークを閲覧できる。

### messages

会話参加者だけがメッセージを閲覧できる。

---

# 9. Server側の権限確認

クライアント側だけで権限チェックしない。

重要な処理では必ずServer側でも、

* 現在のユーザー
* 対象リソース
* 操作権限

を確認する。

例：

投稿削除では、

```text
現在のuser_id === posts.user_id
```

をServer側でも確認する。

---

# 10. NESTA Database

主要テーブル：

```text
profiles

posts
post_media

follows
likes
comments

notifications

hashtags
post_hashtags

bookmarks

conversations
conversation_members
messages

blocks
mutes

reports

subscriptions
```

将来的に必要に応じて、

```text
reposts
mentions
user_interests
trends
advertisements
creator_profiles
analytics_events
```

を追加する。

---

# 11. 投稿仕様

MVP：

* 最大280文字
* テキスト投稿
* 画像投稿
* 投稿削除
* コメント
* 返信

将来的に：

* 再投稿
* 引用投稿
* 動画
* GIF
* 長文
* 投稿編集
* 投票

を追加する。

---

# 12. Timeline

MVPでは以下を提供する。

```text
おすすめ
フォロー中
最新
```

基本的な投稿取得ではCursor Paginationを使用する。

OFFSET Paginationを大量データのタイムラインに使用しない。

基本的に、

```text
created_at
+
id
```

などを利用したCursor Paginationを優先する。

---

# 13. Social機能

MVP：

* Follow
* Unfollow
* Like
* Unlike
* Comment

Phase 2：

* Repost
* Quote
* Bookmark
* Hashtag
* Trend

Phase 3：

* DM
* Block
* Mute
* Report

---

# 14. 通知

MVP：

* Follow
* Like
* Comment
* Reply

将来的に：

* Repost
* Quote
* Mention
* System

を追加する。

未読数を管理する。

---

# 15. UI / UX

NESTAはPCとスマートフォンの両方に対応する。

### PC

3カラムを基本とする。

```text
左：ナビゲーション
中央：タイムライン
右：おすすめ・トレンド
```

### Mobile

ボトムナビゲーションを基本とする。

```text
ホーム
検索
投稿
通知
プロフィール
```

---

# 16. UI状態

すべての非同期UIには適切な状態を用意する。

必須：

* Loading
* Error
* Empty
* Success

例：

投稿一覧が空の場合、

```text
まだ投稿がありません
```

などのEmpty Stateを表示する。

---

# 17. TypeScript

TypeScript strictを維持する。

可能な限り`any`を使用しない。

DB型はSupabaseから生成した型を利用する。

型エラーを無視しない。

---

# 18. Component設計

コンポーネントは責務を分離する。

巨大なコンポーネントを作らない。

例：

```text
components/
├── post/
│   ├── PostCard.tsx
│   ├── PostComposer.tsx
│   └── PostActions.tsx
├── profile/
├── timeline/
├── notification/
└── layout/
```

実際の構造は既存コードを確認して適切に判断する。

---

# 19. セキュリティ

必ず考慮する：

* RLS
* Authentication
* Authorization
* XSS
* CSRF
* SQL Injection
* Rate Limit
* Abuse
* File Upload Validation
* MIME Type Validation
* File Size Limit
* URL Validation
* Admin Permission
* Secrets Protection

ユーザー入力を信頼しない。

---

# 20. File Upload

Supabase Storageを使用する。

アップロード時：

* ファイルサイズを確認
* MIME Typeを確認
* 拡張子を確認
* 許可された形式のみ受け付ける
* ファイル名をユーザー入力のまま信用しない

画像URLを直接DBに保存する場合も、所有者とアクセス権を考慮する。

---

# 21. Rate Limit

将来的に必ず導入する。

対象：

* Login
* Signup
* Post
* Comment
* Like
* Follow
* Search
* DM
* Report

スパム・Bot対策を考慮する。

---

# 22. エラーハンドリング

エラーを握りつぶさない。

ユーザーには安全なエラーメッセージを表示し、秘密情報や内部情報を公開しない。

Server Logsには必要な情報だけ記録する。

---

# 23. Testing

重要機能にはテストを追加する。

特に：

* Authentication
* Authorization
* RLS
* Post Creation
* Post Deletion
* Follow
* Like
* Comment
* Notification

を重点的に確認する。

---

# 24. Git

Git管理を徹底する。

基本ルール：

```text
feature
↓
implementation
↓
test
↓
commit
```

大きな変更を1コミットに詰め込まない。

コミットメッセージは変更内容が分かるものにする。

例：

```text
feat: add profile schema
feat: add post creation
fix: prevent duplicate likes
```

---

# 25. Gitにコミットしないもの

以下を絶対にコミットしない。

```text
.env
.env.local
.env.*.local
```

また、

* API Secret
* Password
* Token
* OAuth Secret
* Service Role Key
* Stripe Secret
* AI API Secret

をGitに保存しない。

---

# 26. .env.example

`.env.example`はGit管理する。

実際の秘密情報は入れない。

例：

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=
```

新しい環境変数を追加した場合は、必要に応じて`.env.example`も更新する。

---

# 27. Claude Codeの実装手順

新しい機能を実装するときは、必ず以下の順序で進める。

### Step 1

既存コードを調査する。

### Step 2

関連ファイルを特定する。

### Step 3

DB変更の有無を確認する。

### Step 4

RLS変更の必要性を確認する。

### Step 5

実装計画を作る。

### Step 6

実装する。

### Step 7

型チェック。

### Step 8

Lint。

### Step 9

関連テスト。

### Step 10

Git diff確認。

### Step 11

変更内容を報告する。

---

# 28. DB変更時のルール

DB変更が必要な場合：

1. migrationを作成
2. SQLを確認
3. RLSを設定
4. Indexを確認
5. Foreign Keyを確認
6. Unique Constraintを確認
7. migrationを適用
8. 動作確認
9. Gitにコミット

DB変更をアプリコードだけで済ませようとしない。

---

# 29. Performance

SNSなので大量データを想定する。

考慮事項：

* Index
* Cursor Pagination
* Query最適化
* 必要な列だけ取得
* N+1 Query回避
* Image Optimization
* Lazy Loading
* Cache
* Realtimeの使いすぎ防止

タイムラインは特に最適化する。

---

# 30. 検索

MVP：

* ユーザー検索
* 投稿検索

Phase 2：

* Hashtag
* Date
* User
* Popular
* Latest

大量データになった場合はPostgreSQLのFull Text Searchなどを検討する。

---

# 31. Monetization

収益化はユーザー体験を壊さない。

将来的に：

```text
NESTA Pro
広告
公式アカウント
クリエイター機能
AI機能
```

を実装する。

MVPでは課金・広告を実装しない。

---

# 32. NESTA Pro

将来的な候補：

* 広告非表示
* 投稿編集
* 長文投稿
* 高画質メディア
* 投稿予約
* 高度な検索
* プロフィールカスタマイズ
* Proバッジ
* 投稿分析
* AI利用枠増加

価格はユーザー調査・利用状況を確認してから決定する。

---

# 33. AI

AI機能はPhase 5以降。

候補：

* AIおすすめ
* AI検索
* 投稿要約
* スレッド要約
* 投稿作成支援
* AIモデレーション

AIだけでユーザーの投稿を無条件に削除しない。

モデレーションではルール、通報、人間による確認などを組み合わせる。

---

# 34. Privacy

個人情報を必要以上に収集しない。

ユーザーデータへのアクセスは最小権限にする。

削除・退会時のデータ処理も将来的に設計する。

---

# 35. アカウント削除

将来的にユーザー自身がアカウントを削除できるようにする。

削除時には、

* Auth user
* Profile
* 投稿
* コメント
* フォロー
* いいね
* 通知
* DM
* その他関連データ

の扱いを明確にする。

CASCADEを使用する場合は意図しないデータ削除が発生しないよう慎重に設計する。

---

# 36. 管理者機能

将来的に管理画面を作る。

権限：

```text
user
moderator
admin
super_admin
```

管理対象：

* Users
* Posts
* Comments
* Reports
* Ads
* Subscriptions
* Trends
* System Settings

管理者権限は必ずServer側・DB側で保護する。

---

# 37. NESTAの開発優先順位

最優先：

```text
Authentication
↓
Profiles
↓
Posts
↓
Timeline
↓
Follow
↓
Like
↓
Comment
↓
Notification
```

次：

```text
Search
↓
Recommendation
↓
Repost
↓
Quote
↓
Bookmark
↓
Hashtag
↓
Trend
```

その後：

```text
DM
↓
Block
↓
Mute
↓
Report
```

収益化：

```text
NESTA Pro
↓
Stripe
↓
Ads
↓
Official Accounts
```

最後に：

```text
AI
```

---

# 38. 現在の開発ステータス

現在は初期セットアップ段階。

完了：

* Next.js
* TypeScript
* Tailwind CSS
* Supabase Project
* Supabase URL
* Supabase Publishable Key
* `.env.local`
* `.gitignore`
* Git
* GitHub
* Claude Code

未実装：

* Auth
* Profiles
* Posts
* Timeline
* Follow
* Like
* Comment
* Notifications

---

# 39. 現在の最優先タスク

次に実装するのは：

## Phase 1 - Authentication

順番：

1. Supabase Auth設定確認
2. Email Signup
3. Email Login
4. Logout
5. Password Reset
6. Auth Callback
7. profilesテーブル
8. 新規ユーザーのprofiles作成
9. RLS
10. 認証済みページ
11. 未認証ページ
12. 認証状態確認
13. テスト

Google OAuthとDiscord OAuthはEmail認証が安定してから追加する。

---

# 40. Claude Codeへの最重要ルール

ユーザーが明示していない機能を勝手に追加しない。

既存コードを大きく変更する前に確認する。

DBを変更する前にmigrationとRLSを考える。

秘密情報を表示しない。

エラーを隠さない。

型安全性を維持する。

セキュリティを最優先する。

パフォーマンスを考慮する。

スマートフォンでも使えるUIにする。

NESTAのコンセプトから逸脱しない。

---

# 41. 実装完了時の報告形式

各タスク終了時には以下を報告する。

```text
【実装内容】

【変更ファイル】

【DB変更】

【RLS変更】

【テスト結果】

【Lint / Typecheck】

【Git Diff確認】

【残課題】

【次に推奨する作業】
```

---

# 42. 重要な原則

NESTAは「機能数を増やすこと」が目的ではない。

目的は、

> ユーザーが登録する
> ↓
> 投稿する
> ↓
> 人とつながる
> ↓
> 楽しい
> ↓
> また戻ってくる

という体験を作ること。

常にユーザー体験・安全性・パフォーマンス・拡張性のバランスを考えて実装する。
