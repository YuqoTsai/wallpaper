# 蘿蔔鳥手機桌布網頁 — 設定說明

裡面有 5 個檔案：
- `index.html` — 網頁本體
- `robirds-trio.png` / `robirds-hotpot.png` / `robirds-trio-quote.png` / `robirds-hotpot-quote.png` — 4 張桌布

## 一、上架網頁（GitHub Pages，免費）

1. 到 github.com 建一個新的 public repository，例如 `robirds-wallpapers`。
2. 把這個資料夾裡的 5 個檔案全部上傳進去（repo 根目錄，不要放進子資料夾）。
3. 進 repo 的 **Settings → Pages**，Source 選 `main` branch、`/ (root)`，儲存。
4. 等 1-2 分鐘，會拿到一個網址，例如：
   `https://你的帳號.github.io/robirds-wallpapers/`
   這就是可以分享出去的下載頁面，手機、電腦都能開。

## 二、設定下載次數統計（Supabase，免費）

### 1. 建立專案
到 [supabase.com](https://supabase.com) 註冊、新增一個 Project（免費方案即可）。

### 2. 建立資料表與函式
進專案的 **SQL Editor**，貼上並執行：

```sql
-- 建立統計表
create table downloads (
  id bigint generated always as identity primary key,
  image_name text unique not null,
  count bigint not null default 0
);

-- 先放入 4 張圖的初始資料
insert into downloads (image_name, count) values
  ('robirds-trio', 0),
  ('robirds-hotpot', 0),
  ('robirds-trio-quote', 0),
  ('robirds-hotpot-quote', 0);

-- 開啟資料表安全性（RLS），預設不開放任何人直接讀寫
alter table downloads enable row level security;

-- 建立一個「只能 +1」的函式，給網頁呼叫用
create or replace function increment_download(img text)
returns void
language sql
security definer
as $$
  update downloads set count = count + 1 where image_name = img;
$$;

-- 允許匿名使用者「執行」這個函式（但仍看不到、改不了資料表本身）
grant execute on function increment_download(text) to anon;
```

這樣設計的好處：訪客的瀏覽器只能「呼叫函式讓數字 +1」，沒辦法直接讀取或竄改 `downloads` 表，你的統計數字很安全。

### 3. 取得 API 金鑰
在 Supabase 專案左側 **Project Settings → API**，複製：
- **Project URL**
- **anon public** key

貼到 `index.html` 裡這兩行（搜尋 `YOUR_SUPABASE_URL`）：

```js
const SUPABASE_URL = 'https://xxxxxxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOi...（你的 anon key）';
```

存檔後，重新上傳到 GitHub（覆蓋原本的 `index.html`）即可生效。

### 4. 查看下載次數（你的「後台」）
不用另外做管理頁面，直接登入 Supabase → 左側 **Table Editor** → `downloads` 表，就能即時看到每張圖片被下載幾次，隨時可以重新整理查看最新數字。

---

## 之後想換圖片？
把新的 png 檔案上傳取代舊檔（檔名要跟 `index.html` 裡 `<img src="...">` 對應的檔名一致），如果是全新的桌布，記得同時：
1. 在 `index.html` 複製一組 `<article class="card">...</article>`，改成新圖片的檔名與標題。
2. 到 Supabase 的 `downloads` 表新增一筆對應 `image_name` 的資料，才能開始計數。
