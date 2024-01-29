# ReversingLabs GitLab CI configuration: rl-scanner-cloud-gitlab-include

ReversingLabs provides the officially supported GitLab CI configuration
for faster and easier deployment of the `rl-scanner-cloud` solution in CI/CD workflows.

The `rl-scanner-cloud-gitlab-include` repository provides the remote include configuration file called
`rl-scanner-cloud-gitlab-include.yml`.

To add the configuration to existing pipelines, use the following syntax:


    include:
      - remote: <raw url of the yml file>


The configuration uses the official
[reversinglabs/rl-scanner-cloud](https://hub.docker.com/r/reversinglabs/rl-scanner-cloud) Docker image
to scan a single build artifact with `secure.software Portal`
and display the analysis status as one of the checks in the GitLab interface.

This configuration is most suitable for experienced users who want to include it into more complex workflows.


## What is the secure.software Portal?

The secure.software Portal is a SaaS solution that's part of the secure.software platform - a new ReversingLabs solution for software supply chain security.
More specifically, the Portal is a web-based application for improving and managing the security of your software releases and verifying third-party software used in your organization.

With the secure.software Portal, you can:

- Scan your software packages to detect potential risks before release.
- Improve your SDLC by applying actionable advice from security scan reports to all phases of software development.
- Organize your software projects and automatically compare package versions to detect potentially dangerous behavior changes in the code.
- Manage software quality policies on the fly to ensure compliance and achieve maturity in your software releases.


## How the include configuration works

The include configuration is a remote file that needs to be included into the main GitLab pipeline and that acts as its test stage.

This configuration expects that the build artifact is produced before the test stage
and uploaded to the job for it to be shared between jobs.
It requires specifying the path and filename of the artifact as the input to the pipeline.
The path must be relative to the root of the GitLab repository.

Users also need to specify the report directory name,
as the pipeline saves all supported analysis report formats for the artifact into that directory.
The directory path must be relative to the root of the GitLab repository.
The CycloneDX report is uploaded separately to the same user-provided report directory and can be used for
[compliance purposes](https://docs.gitlab.com/ee/user/compliance/license_scanning_of_cyclonedx_files/).

This pipeline has specific checks for
[environment variables](#environment-variables)
and
[input parameters](#inputs)
that throw an error if any values are missing.

The scan job runs a set of commands that pull the latest version of the
`reversinglabs/rl-scanner-cloud` Docker image
and scan the provided artifact inside the container.
A specific entrypoint has to be used in order to make the image compatible with GitLab runner.
When the security scan is done,
the container automatically shuts down,
and the action outputs the analysis result as a status message (PASS, FAIL).


## Prerequisites

To successfully use this Docker image, you need:

* **An active secure.software Portal account and a Personal Access Token generated for it**.
If you don't already have a Portal account, you may need to contact the administrator of your Portal organization to
[invite you](https://docs.secure.software/portal/members#invite-a-new-member).
Alternatively, if you're not a secure.software customer yet, you can
[contact ReversingLabs](https://docs.secure.software/portal/#get-access-to-securesoftware-portal)
to sign up for a Portal account.
When you have an account set up,
follow the instructions to
[generate a Personal Access Token](https://docs.secure.software/api/generate-api-token).


**Note for GitLab Enterprise users:** The GitLab CI/CD must be appropriately configured for the repository where you want to use the pipeline.
If you don't have the
[appropriate role](https://docs.gitlab.com/ee/ci/quick_start/index.html),
contact your GitLab organization administrators for help.


## Environment variables

This configuration requires the `secure.software Portal` license data to be passed using a environment variable.

| Environment variable    | Required | Description |
| ---------               | ------   | -------    |
| `RLPORTAL_ACCESS_TOKEN` | **Yes**  | The `rl-secure-portal` Personal Access Token. Define it as a Secret. |


Secrets have to be defined via your GitLab repository `Settings → CI/CD → Variables`.

ReversingLabs strongly recommends following best security practices and [defining these secrets](https://docs.gitlab.com/ee/ci/variables/) on the level of your GitLab organization or repository.


## How to use this include configuration

The most common use-case for this configuration is
to run it as the "test" stage of the pipeline,
after the build artifact has been created.


### Inputs

| Input parameter           | Required | Description |
| ---------                 | -----    | ------      |
| `RLPORTAL_SERVER`      | **Yes**  | Name of the secure.software Portal instance to use for the scan. The Portal instance name usually matches the subdirectory of `my.secure.software` in your Portal URL. For example, if your portal URL is `my.secure.software/demo`, the instance name to use with this parameter is `demo`. |
| `RLPORTAL_ORG`         | **Yes** | The name of a secure.software Portal organization to use for the scan. The organization must exist on the Portal instance specified with `RLPORTAL_SERVER`. The user account authenticated with the token must be a member of the specified organization and have the appropriate permissions to upload and scan a file. Organization names are case-sensitive. |
| `RLPORTAL_GROUP`       | **Yes** | The name of a secure.software Portal group to use for the scan. The group must exist in the Portal organization specified with `RLPORTAL_ORG`. Group names are case-sensitive. |
| `RL_PACKAGE_URL`          | **Yes**  | If using a package store, use this parameter to specify the package URL (PURL) for the scanned artifact. The package URL should be in the format `project/package@version`; for example `testing/demo-rl-scanner@v1.0.3`. |
| `MY_ARTIFACT_TO_SCAN`     | **Yes**  | The file name of the artifact to be scanned. The artifact must exist before the test stage is run. |
| `PACKAGE_PATH`            | **Yes**  | The location relative to the checkout of the artifact to be scanned. The artifact is expected to be found at: `${PACKAGE_PATH}/${MY_ARTIFACT_TO_SCAN}`. |
| `REPORT_PATH`             | **Yes**  | The location relative to the checkout where analysis reports will be stored after the scan is finished. The directory specified as `REPORT_PATH` must be empty. |
| `RL_DIFF_WITH`            | No       | Use this parameter to specify a previously scanned package version to compare (diff) against. |
| `RL_VERBOSE`              | No       | Set to anything but '' to provide more feedback in the output while running the scan. Disabled by default. |
| `RLSECURE_PROXY_SERVER`   | No       | Server name for proxy configuration (IP address or DNS name). |
| `RLSECURE_PROXY_PORT`     | No       | Network port on the proxy server for proxy configuration. Required if `RLSECURE_PROXY_SERVER` is used. |
| `RLSECURE_PROXY_USER`     | No       | User name for proxy authentication. |
| `RLSECURE_PROXY_PASSWORD` | No       | Password for proxy authentication. Required if `RLSECURE_PROXY_USER` is used. |


### Outputs

| Output parameter | Description |
| :--------- | :------ |
| `DESCRIPTION` | The result of the action - a string terminating in FAIL or PASS. |
| `STATUS` | The single-word status representing the result of the action. It can be any of the following: success, failed, error. <br>**Success** indicates that the resulting string contains PASS. <br> **Failed** indicates the resulting string contains FAIL. <br>**Error** indicates that something went wrong during the scan and the action was not able to retrieve the resulting string. |


### Artifacts

The job creates the reports in the directory: `$REPORT_PATH`.
The reports are always created even if the scan job fails.

Users can control the `REPORT_PATH` as an input parameter.
The specified directory must be empty.


## Examples

The following example is a basic pipeline that includes the remote configuration as the test stage.

The workflow scans a build artifact with `rl-scanner-cloud`,
saves the report,
and displays the analysis results.
The remote configuration can be included anywhere in the configuration file,
but it will run only after the build stage.


    # REQUIREMENTS:
    #   RLPORTAL_ACCESS_TOKEN: must be declared as a global variable type 'variable'

    variables:
      RL_VERBOSE: 1
      RLPORTAL_SERVER: test
      RLPORTAL_ORG: Test
      RLPORTAL_GROUP: Default
      RL_PACKAGE_URL: myproject/mypackage@v1
      MY_ARTIFACT_TO_SCAN: bash
      PACKAGE_PATH: ./package
      REPORT_PATH: RlReport

    include:
      - remote: 'https://raw.githubusercontent.com/reversinglabs/rl-scanner-cloud-gitlab-include/main/rl-scanner-cloud-gitlab-include.yml'

    # simulate a build stage
    job-build:
      stage: build

      image:
        # Any image you require
        name: ubuntu:latest

      artifacts:
        name: "build_artifact"
        paths:
          - $PACKAGE_PATH/*

      script:
        - |
          export HOME=$( pwd ); echo $HOME
          # Prepare to build an artifact as the build output
          mkdir $PACKAGE_PATH
          cp /bin/${MY_ARTIFACT_TO_SCAN} $PACKAGE_PATH/

    # the include contains the stage: test and runs after build and before deploy

    # simulate a deploy stage
    job-deploy:
      stage: deploy

      image:
        # Any image you require
        name: ubuntu:latest

      script:
        - |
          # Here we have access to all artifacts from the previous jobs
          # $PACKAGE_PATH and $REPORT_PATH should be visible
          export HOME=$( pwd ); echo $HOME
          ls -la .
          ls -la $PACKAGE_PATH
          ls -la $REPORT_PATH
          echo "This job deploys the artifact from the $CI_COMMIT_BRANCH branch."


## Useful resources

- The official `reversinglabs/rl-scanner-cloud` Docker image [on Docker Hub](https://hub.docker.com/r/reversinglabs/rl-scanner-cloud)
- [Use CI/CD configuration from other files](https://docs.gitlab.com/ee/ci/yaml/includes.html)
