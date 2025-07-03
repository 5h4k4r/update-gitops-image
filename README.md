# Update GitOps Image GitHub Action

The Update GitOps GitHub Action is a custom composite Action designed to automate the process of updating an image tag in a GitOps repository. It streamlines the following tasks:

1. Creating a new branch.
2. Updating the image tag in the specified GitOps manifest file.
3. Creating a pull request for the changes.
4. Automatically merging the pull request into the configured branch.

## Description

This Action simplifies the process of updating an image tag in a GitOps repository. It's designed to be used in conjunction with a GitOps workflow to automate image tag updates. When a pull request is created, the PR URL will be displayed in the GitHub Actions job summary for easy access.

The Action performs the following tasks:
1. Creates a new branch (if it doesn't exist)
2. Updates the image tag in the specified GitOps manifest file
3. Creates a pull request for the changes
4. Automatically merges the pull request for development environments
5. Displays the PR URL in the GitHub Actions job summary

## Authentication Methods

This Action supports two authentication methods for accessing the GitOps repository:

### 1. GitHub App Authentication (Recommended)
Uses a GitHub App with the `app-id` and `private-key` inputs. This method is recommended for most use cases as it provides fine-grained permissions and better security.

### 2. Deploy Key Authentication
Uses a deploy key with the `deploy-private-key` input. This method is useful when you want to use SSH-based authentication or when GitHub App authentication is not available.

## Inputs

This Action accepts the following inputs:

### Required Inputs

1. `repo` (required)
   - Description: The name of the repository on which to update the image tag.

2. `image-tag` (required)
   - Description: The tag of the image to be updated.

3. `component-name` (required)
   - Description: The directory where the component manifests are located.

### Authentication Inputs (choose one method)

**GitHub App Authentication:**
4. `app-id` (required for GitHub App auth)
   - Description: The ID of the GitHub App.

5. `private-key` (required for GitHub App auth)
   - Description: The private key of the GitHub App.

**Deploy Key Authentication:**
6. `deploy-private-key` (optional, alternative to GitHub App auth)
   - Description: The private deploy key for the GitOps repository. When provided, SSH-based authentication will be used instead of GitHub App authentication.

### Optional Inputs

7. `owner` (optional)
   - Description: The owner of the repository if it is not the current one.
   - Default: The owner of the current repository (`${{ github.repository_owner }}`).

8. `overlays-path` (optional)
   - Description: The path where the overlays are stored, it will be suffixed with the component-name.
   - Default: `""` (empty string)

9. `image-tag-file-name` (optional)
   - Description: The name of the file to be updated.
   - Default: `image.yaml`

10. `values-path` (optional)
    - Description: Where the image-tag-path should be applied.
    - Default: `.spec.values`

11. `image-tag-path` (optional)
    - Description: The path to the image tag in the file to be updated.
    - Default: `.global.image.tag`

## Example Usage

### Using GitHub App Authentication

Here's an example of how to include this Action in your GitHub Actions workflow using GitHub App authentication:

```yaml
name: GitOps Image Tag Update

on:
  push:
    branches:
      - dev
    tags:
      - 'v*'  # Trigger on version tags for production releases

jobs:
  update-image-tag:
    runs-on: ubuntu-latest
    # Add a name to make it easier to find in the Actions UI
    name: Update GitOps Image Tag

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Update image tag in GitOps
      id: update-gitops-image
      uses: ditkrg/update-gitops-image@v1
      with:
        owner: ${{ github.repository_owner }}
        repo: my-gitops-repo
        image-tag: ${{ github.ref_name }}  # Use the tag or branch name as image tag
        app-id: ${{ secrets.GITOPS_RUNNER_APP_ID }}
        private-key: ${{ secrets.GITOPS_RUNNER_PRIVATE_KEY }}
        component-name: my-component

    # Optional: Add a step to show the job summary
    - name: Show Job Summary
      if: always()  # Run this step even if previous steps failed
      run: |
        echo "### Job Summary" >> $GITHUB_STEP_SUMMARY
        echo "The PR URL will be automatically displayed in the job summary above."
        echo "You can find the full job summary in the Actions tab of your repository."
```

### Using Deploy Key Authentication

Here's an example using deploy key authentication instead of GitHub App:

```yaml
name: GitOps Image Tag Update with Deploy Key

on:
  push:
    branches:
      - dev
    tags:
      - 'v*'  # Trigger on version tags for production releases

jobs:
  update-image-tag:
    runs-on: ubuntu-latest
    name: Update GitOps Image Tag

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Update image tag in GitOps
      id: update-gitops-image
      uses: ditkrg/update-gitops-image@v1
      with:
        owner: ${{ github.repository_owner }}
        repo: my-gitops-repo
        image-tag: ${{ github.ref_name }}
        deploy-private-key: ${{ secrets.GITOPS_DEPLOY_KEY }}
        component-name: my-component
        # Optional: customize the file structure
        overlays-path: "overlays/"
        image-tag-file-name: "values.yaml"
        values-path: ".spec.template.spec.containers[0]"
        image-tag-path: ".image.tag"
```

### Advanced Configuration Example

Here's an example with all optional parameters configured:

```yaml
name: GitOps Image Tag Update - Advanced

on:
  push:
    branches:
      - dev
    tags:
      - 'v*'

jobs:
  update-image-tag:
    runs-on: ubuntu-latest
    name: Update GitOps Image Tag

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Update image tag in GitOps
      id: update-gitops-image
      uses: ditkrg/update-gitops-image@v1
      with:
        owner: my-org
        repo: my-gitops-repo
        image-tag: ${{ github.ref_name }}
        app-id: ${{ secrets.GITOPS_RUNNER_APP_ID }}
        private-key: ${{ secrets.GITOPS_RUNNER_PRIVATE_KEY }}
        component-name: backend-service
        overlays-path: "environments/"
        image-tag-file-name: "kustomization.yaml"
        values-path: ".spec.values"
        image-tag-path: ".backend.image.tag"
```

## Setup Instructions

### Setting up GitHub App Authentication

1. Create a GitHub App in your organization or personal account
2. Install the app on the GitOps repository
3. Grant the app the following permissions:
   - Repository permissions: Contents (Read & Write), Pull requests (Read & Write), Metadata (Read)
4. Generate a private key for the app
5. Add the following secrets to your repository:
   - `GITOPS_RUNNER_APP_ID`: The GitHub App ID
   - `GITOPS_RUNNER_PRIVATE_KEY`: The private key content

### Setting up Deploy Key Authentication

1. Generate an SSH key pair:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com" -f gitops_deploy_key
   ```
2. Add the public key (`gitops_deploy_key.pub`) as a deploy key in your GitOps repository settings
3. Ensure the deploy key has write access enabled
4. Add the private key content as a secret to your repository:
   - `GITOPS_DEPLOY_KEY`: The private key content

## Environment-Based Deployment

The Action automatically determines the target environment based on the Git reference:

- **Production**: Triggered by tags (e.g., `v1.0.0`)
- **Staging**: Triggered by the `main` branch
- **Development**: Triggered by any other branch

The Action will update the appropriate overlay directory and create environment-specific branches and pull requests.

### Viewing the Job Summary

After the workflow runs, you can view the job summary in two ways:

1. **In the Actions UI**:
   - Go to your repository's "Actions" tab
   - Click on the workflow run
   - The job summary will be visible at the top of the job output
   - The PR URL will be displayed under the "Pull Request Created" section

2. **In the Pull Request**:
   - If the action creates a PR, the job summary will also be visible in the PR's checks section
   - Click on the "Details" link of the workflow run to see the full summary

The job summary is automatically generated and includes:
- The PR URL when a pull request is created
- Any other information added to the summary by the action
- A link to the full workflow run

## Troubleshooting

### Common Issues

1. **Authentication Errors**:
   - Ensure your GitHub App has the correct permissions
   - Verify that the deploy key has write access enabled
   - Check that the secrets are correctly set in your repository

2. **File Not Found Errors**:
   - Verify that the `overlays-path` and `image-tag-file-name` point to existing files
   - Check that the `component-name` matches your directory structure

3. **YAML Path Errors**:
   - Ensure that `values-path` and `image-tag-path` use valid yq syntax
   - Test your YAML paths with yq locally before using them in the action

### Security Considerations

- Use GitHub App authentication when possible for better security and audit trails
- Regularly rotate your deploy keys and GitHub App private keys
- Use repository secrets to store sensitive information
- Consider using environment-specific secrets for different deployment targets
