# ✅ Best Practices — Security, Tips & Patterns

## 🔒 Security

### 1. Luôn dùng GitHub Secrets cho API keys

```yaml
# ✅ Đúng
env:
  ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

# ❌ Sai — KHÔNG BAO GIỜ hardcode key
env:
  ANTHROPIC_API_KEY: "sk-ant-api03-..."
```

### 2. Minimal permissions

```yaml
permissions:
  contents: read        # Chỉ read code, không write
  pull-requests: write  # Chỉ write vào PR, không repo
  issues: write         # Chỉ comment, không delete
  # KHÔNG dùng permissions: write-all
```

### 3. Validate inputs từ issue/PR

```javascript
// Sanitize input trước khi đưa vào AI prompt
const title = issue.title.slice(0, 500);   // Giới hạn độ dài
const body = (issue.body || '').slice(0, 2000);
```

### 4. Giới hạn diff size

```bash
# Tránh token limit và chi phí cao
diff=$(git diff origin/${{ github.base_ref }}...HEAD | head -c 50000)
```

---

## ⚡ Performance

### Cache dependencies

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

- run: npm ci
```

### Path filters — Chỉ trigger khi cần

```yaml
on:
  pull_request:
    paths:
      - 'src/**'          # Chỉ khi source code thay đổi
      - '!docs/**'        # Bỏ qua docs
      - '!*.md'           # Bỏ qua markdown
```

### Chọn model phù hợp với tác vụ

| Model | Dùng cho | Chi phí |
|-------|----------|---------|
| `claude-haiku-4-5-20251001` | Issue triage, simple tasks | $$  |
| `claude-sonnet-4-6` | PR review, code analysis | $$$$ |
| `claude-opus-4-6` | Complex architecture review | $$$$$ |

---

## 🛡️ Reliability

### Timeout cho jobs

```yaml
jobs:
  review:
    timeout-minutes: 10   # Tránh job chạy mãi
```

### Xử lý lỗi parse JSON từ AI

```javascript
let analysis;
try {
  analysis = JSON.parse(response.content[0].text);
} catch (e) {
  console.log('AI returned non-JSON, using defaults');
  analysis = { type: 'other', severity: 'low' };
}
```

### Xử lý rate limits

```javascript
// Retry với exponential backoff
async function callWithRetry(fn, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (e) {
      if (e.status === 429 && i < retries - 1) {
        await new Promise(r => setTimeout(r, 2 ** i * 1000));
      } else throw e;
    }
  }
}
```

---

## 💰 Cost Optimization

### Estimate chi phí

```
claude-haiku:  ~$0.00025 / 1K input tokens
claude-sonnet: ~$0.003 / 1K input tokens
claude-opus:   ~$0.015 / 1K input tokens

Một PR diff ~2000 tokens → sonnet = ~$0.006/review
```

### Tắt AI review cho draft PR

```yaml
on:
  pull_request:
    types: [opened, synchronize, ready_for_review]

jobs:
  review:
    if: github.event.pull_request.draft == false
```

---

## 📋 Checklist Setup mới

```markdown
- [ ] Thêm ANTHROPIC_API_KEY vào GitHub Secrets
- [ ] Copy workflows cần thiết vào .github/workflows/
- [ ] Tạo labels: bug, enhancement, question, priority: high
- [ ] Set minimal permissions trong từng workflow
- [ ] Test thử với một PR nhỏ
- [ ] Monitor chi phí API trong Anthropic Console
```

---

## Resources

- [Anthropic API Docs](https://docs.anthropic.com)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GitHub REST API](https://docs.github.com/en/rest)
- [Claude Models](https://docs.anthropic.com/en/docs/about-claude/models)
