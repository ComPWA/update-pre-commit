# Auto-update pre-commit GitHub Action

> [!WARNING]
> This repository has been deprecated in favor of the [ComPWA/actions/.github/workflows/lock.yml@v4](https://github.com/ComPWA/actions/blob/v4/.github/workflows/lock.yml#L213-L236) workflow.

A small [custom, composite GitHub Action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) that is used by [ComPWA](https://github.com/ComPWA) repositories that pin constraints files (see [update-pip-constraints](https://github.com/ComPWA/update-pip-constraints)). The action is enforced in these repositories through [ComPWA/policy](https://github.com/ComPWA/policy).

## Example workflow

```yaml
jobs:
  pre-commit:
    name: pre-commit autoupdate
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - name: Check if there are changes to pre-commit config
        run: git diff origin/main --exit-code -- .pre-commit-config.yml
        continue-on-error: true
      - name: Update hooks in pre-commit config
        if: success()
        uses: ComPWA/update-pre-commit@v1

  push:
    name: Push changes
    runs-on: ubuntu-24.04
    needs:
      - pre-commit
    steps:
      - uses: actions/checkout@v6
        with:
          token: ${{ secrets.PAT }}
        # GITHUB_TOKEN will not rerun checks after pushing to a PR branch
      - uses: actions/download-artifact@v7
        with:
          path: pre-commit
      - run: |
          [[ -f .pre-commit-config.yaml ]] && mv -f .pre-commit-config.yaml ..
        working-directory: pre-commit
      - name: Commit and push changes
        run: |
          git remote set-url origin https://x-access-token:${{ secrets.PAT }}@github.com/${{ github.repository }}
          git config --global user.name "GitHub"
          git config --global user.email "noreply@github.com"
          git checkout -b ${GITHUB_HEAD_REF}
          if [[ $(git status -s) ]]; then
            git add -A
            git commit -m "ci: upgrade pinned requirements (automatic)"
            git config pull.rebase true
            git pull origin ${GITHUB_HEAD_REF}
            git push origin HEAD:${GITHUB_HEAD_REF}
          fi
```
