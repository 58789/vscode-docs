---
# DO NOT TOUCH — Managed by doc writer
ContentId: 891072bb-c46d-4392-800a-84d747072ce3
DateApproved: 5/15/2019

# Summarize the whole topic in less than 300 characters for SEO purpose
MetaDescription: Use Continuous Integration for testing Visual Studio Code extensions (plug-ins).
---

# Continuous Integration

Extension tests can be run on CI services. The `vscode-test` repository itself contains a sample extension that is tested on Azure Devops Pipelines. You can check out the [build pipeline](https://dev.azure.com/vscode/VSCode/_build?definitionId=14) or jump directly to the [build definition yaml file](https://github.com/microsoft/vscode-test/blob/master/sample/azure-pipelines.yml).

## Azure Pipelines

<a href="https://azure.microsoft.com/services/devops/"><img alt="Azure Pipelines" src="/assets/api/working-with-extensions/continuous-integration/pipelines-logo.png" width="318" /></a>

You can create free projects on [Azure DevOps](https://azure.microsoft.com/services/devops/). This gives you source code hosting, planning boards, building and testing infrastructure, and more. On top of that, you get [10 free parallel jobs](https://azure.microsoft.com/services/devops/pipelines/) for building your projects across all 3 major platforms: Windows, macOS and Linux.

After registering and creating your new project, simply add the following `azure-pipelines.yml` to the root of your extension's repository. Other than the xvfb setup for Linux, the definition is straight-forward:

```yaml
trigger:
- master

strategy:
  matrix:
    linux:
      imageName: 'ubuntu-16.04'
    mac:
      imageName: 'macos-10.13'
    windows:
      imageName: 'vs2017-win2016'

pool:
  vmImage: $(imageName)

steps:

- task: NodeTool@0
  inputs:
    versionSpec: '8.x'
  displayName: 'Install Node.js'

- bash: |
    set -e
    /usr/bin/Xvfb :10 -ac >> /tmp/Xvfb.out 2>&1 &
    disown -ar
    echo "Started xvfb"
  displayName: Start xvfb
  condition: and(succeeded(), eq(variables['Agent.OS'], 'Linux'))

- bash: |
    # vscode-test has its extension located at /sample
    cd sample
    yarn && yarn compile && yarn test
  displayName: Run Tests
  env:
    DISPLAY: :10
```

Next [create a new Pipeline](https://docs.microsoft.com/azure/devops/pipelines/get-started-yaml?view=vsts#get-your-first-build) in your DevOps project and point it to the `azure-pipelines.yml` file. Trigger a build and voilà:

![pipelines](images/continuous-integration/pipelines.png)

You can enable the build to run continuously when pushing to a branch and even on pull requests. [Click here](https://docs.microsoft.com/azure/devops/pipelines/build/triggers) to learn more.
