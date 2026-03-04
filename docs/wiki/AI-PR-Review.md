# 🤖 AI PR Review — Tự động review Pull Request

## Mô tả

Workflow `ai-review.yml` tự động trigger khi có Pull Request được mở hoặc update. Claude sẽ phân tích diff và để lại comment review chi tiết.

## Cách dùng

### Kích hoạt bằng `/github-workflow-automation`

```
/github-workflow-automation

"Tạo workflow tự động review PR, phát hiện bugs và security issues"
```

### Hoặc copy file có sẵn

```bash
cp .github/workflows/ai-review.yml /your-project/.github/workflows/
```

## File YAML

```yaml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: AI Review via Claude
        uses: actions/github-script@v7
        with:
          script: |
            const { Anthropic } = require('@anthropic-ai/sdk');
            const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

            const diff = await exec('git diff origin/${{ github.base_ref }}...HEAD');

            const response = await client.messages.create({
              model: "claude-sonnet-4-6",
              max_tokens: 4096,
              messages: [{
                role: "user",
                content: `Review PR:\n${diff}\n\nProvide: Summary, Issues, Suggestions, Security`
              }]
            });

            await github.rest.pulls.createReview({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
              body: response.content[0].text,
              event: 'COMMENT'
            });
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Output mẫu

Claude sẽ comment vào PR với format:

```markdown
## 📋 Summary
Thêm endpoint API mới cho user authentication...

## ✅ What looks good
- Clean code structure
- Proper error handling
- Good test coverage

## ⚠️ Potential Issues
1. **Line 42**: Possible null pointer — dùng optional chaining
2. **Line 78**: Missing input validation

## 🔒 Security Notes
- Không để lộ sensitive data trong logs
- Cân nhắc rate limiting cho auth endpoint
```

## Tùy chỉnh

### Chỉ review file code (bỏ qua docs/config)

```yaml
- name: Filter code files
  run: |
    files=$(git diff --name-only origin/${{ github.base_ref }}...HEAD | \
            grep -E '\.(ts|tsx|js|jsx|py|go|java)$' || true)
    echo "code_files=$files" >> $GITHUB_OUTPUT
```

### Thay đổi model

```yaml
model: "claude-opus-4-6"   # Mạnh nhất, chậm hơn
model: "claude-sonnet-4-6" # Cân bằng (recommended)
model: "claude-haiku-4-5-20251001"  # Nhanh, rẻ hơn
```

### Thêm ngôn ngữ review

```
content: "Review PR bằng tiếng Việt. Tập trung vào bugs và performance."
```

## Tips

- Giới hạn diff size để tránh token limit: `head -c 50000`
- Dùng `claude-haiku` cho review nhanh, `claude-sonnet` cho review sâu
- Thêm path filters để chỉ trigger khi code thực sự thay đổi
