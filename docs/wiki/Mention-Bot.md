# 🤖 Mention Bot — @ai-helper Commands

## Mô tả

Workflow `mention-bot.yml` lắng nghe comments trong Issues và PRs. Khi ai đó tag `@ai-helper`, Claude sẽ trả lời ngay trong comment.

## Cách dùng

Trong bất kỳ Issue hoặc PR nào, comment:

```
@ai-helper explain this code
@ai-helper tại sao đoạn code này bị lỗi?
@ai-helper viết test cho function này
@ai-helper có security issue không?
```

## Commands phổ biến

| Command | Mô tả |
|---------|-------|
| `@ai-helper explain` | Giải thích code trong PR/issue |
| `@ai-helper review` | Yêu cầu AI review thêm |
| `@ai-helper fix` | Gợi ý cách fix bug |
| `@ai-helper test` | Sinh test cases |
| `@ai-helper docs` | Tạo documentation |
| `@ai-helper translate` | Dịch comment/docs |

## Setup

```bash
# Copy workflow vào repo của bạn
cp .github/workflows/mention-bot.yml /your-project/.github/workflows/

# Thêm ANTHROPIC_API_KEY vào GitHub Secrets
```

## Workflow

```yaml
# .github/workflows/mention-bot.yml
name: AI Mention Bot

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  respond:
    if: contains(github.event.comment.body, '@ai-helper')
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write

    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const { Anthropic } = require('@anthropic-ai/sdk');
            const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

            // Extract question after @ai-helper
            const question = context.payload.comment.body
              .replace(/@ai-helper\s*/gi, '').trim();

            const response = await client.messages.create({
              model: "claude-sonnet-4-6",
              max_tokens: 2048,
              messages: [{
                role: "user",
                content: question
              }]
            });

            // Reply in the same thread
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `> 🤖 **@ai-helper** responding to @${context.payload.comment.user.login}\n\n${response.content[0].text}`
            });
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Thêm context cho bot

Để bot có context tốt hơn, tự động đưa vào diff của PR:

```javascript
// Nếu là PR comment, lấy diff
let context_info = '';
if (context.payload.issue?.pull_request) {
  const { data: pr } = await github.rest.pulls.get({
    owner: context.repo.owner,
    repo: context.repo.repo,
    pull_number: context.issue.number
  });
  context_info = `PR: ${pr.title}\nDescription: ${pr.body}`;
}

const response = await client.messages.create({
  messages: [{
    role: "user",
    content: `Context:\n${context_info}\n\nQuestion: ${question}`
  }]
});
```

## Tips

- Dùng `claude-sonnet-4-6` cho câu hỏi phức tạp
- Dùng `claude-haiku-4-5-20251001` cho câu hỏi đơn giản (rẻ hơn ~10x)
- Giới hạn `max_tokens: 1024` để tránh response quá dài
- Thêm system prompt để định hình phong cách trả lời
