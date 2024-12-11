# Scan your C, C++, and Objective-C code with SonarQube [![Tests](https://github.com/SonarSource/sonarqube-github-c-cpp/actions/workflows/tests.yml/badge.svg)](https://github.com/SonarSource/sonarqube-github-c-cpp/actions/workflows/tests.yml)

This SonarSource project, available as a GitHub Action, scans your C, C++, and Objective-C projects with [SonarQube Server](https://www.sonarsource.com/products/sonarqube/).

![Logo](./images/SQ_Logo_Cloud_Dark_Backgrounds.png#gh-dark-mode-only)
![Logo](./images/SQ_Logo_Cloud_Light_Backgrounds.png#gh-light-mode-only)

[SonarQube Server](https://www.sonarsource.com/products/sonarqube/) is a widely used static analysis solution for continuous code quality and security inspection.

It helps developers detect coding issues in 30+ languages, frameworks, and IaC platforms, including Java, JavaScript, TypeScript, C#, Python, C, C++, and [many more](https://www.sonarsource.com/knowledge/languages/).

The solution also provides fix recommendations leveraging AI with Sonar's AI CodeFix capability.

## Requirements

To run an analysis on your code, you first need to set up your project on SonarQube Server. Your SonarQube Server instance must be accessible from GitHub, and you will need an access token to run the analysis (more information below under **Environment variables**).

Read more information on how to analyze your code [here](https://docs.sonarsource.com/sonarqube-server/latest/devops-platform-integration/github-integration/introduction/).

## Usage

```properties
sonar.projectKey=<replace with the key generated when setting up the project on SonarQube Server>

# relative paths to source directories. More details and properties are described
# at https://docs.sonarsource.com/sonarqube-server/latest/project-administration/analysis-scope/ 
sonar.sources=.
```

The workflow, usually declared under `.github/workflows`, looks like the following:

```yaml
on:
  # Trigger analysis when pushing to your main branches, and when creating a pull request.
  push:
    branches:
      - main
      - master
      - develop
      - 'releases/**'
  pull_request:
      types: [opened, synchronize, reopened]

name: Main Workflow
jobs:
  sonarqube:
    runs-on: ubuntu-latest
    env:
      BUILD_WRAPPER_OUT_DIR: build_wrapper_output_directory # Directory where build-wrapper output will be placed
    steps:
    - uses: actions/checkout@v4
      with:
        # Disabling shallow clone is recommended for improving relevancy of reporting
        fetch-depth: 0
    - name: Install sonar-scanner and build-wrapper
      uses: sonarsource/sonarqube-github-c-cpp@<action version> # Ex: v4.0.0, See the latest version at https://github.com/marketplace/actions/sonarqube-scan-for-c-and-c
    - name: Run build-wrapper
      run: |
        # here goes your compilation wrapped with build-wrapper; See https://docs.sonarsource.com/sonarqube-cloud/advanced-setup/languages/c-family/overview/#analysis-steps-using-build-wrapper for more information
        # build-preparation steps
        # build-wrapper-linux-x86-64 --out-dir ${{ env.BUILD_WRAPPER_OUT_DIR }} build-command
    - name: Run sonar-scanner
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: sonar-scanner --define sonar.cfamily.compile-commands="${{ env.BUILD_WRAPPER_OUT_DIR }}/compile_commands.json" #Consult https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/ for more information and options
```

If you are using SonarQube Server 10.5 or earlier, use `sonar.cfamily.build-wrapper-output` instead of `sonar.cfamily.compile-commands` in the `run` property of the last step, as Build Wrapper does not generate a compile_commands.json file before SonarQube Server 10.6, like this:
```yaml
run: sonar-scanner --define sonar.cfamily.compile-commands="${{ env.BUILD_WRAPPER_OUT_DIR }}/compile_commands.json"
```

See also [example configurations of C++ projects for SonarQube Server](https://github.com/search?q=org%3Asonarsource-cfamily-examples+gh-actions-sq&type=repositories).

## Action parameters

You can change the `build-wrapper` and `sonar-scanner` installation path by using the optional input `installation-path` like this:

```yaml
uses: sonarsource/sonarqube-github-c-cpp@<action version>
with:
  installation-path: my/custom/directory/path
```

Also, the absolute paths to the installed build-wrapper and sonar-scanner binaries are returned as outputs from the action.

Moreover, by default the action will cache sonar-scanner installation. However, you can disable caching by using the optional input: `cache-binaries` like this:
```yaml
uses: sonarsource/sonarqube-github-c-cpp@<action version>
with:
  cache-binaries: false
```

See also [example configurations](https://github.com/sonarsource-cfamily-examples?q=gh-actions-sq&type=all&language=&sort=)

### Environment variables

- `SONAR_TOKEN` – **Required** this is the token used to authenticate access to SonarQube. You can read more about security tokens in the [documentation](https://docs.sonarsource.com/sonarqube-server/latest/user-guide/managing-tokens/). You can set the `SONAR_TOKEN` environment variable in the "Secrets" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).
- *`GITHUB_TOKEN` – Provided by Github (see [Authenticating with the GITHUB_TOKEN](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token)).*
- `SONAR_HOST_URL` – this tells the scanner where SonarQube Server is hosted. You can set the `SONAR_HOST_URL` environment variable in the "Variables" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).
- `SONAR_ROOT_CERT` – Holds an additional certificate (in PEM format) that is used to validate the certificate of SonarQube Server or of a secured proxy to it. You can set the `SONAR_ROOT_CERT` environment variable in the "Secrets" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).

Here is an example of how you can pass a certificate (in PEM format) to the Scanner truststore:

```yaml
- uses: sonarsource/sonarqube-github-c-cpp@<action version>
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_ROOT_CERT: ${{ secrets.SONAR_ROOT_CERT }}
```

If your source code file names contain special characters that are not covered by the locale range of `en_US.UTF-8`, you can configure your desired locale like this:

```yaml
- uses: sonarsource/sonarqube-github-c-cpp@<action version>
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    LC_ALL: "ru_RU.UTF-8"
```

## Do not use this GitHub action if you are in the following situations

* You want to analyze code written in a language other than C or C++. Use the [SonarQube GitHub Action for SonarQube Server and Cloud](https://github.com/SonarSource/sonarqube-scan-action/) instead.
* You want to run the action on a 32-bits system - build wrappers support only 64-bits OS.

## Additional information

This action installs `coreutils` if run on macOS.

## Have question or feedback?

To provide feedback (requesting a feature or reporting a bug) please post on the [SonarSource Community Forum](https://community.sonarsource.com/tags/c/help/sq/github-actions).

## License

The action file and associated scripts and documentation in this project are released under the LGPLv3 License.
