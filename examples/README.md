# Usage Examples

This directory contains examples of how to use the generic workflow templates.

## 🚀 Quick Start Examples

### Example 1: Staging Deployment

Create a `deploy-staging.yml` in your widget repository:

```yaml
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
      aws_delete: false
      wp_delete: true
      wp_cache_clear: false
    secrets: inherit
```

### Example 2: Production Deployment

Create a `deploy-production.yml` in your widget repository:

```yaml
name: PROD Deploy My Widget to Production

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
      deploy_wp_engine:
        description: 'Deploy to WP Engine production'
        type: boolean
        default: false
      aws_delete:
        description: 'Include --delete flag in AWS deployments'
        type: boolean
        default: false
      wp_dry_run:
        description: 'Run WP Engine deployment in dry run mode'
        type: boolean
        default: false

jobs:
  deploy:
    uses: utah-marketing-communications/uu-workflow-templates/.github/workflows/deploy-production.yml@main
    with:
      widget_name: my-widget-name
      tag_name: ${{ github.event.inputs.tag }}
      deploy_aux: ${{ github.event.inputs.deploy_aux }}
      deploy_wp_engine: ${{ github.event.inputs.deploy_wp_engine }}
      aws_delete: ${{ github.event.inputs.aws_delete }}
      wp_dry_run: ${{ github.event.inputs.wp_dry_run }}
    secrets: inherit
```

## 🔧 Customization Options

### Widget-Specific Configuration

Replace `my-widget-name` with your actual widget name in the server paths:

```yaml
# The template automatically constructs the path:
# /var/www/html/aux.umc.utah.edu/wp-content/plugins/uu-so-widgets/uu-so-widgets-bundle/my-widget-name
```

### Environment Selection

Control which environments to deploy to:

```yaml
with:
  deploy_aux: true        # Deploy to AUX
  deploy_bryce: false      # Skip BRYCE
  deploy_cap_reef: true    # Deploy to CAP REEF
  deploy_zion: false       # Skip ZION
  deploy_wp_engine: true   # Deploy to WP Engine
```

### Safety Settings

Configure deployment safety:

```yaml
with:
  aws_delete: false        # Don't delete files on AWS (safe)
  wp_delete: true          # Delete files on WP Engine
  wp_cache_clear: false     # Don't clear cache
  wp_dry_run: false        # Real deployment (not dry run)
```

## 📋 Required Secrets

Ensure your repository has these secrets configured:

### AWS Secrets (for each environment):
- `AUX_STG_SSH_PRIVATE_KEY`, `AUX_STG_SSH_PORT`, `AUX_STG_SSH_USER`, `AUX_STG_SSH_HOST`
- `AUX_PROD_SSH_KEY`, `AUX_PROD_SSH_PORT`, `AUX_PROD_SSH_USER`, `AUX_PROD_SSH_HOST`
- (Repeat for BRYCE, CAP_REEF, ZION)

### WP Engine Secrets:
- `WPE_ORG_API_USER`
- `WPE_ORG_API_PASS`
- `WPE_ORG_SSHG_KEY_PRIVATE`

## 🎯 Best Practices

1. **Start with staging**: Test your configuration with staging deployments first
2. **Use safe defaults**: Keep `aws_delete: false` until you're confident
3. **Test one environment**: Deploy to a single environment before scaling up
4. **Monitor deployments**: Check the Actions tab for deployment status
5. **Version control**: Pin to specific template versions for stability

## 🔍 Troubleshooting

### Common Issues:

1. **Secret not found**: Ensure all required secrets are configured
2. **Path not found**: Verify the widget name is correct
3. **Permission denied**: Check SSH key permissions
4. **Deployment failed**: Review the workflow logs for specific errors

### Getting Help:

- Check the main README.md for detailed documentation
- Review the workflow logs in GitHub Actions
- Contact the UU development team for support
