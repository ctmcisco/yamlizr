# Yamlizr - Azure DevOps Classic-to-YAML Pipelines CLI

[azdo-badge]: https://dev.azure.com/f2calv/github/_apis/build/status/f2calv.CasCap.DevOpsYamlizrCli?branchName=master
[azdo-url]: https://dev.azure.com/f2calv/github/_build/latest?definitionId=8&branchName=master
[azdo-coverage-url]: https://img.shields.io/azure-devops/coverage/f2calv/github/8
[cascap.yamlizr-badge]: https://img.shields.io/nuget/v/yamlizr?color=blue
[cascap.yamlizr-url]: https://nuget.org/packages/yamlizr

[![Build Status][azdo-badge]][azdo-url] <!-- ![Code Coverage][azdo-coverage-url] --> [![Nuget][cascap.yamlizr-badge]][cascap.yamlizr-url]

**yamlizr** is a [.NET Core Global Tool](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) which converts Azure DevOps Classic Build/Release Definitions and any referenced Task Groups en-masse into their [YAML Pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema) or [GitHub Action](https://github.com/features/actions) equivalent.

Currently (as of August 2020) [Azure DevOps](https://dev.azure.com) only allows you to view the YAML for individual tasks in your definitions, however it doesn't allow the exporting of entire pipelines as YAML nor does it support task group-to-YAML conversion ...so hopefully this tool fills that gap.

The tool itself uses the [Azure DevOps .NET Client Libraries](https://docs.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries?view=azure-devops) to pre-cache relevant data from your Azure DevOps organisation/account. This data includes build/release definitions, task groups, tasks/extensions data and variable groups. This Azure DevOps data is converted into Azure DevOps Pipeline objects (stages/jobs/steps) which are then persisted to YAML using the [YamlDotNet](https://github.com/aaubry/YamlDotNet) library.

This _is not_ a delicate tool to create perfectly constructed YAML pipelines. Instead consider it to be a hammer which will spawn as many YAML files as possible and from this YAML you can pick/choose and copy/paste the relevant stages/jobs/steps into your own preferred YAML CI/CD deployment architecture.

Also... there is an optional switch whereby the tool can pass the Azure DevOps Pipeline objects into the [AzurePipelinesToGitHubActionsConverter library](https://github.com/samsmithnz/AzurePipelinesToGitHubActionsConverter) (by @samsmithnz) and export your pipelines as GitHub Actions YAML.

**Disclaimer**: _Do not consider any of the YAML generated by this tool to be 'production ready'. Do your own testing/research and [post any issues](https://github.com/f2calv/yamlizr/issues) or make a PR!_

## Installation/Set-up

- [Create a Personal Access Token (PAT)](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) with the following scopes/permissions;
  | Scope             | Permission    |
  | ----------------- | ------------- |
  | Build             | Read          |
  | Deployment Groups | Read & Manage |
  | Release           | Read          |
  | Task Groups       | Read          |
  | Variable Groups   | Read          |
- Download and install [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)
- From a command line shell install the tool;
  `dotnet tool update --global yamlizr`

### CLI Operation

To generate YAML files in the `c:/temp/myoutputfolder` output folder execute the following command;

```pwsh
yamlizr generate -pat <your PAT here> -org <your AzDO organisation> -proj <your AzDO project> -out c:/temp/myoutputfolder
```

For context-sensitive help execute;
```
yamlizr --help
```

Additional switches;
- `--inline` merge the tasks from task groups in-line rather than by creating templates.
- `--githubactions` generate GitHub Actions code.
- `--filter` filter build/release definitions.

To generate both Azure Pipelines and GitHub Actions YAML for a build definition called 'wibble-CI' and a release definition called 'wibble-CD';
```pwsh
yamlizr generate -pat <your PAT here> -org <your AzDO organisation> -proj <your AzDO project> -out c:/temp/myoutputfolder --filter wibble --githubactions
```

All YAML files generated are output into sub-folders of a project folder, i.e. using the above example of `-o c:/temp/myoutputfolder` the following folders are created;
- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsBuilds/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsReleases/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/AzureDevOpsTaskGroups/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/GitHubBuilds/*.yml`
- `c:/temp/myoutputfolder/<your AzDO project>/GitHubReleases/*.yml`


### Core Dependencies

- [Azure DevOps .NET Client Libraries](https://docs.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries?view=azure-devops)
- [AzurePipelinesToGitHubActionsConverter](https://github.com/samsmithnz/AzurePipelinesToGitHubActionsConverter)
- [YamlDotNet](https://github.com/aaubry/YamlDotNet)
- [CommandLineUtils](https://github.com/natemcmaster/CommandLineUtils)
- [ShellProgressBar](https://github.com/Mpdreamz/shellprogressbar)

### Misc Tips

- The [NuGet package](https://www.nuget.org/packages/yamlizr/) includes [SourceLink](https://github.com/dotnet/sourcelink) which enables you to jump inside the library and debug the API yourself. By default Visual Studio 2017/2019 does not allow this and will pop up an message "You are debugging a Release build of...", to disable this message go into the Visual Studio debugging options and un-check the 'Just My Code' option (menu path, Tools > Options > Debugging).

### Known Issues

- ShellProgressBar gives random formatting problems.
- `--parallelism` command line option for faster processing is a bit buggy so disabled.
- Various YAML structures are 'missing', PR's welcome.

### Feedback/Issues

Please post any issues or feedback [here](https://github.com/f2calv/yamlizr/issues).

### License

yamlizr is Copyright &copy; 2020 [Alex Vincent](https://github.com/f2calv) under the [MIT license](LICENSE).
