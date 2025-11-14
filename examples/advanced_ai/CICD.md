# CI/CD Automation Documentation

## Overview

This document describes the comprehensive CI/CD automation implemented for the Advanced AI Architecture example. The automation suite includes testing, security scanning, deployment, and dependency management workflows.

## Workflows

### 1. Testing Workflow (`advanced-ai-test.yaml`)

**Triggers:**
- Push to `examples/advanced_ai/**`
- Pull requests affecting `examples/advanced_ai/**`
- Daily at 2 AM UTC (scheduled)
- Manual dispatch

**Jobs:**

#### test-notebook
- Validates notebook format using `nbformat`
- Checks Python syntax by converting notebook to script
- Runs linting with `flake8` (flexible rules for notebooks)
- Validates README structure and content
- Verifies `requirements.txt` integrity

#### test-integration
- Runs integration tests with OpenAI API (requires `OPENAI_API_KEY`)
- Only executes on manual trigger or schedule (to manage costs)
- Validates actual API functionality

#### code-quality
- Runs Black formatter checks
- Verifies import sorting with isort
- Generates code quality reports

**Output:**
- Test summary in GitHub Actions summary
- Pass/fail status for all checks

---

### 2. Security Workflow (`advanced-ai-security.yaml`)

**Triggers:**
- Push to `examples/advanced_ai/**`
- Pull requests affecting `examples/advanced_ai/**`
- Weekly on Mondays at 3 AM UTC
- Manual dispatch

**Jobs:**

#### dependency-security
- Scans dependencies with Safety
- Runs pip-audit for vulnerability detection
- Generates security reports

#### secret-scanning
- Uses TruffleHog for filesystem secret scanning
- Checks for hardcoded API keys
- Detects exposed environment variables
- Ensures secrets use proper environment variable patterns

#### code-security
- Runs Bandit for Python security analysis
- Executes Semgrep security rules
- Checks for common security issues:
  - `eval()` usage
  - `exec()` usage
  - `shell=True` in subprocess calls
- Uploads Bandit reports as artifacts (30-day retention)

#### license-compliance
- Checks dependency licenses with pip-licenses
- Verifies license compatibility
- Uploads license reports (90-day retention)

**Output:**
- Security summary with findings
- Bandit JSON report artifact
- License compliance report

---

### 3. Deployment Workflow (`advanced-ai-deploy.yaml`)

**Triggers:**
- Push to main branch (affects `examples/advanced_ai/**`)
- Release published
- Manual dispatch with environment selection (staging/production)

**Jobs:**

#### validate-before-deploy
- Checks all required files exist
- Validates notebook integrity
- Verifies documentation quality
- Ensures README has sufficient content

#### generate-documentation
- Converts notebook to HTML
- Generates Markdown documentation
- Optionally creates PDF (if dependencies available)
- Creates documentation package
- Uploads artifacts (90-day retention)

#### version-tagging
- Creates version tags with date stamps
- Generates version metadata JSON
- Uploads version metadata (365-day retention)

#### deploy-staging
- Deploys to staging environment (manual trigger)
- Simulates staging deployment process
- Generates deployment report

#### deploy-production
- Deploys to production (release or manual trigger)
- Simulates production deployment
- Updates changelog and notifies stakeholders

#### automated-updates
- Checks for outdated dependencies
- Generates update recommendations
- Runs on manual dispatch

#### notification
- Sends deployment notifications
- Always runs after deployment jobs (even on failure)

**Output:**
- Documentation in HTML, Markdown, and PDF formats
- Version metadata JSON
- Deployment status reports

---

### 4. Dependency Management Workflow (`advanced-ai-dependencies.yaml`)

**Triggers:**
- Weekly on Mondays at 9 AM UTC
- Manual dispatch

**Jobs:**

#### check-updates
- Checks for outdated packages
- Generates detailed update report with tabulated data
- Runs security advisories check with pip-audit
- Creates update report artifacts (30-day retention)

#### compatibility-check
- Tests compatibility with Python 3.9, 3.10, 3.11, and 3.12
- Verifies dependency installation across versions
- Tests critical imports
- Matrix strategy for parallel testing

#### performance-check
- Measures dependency installation time
- Analyzes package sizes
- Generates performance recommendations
- Suggests optimization strategies

**Output:**
- Dependency update report (Markdown)
- Compatibility matrix results
- Performance metrics

---

## Security Features

### Permissions
All workflows use minimal permissions following the principle of least privilege:
- Most jobs: `contents: read` (read-only access)
- Notification jobs: No permissions (`{}`)

