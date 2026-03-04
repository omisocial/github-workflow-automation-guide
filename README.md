# 🔧 GitHub Workflow Automation Guide

> Hướng dẫn sử dụng skill **`github-workflow-automation`** trong Claude Code để tự động hóa GitHub workflows với AI.

## 📖 Giới thiệu

Skill `github-workflow-automation` cung cấp các patterns và template YAML/TypeScript để:

- **AI PR Review** — Tự động review Pull Request bằng Claude/AI
- **Issue Triage** — Phân loại và gán nhãn issue tự động
- **Smart CI/CD** — Chạy test thông minh, deploy có AI validation
- **Git Operations** — Auto rebase, cherry-pick, branch cleanup
- **On-demand Bot** — `@mention` bot trả lời câu hỏi về code

## 🗂 Cấu trúc Repository

```
.github/
  workflows/
    ai-review.yml           # AI code review tự động
    issue-triage.yml        # Triage issue tự động
    smart-tests.yml         # Chọn test suite thông minh
    deploy.yml              # Deploy với AI risk assessment
    rollback.yml            # Rollback tự động
    auto-rebase.yml         # Auto rebase qua comment
    branch-cleanup.yml      # Dọn dẹp stale branches
    mention-bot.yml         # @ai-helper bot
  CODEOWNERS               # Cấu hình code ownership
docs/
  wiki/                    # Tài liệu chi tiết (xem GitHub Wiki)
```

## 🚀 Bắt đầu nhanh

### 1. Thêm ANTHROPIC_API_KEY vào GitHub Secrets

```
Settings → Secrets and variables → Actions → New repository secret
Name: ANTHROPIC_API_KEY
Value: sk-ant-...
```

### 2. Copy workflow bạn cần

```bash
# Copy AI PR Review workflow
cp .github/workflows/ai-review.yml /your-project/.github/workflows/
```

### 3. Commit và push

```bash
git add .github/
git commit -m "feat: add AI workflow automation"
git push
```

## 📚 Tài liệu đầy đủ

Xem **[GitHub Wiki](../../wiki)** để có hướng dẫn chi tiết từng workflow.

| Wiki Page | Nội dung |
|-----------|----------|
| [Home](../../wiki) | Tổng quan và điều hướng |
| [Cài đặt](../../wiki/Setup) | Cấu hình ban đầu |
| [AI PR Review](../../wiki/AI-PR-Review) | Tự động review PR |
| [Issue Triage](../../wiki/Issue-Triage) | Phân loại issue |
| [CI/CD Integration](../../wiki/CICD-Integration) | Smart testing & deploy |
| [Git Operations](../../wiki/Git-Operations) | Rebase, cherry-pick |
| [Mention Bot](../../wiki/Mention-Bot) | @ai-helper commands |
| [Best Practices](../../wiki/Best-Practices) | Security & tips |

## 🛠 Skill trong Claude Code

Skill này được kích hoạt trong Claude Code bằng lệnh:

```
/github-workflow-automation
```

Sau đó mô tả workflow bạn muốn tự động hóa và Claude sẽ sinh ra YAML config phù hợp.

## 📄 License

MIT — Tự do sử dụng và chỉnh sửa cho dự án của bạn.
