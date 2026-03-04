# ⚙️ Setup — Cài đặt và cấu hình

## 1. Cài đặt Claude Code

Nếu chưa có Claude Code:

```bash
npm install -g @anthropic-ai/claude-code
```

## 2. Kiểm tra skill đã sẵn sàng

Trong Claude Code terminal:

```
/github-workflow-automation
```

Nếu skill load thành công, bạn sẽ thấy danh sách workflows có sẵn.

## 3. Thêm ANTHROPIC_API_KEY vào GitHub Secrets

Tất cả workflows trong skill này dùng Anthropic API. Cần thêm key:

1. Vào repo GitHub → **Settings**
2. Chọn **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Điền:
   - **Name:** `ANTHROPIC_API_KEY`
   - **Value:** `sk-ant-api03-...` (lấy từ [console.anthropic.com](https://console.anthropic.com))

```yaml
# Trong workflow, key được dùng như sau:
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## 4. Cấu hình permissions cho workflow

Mỗi workflow cần permissions phù hợp. Ví dụ:

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    permissions:
      contents: read        # Đọc code
      pull-requests: write  # Comment vào PR
      issues: write         # Comment vào issues
```

Nếu thiếu permission, workflow sẽ fail với lỗi `403 Forbidden`.

## 5. Enable GitHub Actions

Vào repo → **Settings** → **Actions** → **General** → chọn **Allow all actions**.

## 6. Install @anthropic-ai/sdk trong workflow

Các workflow dùng `actions/github-script@v7`. Để dùng SDK:

```yaml
- name: Install Anthropic SDK
  run: npm install @anthropic-ai/sdk

- name: Run AI step
  uses: actions/github-script@v7
  with:
    script: |
      const { Anthropic } = require('@anthropic-ai/sdk');
      // ...
```

Hoặc dùng `require` inline nếu đã install ở step trước.

## Kiểm tra cài đặt

Sau khi setup xong, tạo thử một PR và xem workflow chạy trong tab **Actions** của repo.
