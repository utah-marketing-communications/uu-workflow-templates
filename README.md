# UU Workflow Templates

Reusable GitHub Actions workflows for deploying UU widgets to AWS and WP Engine environments.

## 📋 Overview

This repository contains reusable workflow templates that can be used across all UU widget repositories to standardize deployment processes.

## 🏗️ Repository Structure

```
.github/
└── workflows/
    ├── deploy-aws.yml              # Reusable AWS deployment workflow
    ├── deploy-wp-engine.yml        # Reusable WP Engine deployment workflow
    ├── deploy-staging.yml         # Generic staging deployment workflow
    └── deploy-production.yml      # Generic production deployment workflow

examples/
└── README.md                      # Usage examples and documentation
```

## 🚀 Quick Start

### 1. Create Deployment Workflows

Create deployment workflows in your widget repository that call the template workflows:

```yaml
# .github/workflows/deploy-staging.yml
name: STAGE Deploy My Widget to Staging

on:
  release:
    types: [published]

jobs:
  deploy:
    uses: utah-marketing-communications/uu-workflow-templates/.github/workflows/deploy-staging.yml@main
    with:
      widget_name: my-widget-name
      tag_name: ${{ github.event.release.tag_name }}
      deploy_aux: true
      deploy_wp_engine: true
    secrets: inherit
```

### 2. Configure Your Widget

Update the workflow with your widget-specific configuration:

- **Widget name** (replaces `my-widget-name`)
- **Environment selection** (which environments to deploy to)
- **Safety settings** (delete flags, cache clearing, etc.)

### 3. Add Required Secrets

Ensure your repository has the required secrets configured:

#### AWS Secrets (for each environment):
- `AUX_STG_SSH_PRIVATE_KEY`
- `AUX_STG_SSH_PORT`
- `AUX_STG_SSH_USER`
- `AUX_STG_SSH_HOST`
- `AUX_PROD_SSH_KEY`
- `AUX_PROD_SSH_PORT`
- `AUX_PROD_SSH_USER`
- `AUX_PROD_SSH_HOST`
- (Repeat for BRYCE, CAP_REEF, ZION)

#### WP Engine Secrets:
- `WPE_ORG_API_USER`
- `WPE_ORG_API_PASS`
- `WPE_ORG_SSHG_KEY_PRIVATE`

## 📖 Workflow Details

### deploy-aws.yml

Reusable workflow for deploying to AWS servers via SSH/rsync.

**Inputs:**
- `environment`: Environment name (e.g., aux-staging, bryce-production)
- `server_path`: Server path for the deployment
- `tag_name`: Tag name to deploy
- `write_version_file`: Write version.txt file after deployment (default: false)
- `delete`: Delete files on server not in local directory (default: false)

**Required Secrets:**
- `SSH_PRIVATE_KEY`: SSH private key for the server
- `SSH_PORT`: SSH port for the server
- `SSH_USER`: SSH user for the server
- `SSH_HOST`: SSH host for the server

### deploy-wp-engine.yml

Reusable workflow for deploying to WP Engine environments.

**Inputs:**
- `environment_type`: Environment type (staging or production)
- `tag_name`: Tag name to deploy (for production)
- `dry_run`: Run deployment in dry run mode (default: false)
- `delete`: Include --delete flag in deployment (default: false)
- `cache_clear`: Clear WP Engine cache after deploy (default: false)
- `max_parallel`: Maximum concurrent WP Engine site deployments (default: 5)
- `throttle_delay_seconds`: Delay before each deploy attempt (default: 1)
- `cleanup_after_deploy`: Run post-deploy cleanup commands (default: false)

**Required Secrets:**
- `WPE_ORG_API_USER`: WP Engine API username
- `WPE_ORG_API_PASS`: WP Engine API password
- `WPE_ORG_SSHG_KEY_PRIVATE`: WP Engine SSH private key

#### Current WP Engine Baseline

The current default template behavior is tuned for speed while preserving file safety:

- PHP lint runs once before deploy fanout (`lint-source` job)
- Per-site action lint is disabled (`PHP_LINT: false`)
- WP Engine deploy uses rsync chmod flags (`--chmod=D775,F664`)
- No post-deploy recursive `find/chmod` pass
- Matrix concurrency defaults to `max_parallel=5`
- Throttle delay defaults to `throttle_delay_seconds=1`
- Post-deploy cleanup remains optional and defaults to `cleanup_after_deploy=false`

## 🔧 Usage Examples

### Staging Deployment

```yaml
# In your widget repo's .github/workflows/deploy-staging.yml
name: STAGE Deploy widget-name to All Staging Environments

on:
  release:
    types: [published]

jobs:
  deploy-aux-staging:
    uses: utah-marketing-communications/uu-workflow-templates/.github/workflows/deploy-aws.yml@main
    with:
      environment: aux-staging
      server_path: /var/www/html/aux.umc.utah.edu/wp-content/plugins/uu-so-widgets/uu-so-widgets-bundle/your-widget-name
      tag_name: ${{ github.event.release.tag_name }}
      write_version_file: false
      delete: true
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.AUX_STG_SSH_PRIVATE_KEY }}
      SSH_PORT: ${{ secrets.AUX_STG_SSH_PORT }}
      SSH_USER: ${{ secrets.AUX_STG_SSH_USER }}
      SSH_HOST: ${{ secrets.AUX_STG_SSH_HOST }}
```

### Production Deployment

```yaml
# In your widget repo's .github/workflows/deploy-production.yml
name: PROD Deploy widget-name to Production Environments

on:
  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to deploy (e.g., v1.2.3)'
        required: true
      deploy_aux:
        description: 'Deploy to AUX production'
        type: boolean
        default: false
      aws_delete:
        description: 'Include --delete flag in AWS deployments'
        type: boolean
        default: false

jobs:
  deploy-aux-production:
    if: inputs.deploy_aux
    uses: utah-marketing-communications/uu-workflow-templates/.github/workflows/deploy-aws.yml@main
    with:
      environment: aux-production
      server_path: /var/www/html/aux.umc.utah.edu/wp-content/plugins/uu-so-widgets/uu-so-widgets-bundle/your-widget-name
      tag_name: ${{ github.event.inputs.tag }}
      write_version_file: true
      delete: ${{ github.event.inputs.aws_delete == 'true' }}
    secrets:
      SSH_PRIVATE_KEY: ${{ secrets.AUX_PROD_SSH_KEY }}
      SSH_PORT: ${{ secrets.AUX_PROD_SSH_PORT }}
      SSH_USER: ${{ secrets.AUX_PROD_SSH_USER }}
      SSH_HOST: ${{ secrets.AUX_PROD_SSH_HOST }}
```

## 🔒 Safety Features

- **Safe defaults**: All delete flags default to `false`
- **Conditional execution**: Only selected environments deploy
- **Input validation**: Required fields prevent misconfigurations
- **Error handling**: Graceful failure with clear error messages

## 📝 Version Control

- Use semantic versioning for template releases
- Pin to specific versions in widget repos for stability
- Test changes in staging before updating production workflows

## 🤝 Contributing

1. Make changes to the reusable workflows
2. Test with a staging widget repository
3. Create a pull request with detailed description
4. Tag a new release after approval

## 📞 Support

For questions or issues with these workflow templates, please contact the UU development team.
