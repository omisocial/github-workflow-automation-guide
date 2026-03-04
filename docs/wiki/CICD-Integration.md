# 🚀 CI/CD Integration — Smart Testing & AI Deploy

## Smart Test Selection

Thay vì chạy toàn bộ test suite mỗi lần, workflow phân tích file thay đổi và chỉ chạy test liên quan.

```
/github-workflow-automation

"Tạo workflow chỉ chạy test suite liên quan đến files thay đổi"
```

### Workflow

```yaml
# .github/workflows/smart-tests.yml
name: Smart Test Selection

on:
  pull_request:

jobs:
  analyze:
    runs-on: ubuntu-latest
    outputs:
      test_suites: ${{ steps.analyze.outputs.suites }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Analyze changed files
        id: analyze
        run: |
          changed=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)
          suites="[]"

          if echo "$changed" | grep -q "^src/api/"; then
            suites=$(echo $suites | jq '. + ["api"]')
          fi
          if echo "$changed" | grep -q "^src/frontend/"; then
            suites=$(echo $suites | jq '. + ["frontend"]')
          fi
          if [ "$suites" = "[]" ]; then
            suites='["all"]'
          fi

          echo "suites=$suites" >> $GITHUB_OUTPUT

  test:
    needs: analyze
    strategy:
      matrix:
        suite: ${{ fromJson(needs.analyze.outputs.test_suites) }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test -- --suite ${{ matrix.suite }}
```

## AI Deployment Risk Assessment

Trước khi deploy, Claude đánh giá rủi ro dựa trên commits gần đây.

```yaml
# .github/workflows/deploy.yml
- name: AI Risk Assessment
  uses: actions/github-script@v7
  with:
    script: |
      const { Anthropic } = require('@anthropic-ai/sdk');
      const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

      const changes = process.env.RECENT_COMMITS;

      const response = await client.messages.create({
        model: "claude-sonnet-4-6",
        max_tokens: 512,
        messages: [{
          role: "user",
          content: `Đánh giá rủi ro deploy dựa trên commits này. Trả về JSON:

${changes}

{"riskLevel":"low"|"medium"|"high","concerns":[],"requiresApproval":false}`
        }]
      });

      const analysis = JSON.parse(response.content[0].text);

      if (analysis.riskLevel === 'high') {
        core.setFailed('High-risk deployment! Manual review required.');
        core.summary.addRaw(analysis.concerns.join('\n'));
      }
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    RECENT_COMMITS: ${{ steps.commits.outputs.log }}
```

## Automated Rollback

```
/github-workflow-automation

"Tạo workflow rollback tự động về version stable khi deploy fail"
```

```yaml
# .github/workflows/rollback.yml
on:
  workflow_dispatch:
    inputs:
      reason:
        description: 'Lý do rollback'
        required: true

jobs:
  rollback:
    steps:
      - name: Find last stable tag
        run: |
          stable=$(git tag -l 'v*' --sort=-version:refname | head -1)
          echo "VERSION=$stable" >> $GITHUB_ENV

      - name: Deploy stable version
        run: |
          git checkout ${{ env.VERSION }}
          npm run deploy

      - name: Notify Slack
        # ... thông báo cho team
```

## Tips

- Dùng `jq` để parse JSON an toàn từ AI response
- Luôn có fallback nếu AI response không parse được
- Risk level `high` nên block deploy và require manual approval
