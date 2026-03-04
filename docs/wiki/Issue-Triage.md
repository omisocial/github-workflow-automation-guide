# 🏷️ Issue Triage — Phân loại issue tự động

## Mô tả

Workflow `issue-triage.yml` tự động phân tích issue mới, gán labels phù hợp, và hỏi thêm thông tin nếu cần.

## Cách dùng

```
/github-workflow-automation

"Tạo workflow tự động phân loại issues mới, gán bug/enhancement/question labels"
```

## Workflow hoạt động thế nào

```
Issue mới được tạo
        ↓
Claude phân tích title + body
        ↓
Phân loại: bug | feature | question | docs | other
        ↓
Gán labels tự động
        ↓
Nếu là bug mà thiếu repro steps → comment hỏi thêm
```

## Labels được gán tự động

| Loại issue | Labels |
|-----------|--------|
| Bug | `bug` |
| Feature request | `enhancement` |
| Question | `question` |
| Docs | `documentation` |
| Severity cao | `priority: high` |

> **Lưu ý:** Labels phải được tạo trước trong repo. Vào **Issues** → **Labels** để tạo.

## Tạo labels cần thiết

```bash
# Dùng gh CLI để tạo labels
gh label create "bug" --color "d73a4a" --repo omisocial/your-repo
gh label create "enhancement" --color "a2eeef" --repo omisocial/your-repo
gh label create "question" --color "d876e3" --repo omisocial/your-repo
gh label create "documentation" --color "0075ca" --repo omisocial/your-repo
gh label create "priority: high" --color "e4e669" --repo omisocial/your-repo
```

## Tùy chỉnh prompt phân loại

```javascript
const response = await client.messages.create({
  model: "claude-haiku-4-5-20251001",  // Dùng Haiku để tiết kiệm chi phí
  max_tokens: 512,
  messages: [{
    role: "user",
    content: `Phân tích issue GitHub này và trả về JSON:

Title: ${issue.title}
Body: ${issue.body}

JSON format:
{
  "type": "bug" | "feature" | "question" | "docs" | "other",
  "severity": "low" | "medium" | "high" | "critical",
  "hasReproSteps": true | false,
  "suggestedLabels": ["label1", "label2"]
}`
  }]
});
```

## Stale Issue Management

Thêm workflow dọn dẹp issues không hoạt động:

```yaml
# .github/workflows/stale.yml
- uses: actions/stale@v9
  with:
    days-before-stale: 60
    days-before-close: 14
    stale-issue-label: "stale"
    stale-issue-message: |
      Issue này chưa có hoạt động mới trong 60 ngày và sẽ được đóng sau 14 ngày.
      Comment để giữ issue này mở.
```

## Tips

- Dùng `claude-haiku` cho triage (nhanh + rẻ hơn)
- Thêm auto-assign dựa trên `area` trong JSON response
- Tạo template issue để AI có context tốt hơn
