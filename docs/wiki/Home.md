# 🔧 GitHub Workflow Automation — Hướng dẫn sử dụng

Chào mừng đến với wiki hướng dẫn sử dụng skill **`github-workflow-automation`** trong Claude Code!

## Skill này làm gì?

Skill `github-workflow-automation` giúp bạn **tự động hóa GitHub workflows bằng AI** thông qua GitHub Actions. Chỉ cần gọi `/github-workflow-automation` trong Claude Code và mô tả nhu cầu — Claude sẽ sinh ra YAML config sẵn sàng dùng.

## 📑 Mục lục

| Trang | Mô tả |
|-------|-------|
| [Setup](Setup) | Cài đặt và cấu hình ban đầu |
| [AI PR Review](AI-PR-Review) | Tự động review Pull Request bằng Claude |
| [Issue Triage](Issue-Triage) | Phân loại và gán nhãn issue tự động |
| [CI/CD Integration](CICD-Integration) | Smart testing & AI-assisted deploy |
| [Git Operations](Git-Operations) | Auto rebase, cherry-pick, cleanup |
| [Mention Bot](Mention-Bot) | @ai-helper chatbot trong issues/PRs |
| [Best Practices](Best-Practices) | Security, tips & patterns |

## 🚀 Bắt đầu nhanh (3 bước)

```bash
# Bước 1: Trong Claude Code terminal, gọi skill
/github-workflow-automation

# Bước 2: Mô tả workflow bạn muốn
"Tôi muốn tự động review PR khi có người mở Pull Request mới"

# Bước 3: Claude sinh ra YAML — copy vào .github/workflows/
cp generated-workflow.yml .github/workflows/ai-review.yml
git add . && git commit -m "feat: add AI PR review" && git push
```

## ⚙️ Yêu cầu

- GitHub repository với **Actions** enabled
- `ANTHROPIC_API_KEY` trong **GitHub Secrets** (Settings → Secrets)
- Claude Code với skill `github-workflow-automation` đã cài

## 📁 Repo structure

```
.github/
  workflows/
    ai-review.yml        # AI review PR tự động
    issue-triage.yml     # Triage issue + auto-label
    mention-bot.yml      # @ai-helper bot
    auto-rebase.yml      # /rebase command
docs/wiki/               # Tài liệu này
```
