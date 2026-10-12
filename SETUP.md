# 告白长信：共享内容配置说明

已经新增独立编辑页：`editor.html`。它通过 Supabase Auth 登录，并把正文写入 Supabase 数据库。要让不同设备看到同一份内容，需要完成下面的一次性配置。

## 1. 创建 Supabase 项目
打开 https://supabase.com/ ，创建一个项目。在 Project Settings / API 中复制 Project URL 和公开的 publishable key（旧项目可能显示 anon key）。**不要把 service_role 或 secret key 放进网页。**

## 2. 创建数据库表
在 Supabase 的 SQL Editor 中执行：

```sql
create table if not exists public.letter_content (
  id integer primary key check (id = 1),
  title text not null default '有些话，想认真说完',
  body text not null default '',
  updated_at timestamptz not null default now()
);

alter table public.letter_content enable row level security;

-- 公开页面只允许读取已发布信件
create policy "Public can read letter"
on public.letter_content for select
to anon, authenticated
using (true);

-- 仅登录用户可以写入；不要创建 anon 的 insert/update/delete 策略
create policy "Authenticated users can insert letter"
on public.letter_content for insert
to authenticated
with check (true);

create policy "Authenticated users can update letter"
on public.letter_content for update
to authenticated
using (true)
with check (true);

grant select on public.letter_content to anon, authenticated;
grant insert, update on public.letter_content to authenticated;
```

## 3. 创建自己的编辑账号
在 Supabase 的 Authentication / Users 中添加一个你自己掌握的用户邮箱和密码。公开访客不需要这个账号。不要在网页里硬编码密码。

## 4. 配置编辑页
打开网站的 `editor.html`，输入 Project URL 和公开客户端 key，点击“连接共享存储”，再用上一步创建的账号登录。

## 5. 公开页面读取最新正文
`index.html` 需要配置同一组 Project URL / publishable key，并从 `letter_content` 表读取 id=1 的 title/body。未完成这一步前，原有首页仍然显示旧版静态信件。请勿在公开页面显示编辑控件。

## 注意
当前编辑页的配置保存在使用它的浏览器 localStorage；数据库里的正文才是跨设备共享内容。请妥善保管编辑账号。当前 RLS 写策略允许任何已认证用户写入；如果项目将来开放注册，应把写策略收紧到指定用户 UUID。
