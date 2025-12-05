# PR Description Manager Action

A lightweight GitHub Action that manages PR descriptions with unique section identifiers. Built using `actions/github-script` for reliability and maintainability.

## Features

- ✅ Add or update specific sections in PR descriptions independently
- 🔖 Each section has a unique ID for targeted updates
- 🔄 Automatically detects existing sections and updates them
- 📝 Adds a header marker to identify auto-generated content
- 🎯 Works with any PR in your repository
- 🪶 Lightweight - uses GitHub's official github-script action

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token for API authentication | Yes | - |
| `section-id` | Unique identifier for the section | Yes | - |
| `content` | Content to insert (markdown supported) | Yes | - |
| `pr-number` | PR number (auto-detected if not provided) | No | Current PR |

## Outputs

| Output | Description |
|--------|-------------|
| `updated` | `true` if section was updated, `false` if newly created |

## Usage

### Basic Example

```yaml
name: Update PR Description

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  pull-requests: write

jobs:
  update-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Add deployment info
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'deployment-preview'
          content: |
            ## 🚀 Deployment Preview
            
            Your changes are deployed at:
            https://pr-${{ github.event.pull_request.number }}.preview.example.com
```

### Multiple Independent Sections

```yaml
jobs:
  update-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Add test results
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'test-results'
          content: |
            ## 🧪 Test Results
            ✅ All tests passed
            📊 Coverage: 85%
      
      - name: Add build info
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'build-info'
          content: |
            ## 🏗️ Build Information
            ⏱️ Duration: 2m 15s
            📦 Size: 3.2MB
      
      - name: Add security scan
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'security-scan'
          content: |
            ## 🔒 Security Scan
            ✅ No vulnerabilities detected
```

### Dynamic Content with Job Outputs

```yaml
jobs:
  test-and-update:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        id: tests
        run: |
          # Your test commands here
          echo "status=passed" >> $GITHUB_OUTPUT
          echo "coverage=87.5" >> $GITHUB_OUTPUT
          echo "duration=3m 45s" >> $GITHUB_OUTPUT

      - name: Update PR with test results
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'test-results'
          content: |
            ## 🧪 Test Results
            
            | Metric | Value |
            |--------|-------|
            | Status | ${{ steps.tests.outputs.status }} |
            | Coverage | ${{ steps.tests.outputs.coverage }}% |
            | Duration | ${{ steps.tests.outputs.duration }} |
            
            _Last run: ${{ github.event.head_commit.timestamp }}_
```

### Conditional Updates

```yaml
jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy preview
        id: deploy
        run: |
          # Deployment logic
          echo "url=https://preview.example.com" >> $GITHUB_OUTPUT
          echo "success=true" >> $GITHUB_OUTPUT

      - name: Update PR with deployment info
        if: steps.deploy.outputs.success == 'true'
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'deployment'
          content: |
            ## ✅ Deployment Successful
            
            🔗 Preview: ${{ steps.deploy.outputs.url }}
      
      - name: Update PR with failure notice
        if: steps.deploy.outputs.success != 'true'
        uses: ./.github/actions/update-pr-description
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          section-id: 'deployment'
          content: |
            ## ❌ Deployment Failed
            
            Please check the logs for details.
```

### Update Specific PR

```yaml
- name: Update different PR
  uses: ./.github/actions/update-pr-description
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    section-id: 'release-notes'
    pr-number: 123
    content: |
      ## 📋 Release Notes
      
      This PR is included in the next release.
```

## How It Works

### First Run
When the action runs for the first time on a PR:

1. Adds the header marker (if not present):
   ```markdown
   <!-- This is an auto-generated comment: created by zouwu.fr -->
   ```

2. Appends your section with markers:
   ```markdown
   <!-- your-section-id -->
   Your content here
   <!-- end: your-section-id -->
   ```

### Subsequent Runs
When the action runs again with the same `section-id`:

1. Finds the existing section markers
2. Replaces **only** the content between those markers
3. Preserves all other content in the PR description

### Multiple Sections
Each `section-id` is managed independently:

```markdown
<!-- This is an auto-generated comment: created by zouwu.fr -->

<!-- test-results -->
## 🧪 Test Results
All tests passed
<!-- end: test-results -->

<!-- deployment -->
## 🚀 Deployment
Preview: https://preview.example.com
<!-- end: deployment -->

<!-- security-scan -->
## 🔒 Security
No issues found
<!-- end: security-scan -->
```

## Installation

### Option 1: Local Action
1. Create directory: `.github/actions/update-pr-description/`
2. Copy `action.yml` to that directory
3. Reference it in workflows: `uses: ./.github/actions/update-pr-description`

### Option 2: Reusable Repository
1. Create a repository for the action
2. Push the `action.yml` file
3. Reference it: `uses: your-org/update-pr-description@v1`

## Required Permissions

Your workflow needs these permissions:

```yaml
permissions:
  pull-requests: write
  contents: read  # for checkout if needed
```

## Advantages of This Implementation

- ✅ **Simple**: Uses GitHub's official `github-script` action
- ✅ **Reliable**: Direct API calls, no shell parsing
- ✅ **Maintainable**: JavaScript is easier to debug than complex bash
- ✅ **Type-safe**: Better error handling and validation
- ✅ **No dependencies**: Only requires `actions/github-script@v7`

## Tips & Best Practices

1. **Use descriptive section IDs**: `deployment-preview`, `test-results`, not `section1`
2. **Keep content focused**: Each section should have a single purpose
3. **Use markdown**: Format your content for better readability
4. **Add timestamps**: Include when the section was last updated
5. **Handle failures**: Use conditional steps to show both success and failure states

## Troubleshooting

### "Could not determine PR number"
Make sure the action runs in a `pull_request` context or provide `pr-number` explicitly:
```yaml
on:
  pull_request:  # Required if not providing pr-number
```

### Permission denied
Ensure your workflow has the required permissions:
```yaml
permissions:
  pull-requests: write
```

### Section not updating
Verify that your `section-id` matches exactly (case-sensitive):
```yaml
section-id: 'test-results'  # Must match in all uses
```

## Examples Repository

Check out the `.github/workflows/update-pr-example.yml` file for complete working examples.

## Contributing

Contributions welcome! Please test your changes thoroughly before submitting.

## License

MIT License

## Author

Created by [zouwu.fr](https://zouwu.fr)