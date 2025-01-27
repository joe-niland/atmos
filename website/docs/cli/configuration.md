---
title: Configuration
---

<head>
  <title>CLI Configuration</title>
  <meta
    name="description"
    content="CLI Configuration."
  />
</head>

## atmos.yaml

```yaml
# CLI config is loaded from the following locations (from lowest to highest priority):
# system dir (`/usr/local/etc/atmos` on Linux, `%LOCALAPPDATA%/atmos` on Windows)
# home dir (~/.atmos)
# current directory
# ENV vars
# Command-line arguments
#
# It supports POSIX-style Globs for file names/paths (double-star `**` is supported)
# https://en.wikipedia.org/wiki/Glob_(programming)

# Base path for components, stacks and workflows configurations.
# Can also be set using `ATMOS_BASE_PATH` ENV var, or `--base-path` command-line argument.
# Supports both absolute and relative paths.
# If not provided or is an empty string, `components.terraform.base_path`, `components.helmfile.base_path`, `stacks.base_path` and `workflows.base_path`
# are independent settings (supporting both absolute and relative paths).
# If `base_path` is provided, `components.terraform.base_path`, `components.helmfile.base_path`, `stacks.base_path` and `workflows.base_path`
# are considered paths relative to `base_path`.
base_path: "."

components:
  terraform:
    # Can also be set using `ATMOS_COMPONENTS_TERRAFORM_BASE_PATH` ENV var, or `--terraform-dir` command-line argument
    # Supports both absolute and relative paths
    base_path: "components/terraform"
    # Can also be set using `ATMOS_COMPONENTS_TERRAFORM_APPLY_AUTO_APPROVE` ENV var
    apply_auto_approve: false
    # Can also be set using `ATMOS_COMPONENTS_TERRAFORM_DEPLOY_RUN_INIT` ENV var, or `--deploy-run-init` command-line argument
    deploy_run_init: true
    # Can also be set using `ATMOS_COMPONENTS_TERRAFORM_INIT_RUN_RECONFIGURE` ENV var, or `--init-run-reconfigure` command-line argument
    init_run_reconfigure: true
    # Can also be set using `ATMOS_COMPONENTS_TERRAFORM_AUTO_GENERATE_BACKEND_FILE` ENV var, or `--auto-generate-backend-file` command-line argument
    auto_generate_backend_file: true
  helmfile:
    # Can also be set using `ATMOS_COMPONENTS_HELMFILE_BASE_PATH` ENV var, or `--helmfile-dir` command-line argument
    # Supports both absolute and relative paths
    base_path: "components/helmfile"
    # Can also be set using `ATMOS_COMPONENTS_HELMFILE_KUBECONFIG_PATH` ENV var
    kubeconfig_path: "/dev/shm"
    # Can also be set using `ATMOS_COMPONENTS_HELMFILE_HELM_AWS_PROFILE_PATTERN` ENV var
    helm_aws_profile_pattern: "{namespace}-{tenant}-gbl-{stage}-helm"
    # Can also be set using `ATMOS_COMPONENTS_HELMFILE_CLUSTER_NAME_PATTERN` ENV var
    cluster_name_pattern: "{namespace}-{tenant}-{environment}-{stage}-eks-cluster"

stacks:
  # Can also be set using `ATMOS_STACKS_BASE_PATH` ENV var, or `--config-dir` and `--stacks-dir` command-line arguments
  # Supports both absolute and relative paths
  base_path: "stacks"
  # Can also be set using `ATMOS_STACKS_INCLUDED_PATHS` ENV var (comma-separated values string)
  included_paths:
    - "orgs/**/*"
  # Can also be set using `ATMOS_STACKS_EXCLUDED_PATHS` ENV var (comma-separated values string)
  excluded_paths:
    - "**/_defaults.yaml"
  # Can also be set using `ATMOS_STACKS_NAME_PATTERN` ENV var
  name_pattern: "{tenant}-{environment}-{stage}"

workflows:
  # Can also be set using `ATMOS_WORKFLOWS_BASE_PATH` ENV var, or `--workflows-dir` command-line arguments
  # Supports both absolute and relative paths
  base_path: "stacks/workflows"

logs:
  verbose: false
  colors: true

# Custom CLI commands
commands:
  - name: tf
    description: Execute terraform commands
    # subcommands
    commands:
      - name: plan
        description: This command plans terraform components
        arguments:
          - name: component
            description: Name of the component
        flags:
          - name: stack
            shorthand: s
            description: Name of the stack
            required: true
        env:
          - key: ENV_VAR_1
            value: ENV_VAR_1_value
          - key: ENV_VAR_2
            # `valueCommand` is an external command to execute to get the value for the ENV var
            # Either 'value' or 'valueCommand' can be specified for the ENV var, but not both
            valueCommand: echo ENV_VAR_2_value
        # steps support Go templates
        steps:
          - atmos terraform plan {{ .Arguments.component }} -s {{ .Flags.stack }}
  - name: terraform
    description: Execute terraform commands
    # subcommands
    commands:
      - name: provision
        description: This command provisions terraform components
        arguments:
          - name: component
            description: Name of the component
        flags:
          - name: stack
            shorthand: s
            description: Name of the stack
            required: true
        # ENV var values support Go templates
        env:
          - key: ATMOS_COMPONENT
            value: "{{ .Arguments.component }}"
          - key: ATMOS_STACK
            value: "{{ .Flags.stack }}"
        steps:
          - atmos terraform plan $ATMOS_COMPONENT -s $ATMOS_STACK
          - atmos terraform apply $ATMOS_COMPONENT -s $ATMOS_STACK
  - name: play
    description: This command plays games
    steps:
      - echo Playing...
    # subcommands
    commands:
      - name: hello
        description: This command says Hello world
        steps:
          - echo Saying Hello world...
          - echo Hello world
      - name: ping
        description: This command plays ping-pong
        steps:
          - echo Playing ping-pong...
          - echo pong
```
