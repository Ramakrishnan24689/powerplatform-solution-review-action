# Power Platform Review Action

A GitHub Action that automatically reviews Power Platform solutions for best practices, security, and quality standards using the `powerplatform-review-tool` NPM package.

## 🚀 Features

- **Comprehensive Analysis**: Reviews Canvas Apps, Power Automate flows, Dataverse customizations, and PCF controls
- **Security Scanning**: Identifies potential security vulnerabilities and compliance issues
- **Best Practices**: Validates against Microsoft Power Platform best practices
- **SARIF Integration**: Results appear in GitHub Security tab for easy tracking
- **PR Comments**: Automatic feedback on pull requests with detailed findings
- **Quality Gates**: Configurable thresholds to pass/fail builds based on review scores
- **Artifact Storage**: Detailed review reports stored as downloadable artifacts

## 📋 Quick Start

### 1. Add Solution File
Place your Power Platform solution (.zip file) in your repository:
```
your-repo/
├── solutions/
│   └── YourSolution.zip
└── .github/
    └── workflows/
        └── solution-review.yml
```

### 2. Create Workflow
Copy `.github/workflows/sample-solution-review.yml` to your repository and customize:

```yaml
name: Power Platform Solution Review

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          
      - name: Install Power Platform Review Tool
        run: npm install -g powerplatform-review-tool
        
      - name: Review Solution
        run: |
          mkdir -p review-output
          npx powerplatform-review-tool review solutions/YourSolution.zip --output review-output/
          
      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: review-output/results.sarif
```

### 3. Run and Review
- Push code or create PR
- Check **Security** tab for findings
- Download detailed reports from **Actions** artifacts
- Review PR comments for specific feedback

## 📁 Repository Structure

```
.github/
├── workflows/
│   ├── sample-solution-review.yml    # Main workflow
│   └── README.md                     # Workflow documentation
├── TESTING_LOCALLY.md               # Local testing guide
├── QUICK_SETUP.md                   # Quick reference
└── REPOSITORY_STRUCTURE.md          # This structure guide
test-workflow.ps1                    # PowerShell test script
test-workflow.sh                     # Bash test script
README.md                           # This file
```

## 🔍 What Gets Reviewed

### Canvas Apps
- Performance optimizations
- Accessibility compliance
- Formula best practices
- Control usage patterns
- Data source efficiency

### Power Automate
- Flow structure and complexity
- Connector usage and security
- Error handling patterns
- Performance considerations
- Trigger and action optimization

### Dataverse
- Entity relationship design
- Security role configurations
- Plugin and workflow patterns
- Data model best practices

### PCF Controls
- Code quality and structure
- TypeScript best practices
- Manifest configuration
- Performance patterns

## 📊 Viewing Results

### 1. GitHub Security Tab
- Navigate to **Security** > **Code scanning alerts**
- Filter by tool: "powerplatform-review-tool"
- View categorized findings with severity levels

### 2. Action Artifacts
- Go to **Actions** > Select your workflow run
- Download "review-reports" artifact
- Contains detailed JSON and HTML reports

### 3. Pull Request Comments
- Automatic comments on PRs with:
  - Overall solution score
  - Critical findings summary
  - Links to detailed reports

## ⚙️ Configuration Options

### Quality Gates
Set minimum score thresholds:

```yaml
- name: Check Quality Gate
  run: |
    SCORE=$(jq -r '.overallScore' review-output/results.json)
    if (( $(echo "$SCORE < 70" | bc -l) )); then
      echo "Quality gate failed: Score $SCORE below threshold 70"
      exit 1
    fi
```

### Custom Output Formats
```yaml
- name: Generate Custom Reports
  run: |
    npx powerplatform-review-tool review solution.zip \
      --output ./reports/ \
      --format json,sarif,html \
      --verbose
```

### Notification Integration
```yaml
- name: Notify Slack
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    text: "Power Platform review failed with critical issues"
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

## 🧪 Local Testing

### Option 1: Using act (GitHub Actions locally)
```bash
# Install act
# Windows (using chocolatey)
choco install act-cli

# macOS
brew install act

# Run workflow locally
act -W .github/workflows/sample-solution-review.yml
```

### Option 2: Direct CLI Testing
```bash
# Install globally
npm install -g powerplatform-review-tool

# Test with your solution
npx powerplatform-review-tool review YourSolution.zip --output ./test-results/
```

### Option 3: Use Test Scripts
```bash
# Windows PowerShell
.\test-workflow.ps1

# Linux/macOS
chmod +x test-workflow.sh
./test-workflow.sh
```

See `.github/TESTING_LOCALLY.md` for comprehensive testing guide.

## 🔧 Advanced Examples

### Scheduled Reviews
```yaml
on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM
  workflow_dispatch:     # Manual trigger
```

### Matrix Strategy (Multiple Solutions)
```yaml
strategy:
  matrix:
    solution: 
      - CustomerPortal.zip
      - EmployeeApp.zip
      - DataIntegration.zip
```

### Environment-Specific Reviews
```yaml
- name: Review for Production
  if: github.ref == 'refs/heads/main'
  run: |
    npx powerplatform-review-tool review solution.zip \
      --config production-rules.json \
      --strict
```

## 🐛 Troubleshooting

### Common Issues

**Issue**: "Module not found" error
```bash
# Solution: Clear npm cache and reinstall
npm cache clean --force
npm install -g powerplatform-review-tool
```

**Issue**: SARIF upload fails
```yaml
# Solution: Always use upload-sarif action
- name: Upload SARIF results
  uses: github/codeql-action/upload-sarif@v3
  if: always()  # Upload even if review fails
```

**Issue**: Large solution timeouts
```yaml
# Solution: Increase timeout
- name: Review Large Solution
  timeout-minutes: 30
  run: npx powerplatform-review-tool review LargeSolution.zip
```

### Debug Mode
Enable verbose logging:
```yaml
- name: Debug Review
  run: |
    npx powerplatform-review-tool review solution.zip \
      --verbose \
      --debug
```

## 📚 Documentation Links

- [Workflow Documentation](.github/workflows/README.md)
- [Local Testing Guide](.github/TESTING_LOCALLY.md)
- [Quick Setup Reference](.github/QUICK_SETUP.md)
- [NPM Package](https://www.npmjs.com/package/powerplatform-review-tool)

## 🆘 Support

For issues and questions:
1. Check the troubleshooting section above
2. Review the detailed documentation files
3. Test locally using the provided scripts
4. Check GitHub Actions logs for specific error details

## 📜 Quick Reference

```bash
# Install tool
npm install -g powerplatform-review-tool

# Basic review
npx powerplatform-review-tool review solution.zip

# With custom output
npx powerplatform-review-tool review solution.zip --output ./reports/

# Extract solution contents
npx powerplatform-review-tool extract solution.zip --output ./extracted/

# Test locally with act
act -W .github/workflows/sample-solution-review.yml

# Run test scripts
.\test-workflow.ps1  # Windows
./test-workflow.sh   # Linux/macOS
```

---

**Ready to improve your Power Platform solutions? Add this action to your repository and start getting automated reviews on every commit!** 🚀