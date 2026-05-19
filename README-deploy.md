# Hướng dẫn Deploy lên Cloudflare Pages

App dùng **TanStack Start + Cloudflare Workers** — cần deploy lên **Cloudflare Pages**, không phải hosting Node.js thông thường.

## Tại sao phải Cloudflare Pages?

- `@cloudflare/vite-plugin` build output là Cloudflare Worker (edge runtime)
- Tenten / VPS Node.js không tương thích với runtime này
- Cloudflare Pages miễn phí với 100,000 requests/ngày

---

## Bước 1 — Chuẩn bị repo GitHub

```bash
# Push code lên GitHub (bình thường)
git init
git add .
git commit -m "init"
git remote add origin https://github.com/your-user/your-repo.git
git push -u origin main
```

> **Lưu ý:** File `.env` đã bị bỏ khỏi git. Không bao giờ commit key thật.

---

## Bước 2 — Tạo project trên Cloudflare Pages

1. Vào [dash.cloudflare.com](https://dash.cloudflare.com) → **Workers & Pages** → **Create**
2. Chọn **Pages** → **Connect to Git** → chọn repo GitHub
3. **Build settings:**
   - Framework preset: `None` (hoặc `Vite`)
   - Build command: `npm install && npm run build`
   - Build output directory: `.output/public` (TanStack Start với Cloudflare output vào đây)

---

## Bước 3 — Đặt Environment Variables

Trong Cloudflare Pages → **Settings** → **Environment variables** → **Add variable**:

| Tên biến | Giá trị | Môi trường |
|---|---|---|
| `SUPABASE_URL` | `https://xxxx.supabase.co` | Production + Preview |
| `SUPABASE_PUBLISHABLE_KEY` | `eyJ...` (anon key) | Production + Preview |
| `SUPABASE_SERVICE_ROLE_KEY` | `eyJ...` (service_role) | Production + Preview |
| `VITE_SUPABASE_URL` | `https://xxxx.supabase.co` | Production + Preview |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | `eyJ...` (anon key) | Production + Preview |
| `VITE_SUPABASE_PROJECT_ID` | `xxxx` | Production + Preview |
| `BEEKNOEE_API_KEY` | `sk-bee-...` | Production + Preview |

> Nếu không có Beeknoee, thay bằng `GEMINI_API_KEY` = `AIza...`

---

## Bước 4 — Deploy

Nhấn **Save and Deploy**. Cloudflare tự build từ GitHub.

Mỗi lần push lên `main` → tự động redeploy.

---

## Cấu hình AI key trong Admin panel

Ngoài env var cố định, app còn cho phép admin đặt key động qua DB:

1. Đăng nhập tài khoản admin
2. Vào `/admin` → phần **AI Key**
3. Nhập Beeknoee key (`sk-bee-...`) hoặc Gemini key (`AIza...`)
4. Key DB sẽ được ưu tiên hơn env var

---

## Cấu trúc AI fallback

```
Key từ DB admin (getSharedAiKey)
  → Beeknoee (sk-bee-* / sk-*)
  → Gemini trực tiếp (AIza*)

Nếu không có key DB:
  → BEEKNOEE_API_KEY (env)
  → GEMINI_API_KEY (env)
```

Models thử theo thứ tự: `gemini-2.5-flash-lite` → `gemini-2.5-flash` → `gemini-2.0-flash`
