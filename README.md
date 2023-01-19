# Wizcli Wrapper v1

This is a repository for the **Wizcli Wrapper v1** GitHub action. 

You can use this action for the following:
- scanning Infrastructure as Code (IaC) files in your repository for vulnerabilities and compliance issues:
  - Terraform	-> HCL (.tf) files
  - Terraform	-> JSON output of Terraform plan. Generate this file in your plan stage using this command: `terraform plan -out plan.tfplan && terraform show -json plan.tfplan > plan.tfplanjson`
  - AWS CloudFormation	-> JSON or YAML files
  - Azure Resource Manager	-> JSON files
  - Kubernetes	-> YAML manifest files
  - Helm -> YAML chart files
  - Docker -> Files named Dockerfile or with a .dockerfile extension
- scanning local Docker images for vulnerabilities
  - The wiz-cli scans local Docker images, analyzing binaries and packages in the container image, performing full vulnerability and secrets scans, and then outputting the results to the terminal
- scanning for secrets in your repository and Docker images
  - the wizcli will search for secrets during the IaC and the Docker scan by default

Not supported:

- scanning of VM Images
- scanning of Virtual Machines

# Prerequisites

- Wiz [service account](https://docs.wiz.io/wiz-docs/docs/set-up-wiz-cli#generate-a-wiz-service-account-key)
- The following secrets in GitHub repo or organization. (`WP only`) We already completed this step for your Github EMU Organization and added the secrets
   - `WIZ_CLIENT_ID`
   - `WIZ_CLIENT_SECRET`
- Linux runner (Ubuntu, Debian, CentOS, etc.)

# Usage

```yaml
  uses: aleksei-aikashev/wizcli-wrapper@v1
  with: 
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}

```

# Table of Contents

- [Wizcli Wrapper v1](#wizcli-wrapper-v1)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
- [Table of Contents](#table-of-contents)
  - [All default values expanded](#all-default-values-expanded)
- [Scenarios](#scenarios)
  - [Scan only IaC with custom policy](#scan-only-iac-with-custom-policy)
  - [Scan only Docker images with custom policy and relative Dockerfile path](#scan-only-docker-images-with-custom-policy-and-relative-dockerfile-path)
  - [Scanning for secrets and breaking the build](#scanning-for-secrets-and-breaking-the-build)
- [Development](#development)
  - [Contributing](#contributing)
  - [Release instructions](#release-instructions)
- [License](#license)

## All default values expanded

This section contains all inputs of the action and their default values. You can use this section as a template for your workflow file to modify some of the values. You can find detailed descriptions of the inputs in the [action.yml](./action.yml) file.

```yaml
name: Wiz Full Scan
on:
  pull_request:
    paths:
      - "**.tf**"
      - "**.tfvars"
      - "**.tfplanjson"
      - "**.json"
      - "**.yaml"
      - "**.yml"
      - "**Dockerfile"
      - "**.dockerfile"
  push:
    branches:
      - main

jobs:
  wiz-full-scan:
    name: wiz-full-scan
    runs-on: [REPLACE WITH YOUR RUNNER] # e.g. [ubuntu-latest] or [self-hosted,sg-kubernetes,ap-northeast-1]
    steps:
      
      # Checkout the repository to the GitHub Actions runner
      - name: Check out repository
        uses: actions/checkout@v3

      # Run both wiz iac and docker scans, and upload the results to Wiz
      - name: Wiz Full Scan With Default Values
        uses: aleksei-aikashev/wizcli-wrapper@v1
        with: 
          # IaC scan defaults
          iac_scan_path: "."
          wiz_iac_policy: "Default IaC policy"
          wiz_iac_report_name: "${{ github.repository }}-${{ github.run_number }}"
          wiz_iac_tags: "repo=${{ github.repository }},commit_sha=${{ github.sha }},pr_title=${{ github.event.pull_request.title }},pr_number=${{ github.event.number}},event_name=${{ github.event_name }},github_workflow=${{ github.workflow }}"
          skip_iac_scan: null
          

          # Docker images vulnerability scan defaults
          docker_scan_path: "."
          wiz_docker_vulnerabilities_policy: "Default vulnerabilities policy"
          wiz_docker_report_name: "${{ github.repository }}-${{ github.run_number }}"
          skip_docker_scan: null

          # Common inputs
          wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
          wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

# Scenarios
## Scan only IaC with custom policy

```yaml
- name: Wiz IaC Scan
  uses: aleksei-aikashev/wizcli-wrapper@v1
  with: 
    wiz_iac_policy: "YOUR_CUSTOM_IAC_POLICY"
    skip_docker_scan: "skip"

    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

## Scan only Docker images with custom policy and relative Dockerfile path

```yaml
- name: Wiz Docker Image Vulnerability Scan
  uses: aleksei-aikashev/wizcli-wrapper@v1
  with: 
    docker_scan_path: "./YOUR_RELATIVE_PATH_TO_DOCKERFILE_FOLDER"
    wiz_iac_policy: "YOUR_CUSTOM_VULN_POLICY"
    skip_iac_scan: "skip"

    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

## Scanning for secrets and breaking the build

```yaml
- name: Wiz Full Scan With Secrets Check
  uses: aleksei-aikashev/wizcli-wrapper@v1
  with: 
    wiz_iac_policy: "Default IaC policy,YOUR_CUSTOM_SECRETS_POLICY"
    wiz_docker_vulnerabilities_policy: "Default vulnerabilities policy,YOUR_CUSTOM_SECRETS_POLICY"
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

# Development
## Contributing

1. Fork the repository 
2. Create a feature branch `feature/your-feature-name`
3. Test locally and create a pull request to the `main` branch
## Release instructions

1. Locate the semantic version of the [upcoming release][release-list] (a draft is maintained by the [`draft-release` workflow][draft-release])
2. Publish the draft release from the `main` branch with semantic version as the tag name, _without_ the checkbox to publish to the GitHub Marketplace checked
3. After publishing the release, the [`release` workflow][release] will automatically run to create/update the corresponding the major version tag such as `v0`


# License

The scripts and documentation in this project are released under the [MIT License](./LICENSE)


<!-- references -->
[release-list]: /releases
[draft-release]: .github/workflows/draft-release.yml
[release]: .github/workflows/release.yml
