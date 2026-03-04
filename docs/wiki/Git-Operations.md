# 🔀 Git Operations — Rebase, Cherry-pick & Cleanup

## Auto Rebase qua Comment

Gõ `/rebase` trong comment của PR để tự động rebase lên `main`.

```
/github-workflow-automation

"Tạo workflow cho phép rebase PR bằng cách comment /rebase"
```

### Cách dùng

1. Mở Pull Request
2. Comment `/rebase` vào PR
3. Bot tự động rebase và push

```
# Comment trong PR:
/rebase
```

### Workflow

```yaml
# .github/workflows/auto-rebase.yml
on:
  issue_comment:
    types: [created]

jobs:
  rebase:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/rebase')
    steps:
      - name: Rebase PR
        run: |
          gh pr checkout ${{ github.event.issue.number }}
          git fetch origin main
          if git rebase origin/main; then
            git push --force-with-lease
            # ✅ Comment thành công
          else
            git rebase --abort
            # ❌ Comment thất bại
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Branch Cleanup tự động

Xóa các branches không hoạt động hơn 30 ngày.

```
/github-workflow-automation

"Tạo workflow hàng tuần tìm và dọn dẹp stale branches"
```

```yaml
# .github/workflows/branch-cleanup.yml
on:
  schedule:
    - cron: '0 0 * * 0'  # Mỗi Chủ nhật

jobs:
  cleanup:
    steps:
      - name: Find stale branches
        run: |
          stale=$(git for-each-ref --sort=-committerdate refs/remotes/origin \
            --format='%(refname:short) %(committerdate:relative)' | \
            grep -E '[3-9][0-9]+ days|[0-9]+ months' | \
            grep -v 'origin/main\|origin/develop' | \
            cut -d' ' -f1 | sed 's|origin/||')

          echo "Stale branches: $stale"

      - name: Create cleanup issue
        # Tạo issue liệt kê các branches cần xóa
```

## Các Commands có sẵn

| Command | Mô tả |
|---------|-------|
| `/rebase` | Rebase PR lên `main` |
| `/update` | Merge `main` vào branch (thay vì rebase) |

## Smart Cherry-Pick (TypeScript)

Dùng AI để phân tích conflict trước khi cherry-pick:

```typescript
async function smartCherryPick(commitHash: string, targetBranch: string) {
  const commitInfo = await exec(`git show ${commitHash} --stat`);
  const targetDiff = await exec(`git diff ${targetBranch}...HEAD`);

  const analysis = await ai.analyze(`
    Cherry-pick ${commitHash} vào ${targetBranch}.
    Commit info: ${commitInfo}
    Current diff: ${targetDiff}

    Có conflict không? Suggest resolution strategy.
  `);

  if (analysis.willConflict) {
    console.log('⚠️ Conflict predicted. Creating branch for manual resolution...');
    await exec(`git checkout -b cherry-pick-${commitHash.slice(0,7)} ${targetBranch}`);
  }

  await exec(`git cherry-pick ${commitHash}`);
}
```

## Tips

- `force-with-lease` an toàn hơn `--force` khi push
- Luôn fetch trước khi rebase để có latest main
- Branch cleanup nên tạo issue để review, không xóa trực tiếp
