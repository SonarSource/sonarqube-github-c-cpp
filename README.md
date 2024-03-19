# Scan your code with SonarQube [![Tests](https://github.com/SonarSource/sonarqube-github-c-cpp/actions/workflows/tests.yml/badge.svg)](https://github.com/SonarSource/sonarqube-github-c-cpp/actions/workflows/tests.yml)

Using this GitHub Action, achieve [Clean Code](https://www.sonarsource.com/solutions/clean-code/?utm_medium=referral&utm_source=github&utm_campaign=clean-code&utm_content=sonarqube-scan-action) 
with [SonarQube](https://www.sonarqube.org/) by scanning to detect Bugs, Vulnerabilities, and Code Smells in C, C++ and Objective-C!

<img alt="The SonarQube logo" src="./images/SonarQube-72px.png">

SonarQube is the leading product for Continuous Code Quality and code Security. 
It supports most popular programming languages, including Java, JavaScript, TypeScript, C#, Python, C, C++, and many more.

## Requirements

To run an analysis on your code, you first need to set up your project on SonarQube. 
Your SonarQube instance must be accessible from GitHub, and you will need a Project analysis token or a Global analysis token to run the analysis (more information below under **Environment variables**).

Read more information on how to analyze your code [here](https://docs.sonarqube.org/latest/analysis/github-integration/).


## Usage


Project metadata, including the location to the sources to be analyzed, must be declared in the file `sonar-project.properties` in the base directory:

```properties
sonar.projectKey=<replace with the key generated when setting up the project on SonarQube>

# relative paths to source directories. More details and properties are described
# in https://docs.sonarsource.com/sonarqube/latest/project-administration/analysis-scope/
sonar.sources=.
```

The workflow, usually declared in `.github/workflows/build.yml`, looks like:

```yaml
on:
  # Trigger analysis when pushing in master or pull requests, and when creating
  # a pull request.
  push:
    branches:
      - master
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
    - name:  Install sonar-scanner and build-wrapper
      uses: sonarsource/sonarqube-github-c-cpp@v1
      env:
        SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        SONAR_ROOT_CERT: ${{ secrets.SONAR_ROOT_CERT }}
    - name: Run build-wrapper
      run: |
      #here goes your compilation wrapped with build-wrapper; See https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/languages/c-family/#using-build-wrapper for more information
      # build-preparation steps
      # build-wrapper-linux-x86-64 --out-dir ${{ env.BUILD_WRAPPER_OUT_DIR }} build-command
    - name: Run sonar-scanner
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      run: sonar-scanner --define sonar.cfamily.build-wrapper-output="${{ env.BUILD_WRAPPER_OUT_DIR }}" #Consult https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner/ for more information and options
```

You can change the `build-wrapper` and `sonar-scanner` installation path by using the optional input `installation-path` like this:

```yaml
uses: sonarsource/sonarqube-github-c-cpp@v1
with:
  installation-path: my/custom/directory/path
```
Also, the absolute paths to the installed build-wrapper and sonar-scanner binaries are returned as outputs from the action.

Moreover, by default the action will cache sonar-scanner installation. However, you can disable caching by using the optional input: `cache-binaries` like this:
```yaml
uses: sonarsource/sonarqube-github-c-cpp@v1
with:
  cache-binaries: false
```

If your SonarQube server uses a self-signed certificate, you can pass a root certificate (in PEM format) to the java certificate store:

```yaml
uses: sonarsource/sonarqube-github-c-cpp@v1
env:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
  SONAR_ROOT_CERT: ${{ secrets.SONAR_ROOT_CERT }}
```

See also [example configurations](https://github.com/sonarsource-cfamily-examples?q=gh-actions-sc&type=all&language=&sort=)

### Secrets and environment variables

Following secrets are required for successful invocation of sonar-scanner: 
- `SONAR_TOKEN` – **Required** this is the token used to authenticate access to SonarQube. You can read more about security tokens [here](https://docs.sonarqube.org/latest/user-guide/user-token/). You can set the `SONAR_TOKEN` environment variable in the "Secrets" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).
- *`GITHUB_TOKEN` – Provided by Github (see [Authenticating with the GITHUB_TOKEN](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/authenticating-with-the-github_token)).*

Environment variables:
- `SONAR_HOST_URL` – **Required** this tells the scanner where SonarQube is hosted. You can set the `SONAR_HOST_URL` environment variable in the "Secrets" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).
- `SONAR_ROOT_CERT` – Holds an additional root certificate (in PEM format) that is used to validate the SonarQube server certificate. You can set the `SONAR_ROOT_CERT` environment variable in the "Secrets" settings page of your repository, or you can add them at the level of your GitHub organization (recommended).

## Do not use this GitHub action if you are in the following situations

* You want to analyze code written in a language other than C, C++ or Objective-C?. Use the [SonarQube Scan GitHub Action](https://github.com/SonarSource/sonarqube-scan-action) instead
* You want to run the action on a 32-bits system - build wrappers support only 64-bits OS

## Additional information

This action installs `coreutils` if run on macOS

## Have question or feedback?

To provide feedback (requesting a feature or reporting a bug) please post on the [SonarSource Community Forum](https://community.sonarsource.com/) with the tag `sonarqube`.

## License

The action file and associated scripts and documentation in this project are released under the LGPLv3 License.
