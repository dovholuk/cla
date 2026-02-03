# CLA Setup Guide for Repository Maintainers

This guide explains how to enable CLA checking on repositories in the NetFoundry, OpenZiti, or OpenZiti-Test-Kitchen organizations.

## Prerequisites

1. **CLA_TOKEN secret** - A fine-grained Personal Access Token with:
   - Repository access: `netfoundry/cla` only
   - Permissions: Contents (read and write)

   This token should be added as an **organization secret** named `CLA_TOKEN` in each org, or as a repository secret for individual repos.

## Setup Steps

### Option 1: Add to a Single Repository

1. Copy `workflow-template/cla.yml` to your repo at `.github/workflows/cla.yml`
2. Ensure the `CLA_TOKEN` secret is available (org-level or repo-level)
3. Commit and push
4. Test by opening a PR from a non-member account

### Option 2: Add to All Repositories in an Organization

You can use GitHub's organization-level workflow templates or a script to add the workflow to multiple repos at once.

**Using gh CLI:**

```bash
# Clone a repo, add the workflow, push
for repo in repo1 repo2 repo3; do
  gh repo clone "openziti/$repo" "/tmp/$repo"
  mkdir -p "/tmp/$repo/.github/workflows"
  cp workflow-template/cla.yml "/tmp/$repo/.github/workflows/cla.yml"
  cd "/tmp/$repo"
  git add .github/workflows/cla.yml
  git commit -m "Add CLA check workflow"
  git push
  cd -
done
```

## Creating the CLA_TOKEN

1. Go to https://github.com/settings/tokens?type=beta (fine-grained tokens)
2. Click "Generate new token"
3. Name: `CLA Bot Token` (or similar)
4. Expiration: Set as needed (consider 1 year, with a reminder to rotate)
5. Resource owner: `netfoundry`
6. Repository access: "Only select repositories" → select `netfoundry/cla`
7. Permissions:
   - Contents: Read and write
8. Generate and copy the token

### Add as Organization Secret

For each organization (netfoundry, openziti, openziti-test-kitchen):

1. Go to Organization Settings → Secrets and variables → Actions
2. Click "New organization secret"
3. Name: `CLA_TOKEN`
4. Value: (paste the token)
5. Repository access: "All repositories" or select specific ones

## Testing

1. Add the workflow to a test repository
2. Create a test PR from an account that hasn't signed the CLA
3. Verify the bot comments asking for a signature
4. Sign by commenting: `I have read the CLA Document and I hereby sign the CLA`
5. Verify the signature appears in `signatures/cla.json` in this repo
6. Verify the PR status check passes

## Organization Members

The workflow automatically skips the CLA check for organization members and owners using GitHub's `author_association` field. Employees don't need to sign - their contributions are covered by their employment agreement.

This check happens at the job level:
```yaml
if: |
  github.event_name == 'issue_comment' ||
  (github.event_name == 'pull_request_target' &&
   github.event.pull_request.author_association != 'MEMBER' &&
   github.event.pull_request.author_association != 'OWNER')
```

## Allowlist

The workflow also includes an allowlist for bots:
```
allowlist: dependabot[bot],renovate[bot],github-actions[bot],bot*
```

To add additional users who should bypass the CLA (e.g., contractors not in the org):
- Add their GitHub usernames to the allowlist (comma-separated)
- Or add them directly to `signatures/cla.json` manually

## CLA Documents

The workflows link to the official NetFoundry CLA PDFs:
- **Individual CLA:** https://netfoundry.io/docs/assets/files/NetFoundry-ICLA-32974791ae564dd1878a7d2ab1ab8d5e.pdf
- **Corporate CLA:** https://netfoundry.io/docs/assets/files/NetFoundry-CCLA-a68e768031f697589e7d435f17e0cf31.pdf

If these URLs change, update the `path-to-document` value in the workflow files.

## Troubleshooting

**"Resource not accessible by integration"**
- The `CLA_TOKEN` secret is missing or doesn't have correct permissions
- Check that the token has Contents (read/write) access to `netfoundry/cla`

**Bot doesn't comment on PRs**
- Check that the workflow file is in `.github/workflows/cla.yml`
- Check the Actions tab for workflow run errors
- Ensure `pull_request_target` trigger is present (not just `pull_request`)

**Signatures not being recorded**
- The `PERSONAL_ACCESS_TOKEN` env var must be set to `${{ secrets.CLA_TOKEN }}`
- The token must have write access to this repo