### Secret Management
- API keys stored in GitHub Secrets
- No hardcoded credentials in code
- Environment variable validation
- TruffleHog scanning for exposed secrets

### Vulnerability Scanning
- Safety - Known vulnerability database
- pip-audit - PyPA's official security scanner
- Bandit - Python code security linter
- Semgrep - Pattern-based security analysis

---

## Artifacts

### Test Reports
- **Retention**: 30 days
- **Contents**: Test results, lint reports, coverage data

### Security Reports
- **Retention**: 30 days
- **Contents**: Bandit JSON reports, vulnerability scans

### Documentation
- **Retention**: 90 days
- **Contents**: HTML, Markdown, PDF versions of notebook

### License Reports
- **Retention**: 90 days
- **Contents**: Dependency license information

### Version Metadata
- **Retention**: 365 days
- **Contents**: Version tags, release information, commit SHAs

---

## Usage

### Running Workflows Manually

#### Test Workflow
```bash
# Via GitHub CLI
gh workflow run advanced-ai-test.yaml

# Via GitHub UI
Actions → Advanced AI Architecture - Testing → Run workflow
```

#### Security Scan
```bash
gh workflow run advanced-ai-security.yaml
```

#### Deploy
```bash
# Deploy to staging
gh workflow run advanced-ai-deploy.yaml -f deploy_environment=staging

# Deploy to production
gh workflow run advanced-ai-deploy.yaml -f deploy_environment=production
```

#### Check Dependencies
```bash
gh workflow run advanced-ai-dependencies.yaml
```

---

## Environment Setup

### Required Secrets
- `OPENAI_API_KEY` - For integration tests (optional but recommended)

### Optional Secrets
- Deployment secrets (if implementing actual deployments)
- Notification webhooks (Slack, Teams, etc.)

---

## Customization

### Modifying Schedules
Edit the `cron` expressions in workflow files:
```yaml
schedule:
  - cron: '0 2 * * *'  # Daily at 2 AM UTC
```

### Adding New Checks
Add new jobs to workflows:
```yaml
  custom-check:
    name: Custom Check
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Custom step
        run: echo "Custom check"
```

### Changing Python Versions
Modify the matrix in `advanced-ai-dependencies.yaml`:
```yaml
strategy:
  matrix:
    python-version: ['3.9', '3.10', '3.11', '3.12', '3.13']
```

---

## Troubleshooting

### Workflow Fails with "No OPENAI_API_KEY"
- Integration tests require the API key
- Either:
  1. Add the secret to GitHub Secrets
  2. Skip integration tests (they only run on schedule/manual trigger)

### Security Scan False Positives
- Review Bandit reports in artifacts
- Add `# nosec` comments to suppress false positives
- Update `.banditrc` if needed

### Documentation Generation Fails
- PDF generation requires additional LaTeX dependencies
- HTML and Markdown generation should always work
- Check job logs for specific errors

### Dependency Check Reports Errors
- May indicate actual outdated packages
- Review the update report artifact
- Test updates in development before applying

---

## Best Practices

1. **Regular Reviews**: Check security and dependency reports weekly
2. **Timely Updates**: Apply security patches promptly
3. **Test Before Deploy**: Always test in staging before production
4. **Monitor Costs**: Integration tests use API credits
5. **Keep Workflows Updated**: Update actions to latest versions
6. **Document Changes**: Update this file when modifying workflows

---

## Monitoring

### GitHub Actions UI
- View workflow runs: Actions tab → Select workflow
- Download artifacts: Workflow run → Artifacts section
- View logs: Workflow run → Job name → Step details

### Notifications
Configure notifications in `.github/workflows/`:
- Slack webhooks
- Email notifications
- Custom integrations

---

## Compliance

### Security Compliance
- ✓ Secrets scanning
- ✓ Dependency vulnerability scanning
- ✓ Code security analysis
- ✓ License compliance checking

### Best Practices
- ✓ Minimal permissions
- ✓ Regular security scans
- ✓ Automated dependency updates
- ✓ Comprehensive testing
- ✓ Documentation generation

---

## Support

For issues or questions:
1. Check workflow logs in GitHub Actions
2. Review this documentation
3. Check artifact reports for details
4. Open an issue in the repository

---

## Version History

### v1.0.0 (2025-11-14)
- Initial CI/CD automation implementation
- Four comprehensive workflows
- Security scanning suite
- Documentation generation
- Dependency management
