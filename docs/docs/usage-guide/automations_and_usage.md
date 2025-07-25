## Local repo (CLI)

When running from your locally cloned Qodo Merge repo (CLI), your local configuration file will be used.
Examples of invoking the different tools via the CLI:

- **Review**:       `python -m pr_agent.cli --pr_url=<pr_url>  review`
- **Describe**:     `python -m pr_agent.cli --pr_url=<pr_url>  describe`
- **Improve**:      `python -m pr_agent.cli --pr_url=<pr_url>  improve`
- **Ask**:          `python -m pr_agent.cli --pr_url=<pr_url>  ask "Write me a poem about this PR"`
- **Update Changelog**:      `python -m pr_agent.cli --pr_url=<pr_url>  update_changelog`

`<pr_url>` is the url of the relevant PR (for example: [#50](https://github.com/Codium-ai/pr-agent/pull/50)).

**Notes:**

1. in addition to editing your local configuration file, you can also change any configuration value by adding it to the command line:

```
python -m pr_agent.cli --pr_url=<pr_url>  /review --pr_reviewer.extra_instructions="focus on the file: ..."
```

2. You can print results locally, without publishing them, by setting in `configuration.toml`:

```
[config]
publish_output=false
verbosity_level=2
```

This is useful for debugging or experimenting with different tools.

3. **git provider**: The [git_provider](https://github.com/Codium-ai/pr-agent/blob/main/pr_agent/settings/configuration.toml#L5) field in a configuration file determines the GIT provider that will be used by Qodo Merge. Currently, the following providers are supported:
`github` **(default)**, `gitlab`, `bitbucket`, `azure`, `codecommit`, `local`, and `gitea`.

### CLI Health Check

To verify that Qodo Merge has been configured correctly, you can run this health check command from the repository root:

```bash
python -m tests.health_test.main
```

If the health check passes, you will see the following output:

```
========
Health test passed successfully
========
```

At the end of the run.

Before running the health check, ensure you have:

- Configured your [LLM provider](https://qodo-merge-docs.qodo.ai/usage-guide/changing_a_model/)
- Added a valid GitHub token to your configuration file

## Online usage

Online usage means invoking Qodo Merge tools by [comments](https://github.com/Codium-ai/pr-agent/pull/229#issuecomment-1695021901) on a PR.
Commands for invoking the different tools via comments:

- **Review**:       `/review`
- **Describe**:     `/describe`
- **Improve**:      `/improve`  (or `/improve_code` for bitbucket, since `/improve` is sometimes reserved)
- **Ask**:          `/ask "..."`
- **Update Changelog**:      `/update_changelog`

To edit a specific configuration value, just add `--config_path=<value>` to any command.
For example, if you want to edit the `review` tool configurations, you can run:

```
/review --pr_reviewer.extra_instructions="..." --pr_reviewer.require_score_review=false
```

Any configuration value in [configuration file](https://github.com/Codium-ai/pr-agent/blob/main/pr_agent/settings/configuration.toml) file can be similarly edited. Comment `/config` to see the list of available configurations.

## Qodo Merge Automatic Feedback

### Disabling all automatic feedback

To easily disable all automatic feedback from Qodo Merge (GitHub App, GitLab Webhook, BitBucket App, Azure DevOps Webhook), set in a configuration file:

```toml
[config]
disable_auto_feedback = true
```

When this parameter is set to `true`, Qodo Merge will not run any automatic tools (like `describe`, `review`, `improve`) when a new PR is opened, or when new code is pushed to an open PR.

### GitHub App

!!! note "Configurations for Qodo Merge"
    Qodo Merge for GitHub is an App, hosted by Qodo. So all the instructions below are relevant also for Qodo Merge users.
    Same goes for [GitLab webhook](#gitlab-webhook) and [BitBucket App](#bitbucket-app) sections.

#### GitHub app automatic tools when a new PR is opened

The [github_app](https://github.com/Codium-ai/pr-agent/blob/main/pr_agent/settings/configuration.toml#L220) section defines GitHub app specific configurations.

The configuration parameter `pr_commands` defines the list of tools that will be **run automatically** when a new PR is opened:

```toml
[github_app]
pr_commands = [
    "/describe",
    "/review",
    "/improve",
]
```

This means that when a new PR is opened/reopened or marked as ready for review, Qodo Merge will run the `describe`, `review` and `improve` tools.  

**Draft PRs:** 

By default, draft PRs are not considered for automatic tools, but you can change this by setting the `feedback_on_draft_pr` parameter to `true` in the configuration file.

```toml
[github_app]
feedback_on_draft_pr = true
```

**Changing default tool parameters:**

You can override the default tool parameters by using one the three options for a [configuration file](https://qodo-merge-docs.qodo.ai/usage-guide/configuration_options/): **wiki**, **local**, or **global**.
For example, if your configuration file contains:

```toml
[pr_description]
generate_ai_title = true
```

Every time you run the `describe` tool (including automatic runs) the PR title will be generated by the AI.


**Parameters for automated runs:**

You can customize configurations specifically for automated runs by using the `--config_path=<value>` parameter.
For instance, to modify the `review` tool settings only for newly opened PRs, use:

```toml
[github_app]
pr_commands = [
    "/describe",
    "/review --pr_reviewer.extra_instructions='focus on the file: ...'",
    "/improve",
]
```

#### GitHub app automatic tools for push actions (commits to an open PR)

In addition to running automatic tools when a PR is opened, the GitHub app can also respond to new code that is pushed to an open PR.

The configuration toggle `handle_push_trigger` can be used to enable this feature.
The configuration parameter `push_commands` defines the list of tools that will be **run automatically** when new code is pushed to the PR.

```toml
[github_app]
handle_push_trigger = true
push_commands = [
    "/describe",
    "/review",
]
```

This means that when new code is pushed to the PR, the Qodo Merge will run the `describe` and `review` tools, with the specified parameters.

### GitHub Action

`GitHub Action` is a different way to trigger Qodo Merge tools, and uses a different configuration mechanism than `GitHub App`.<br>
You can configure settings for `GitHub Action` by adding environment variables under the env section in `.github/workflows/pr_agent.yml` file.
Specifically, start by setting the following environment variables:

```yaml
      env:
        OPENAI_KEY: ${{ secrets.OPENAI_KEY }} # Make sure to add your OpenAI key to your repo secrets
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # Make sure to add your GitHub token to your repo secrets
        github_action_config.auto_review: "true" # enable\disable auto review
        github_action_config.auto_describe: "true" # enable\disable auto describe
        github_action_config.auto_improve: "true" # enable\disable auto improve
        github_action_config.pr_actions: '["opened", "reopened", "ready_for_review", "review_requested"]'
```

`github_action_config.auto_review`, `github_action_config.auto_describe` and `github_action_config.auto_improve` are used to enable/disable automatic tools that run when a new PR is opened.
If not set, the default configuration is for all three tools to run automatically when a new PR is opened.

`github_action_config.pr_actions` is used to configure which `pull_requests` events will trigger the enabled auto flags
If not set, the default configuration is `["opened", "reopened", "ready_for_review", "review_requested"]`

`github_action_config.enable_output` are used to enable/disable github actions [output parameter](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-docker-container-and-javascript-actions) (default is `true`).
Review result is output as JSON to `steps.{step-id}.outputs.review` property.
The JSON structure is equivalent to the yaml data structure defined in [pr_reviewer_prompts.toml](https://github.com/qodo-ai/pr-agent/blob/main/pr_agent/settings/pr_reviewer_prompts.toml).

Note that you can give additional config parameters by adding environment variables to `.github/workflows/pr_agent.yml`, or by using a `.pr_agent.toml` [configuration file](https://qodo-merge-docs.qodo.ai/usage-guide/configuration_options/#global-configuration-file) in the root of your repo

For example, you can set an environment variable: `pr_description.publish_labels=false`, or add a `.pr_agent.toml` file with the following content:

```toml
[pr_description]
publish_labels = false
```

to prevent Qodo Merge from publishing labels when running the `describe` tool.

#### Quick Reference: Model Configuration in GitHub Actions

For detailed step-by-step examples of configuring different models (Gemini, Claude, Azure OpenAI, etc.) in GitHub Actions, see the [Configuration Examples](../installation/github.md#configuration-examples) section in the installation guide.

**Common Model Configuration Patterns:**

- **OpenAI**: Set `config.model: "gpt-4o"` and `OPENAI_KEY`
- **Gemini**: Set `config.model: "gemini/gemini-1.5-flash"` and `GOOGLE_AI_STUDIO.GEMINI_API_KEY` (no `OPENAI_KEY` needed)
- **Claude**: Set `config.model: "anthropic/claude-3-opus-20240229"` and `ANTHROPIC.KEY` (no `OPENAI_KEY` needed)
- **Azure OpenAI**: Set `OPENAI.API_TYPE: "azure"`, `OPENAI.API_BASE`, and `OPENAI.DEPLOYMENT_ID`
- **Local Models**: Set `config.model: "ollama/model-name"` and `OLLAMA.API_BASE`

**Environment Variable Format:**
- Use dots (`.`) to separate sections and keys: `config.model`, `pr_reviewer.extra_instructions`
- Boolean values as strings: `"true"` or `"false"`
- Arrays as JSON strings: `'["item1", "item2"]'`

For complete model configuration details, see [Changing a model in PR-Agent](changing_a_model.md).

### GitLab Webhook

After setting up a GitLab webhook, to control which commands will run automatically when a new MR is opened, you can set the `pr_commands` parameter in the configuration file, similar to the GitHub App:

```toml
[gitlab]
pr_commands = [
    "/describe",
    "/review",
    "/improve",
]
```

the GitLab webhook can also respond to new code that is pushed to an open MR.
The configuration toggle `handle_push_trigger` can be used to enable this feature.
The configuration parameter `push_commands` defines the list of tools that will be **run automatically** when new code is pushed to the MR.

```toml
[gitlab]
handle_push_trigger = true
push_commands = [
    "/describe",
    "/review",
]
```

Note that to use the 'handle_push_trigger' feature, you need to give the gitlab webhook also the "Push events" scope.

### BitBucket App

Similar to GitHub app, when running Qodo Merge from BitBucket App, the default [configuration file](https://github.com/Codium-ai/pr-agent/blob/main/pr_agent/settings/configuration.toml) will be initially loaded.

By uploading a local `.pr_agent.toml` file to the root of the repo's default branch, you can edit and customize any configuration parameter. Note that you need to upload `.pr_agent.toml` prior to creating a PR, in order for the configuration to take effect.

For example, if your local `.pr_agent.toml` file contains:

```toml
[pr_reviewer]
extra_instructions = "Answer in japanese"
```

Each time you invoke a `/review` tool, it will use the extra instructions you set in the local configuration file.

Note that among other limitations, BitBucket provides relatively low rate-limits for applications (up to 1000 requests per hour), and does not provide an API to track the actual rate-limit usage.
If you experience a lack of responses from Qodo Merge, you might want to set: `bitbucket_app.avoid_full_files=true` in your configuration file.
This will prevent Qodo Merge from acquiring the full file content, and will only use the diff content. This will reduce the number of requests made to BitBucket, at the cost of small decrease in accuracy, as dynamic context will not be applicable.

#### BitBucket Self-Hosted App automatic tools

To control which commands will run automatically when a new PR is opened, you can set the `pr_commands` parameter in the configuration file:
Specifically, set the following values:

```toml
[bitbucket_app]
pr_commands = [
    "/review",
    "/improve --pr_code_suggestions.commitable_code_suggestions=true --pr_code_suggestions.suggestions_score_threshold=7",
]
```

Note that we set specifically for bitbucket, we recommend using: `--pr_code_suggestions.suggestions_score_threshold=7` and that is the default value we set for bitbucket.
Since this platform only supports inline code suggestions, we want to limit the number of suggestions, and only present a limited number.

To enable BitBucket app to respond to each **push** to the PR, set (for example):

```toml
[bitbucket_app]
handle_push_trigger = true
push_commands = [
    "/describe",
    "/review",
]
```

### Azure DevOps provider

To use Azure DevOps provider use the following settings in configuration.toml:

```toml
[config]
git_provider="azure"
```

Azure DevOps provider supports [PAT token](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows) or [DefaultAzureCredential](https://learn.microsoft.com/en-us/azure/developer/python/sdk/authentication-overview#authentication-in-server-environments) authentication.
PAT is faster to create, but has build in expiration date, and will use the user identity for API calls.
Using DefaultAzureCredential you can use managed identity or Service principle, which are more secure and will create separate ADO user identity (via AAD) to the agent.

If PAT was chosen, you can assign the value in .secrets.toml.
If DefaultAzureCredential was chosen, you can assigned the additional env vars like AZURE_CLIENT_SECRET directly,
or use managed identity/az cli (for local development) without any additional configuration.
in any case, 'org' value must be assigned in .secrets.toml:

```
[azure_devops]
org = "https://dev.azure.com/YOUR_ORGANIZATION/"
# pat = "YOUR_PAT_TOKEN" needed only if using PAT for authentication
```

#### Azure DevOps Webhook

To control which commands will run automatically when a new PR is opened, you can set the `pr_commands` parameter in the configuration file, similar to the GitHub App:

```toml
[azure_devops_server]
pr_commands = [
    "/describe",
    "/review",
    "/improve",
]
```

### Gitea Webhook

After setting up a Gitea webhook, to control which commands will run automatically when a new MR is opened, you can set the `pr_commands` parameter in the configuration file, similar to the GitHub App:

```toml
[gitea]
pr_commands = [
    "/describe",
    "/review",
    "/improve",
]
```
