# Red Hat AppStudio E2E Tests and Testing Framework

Testing framework and E2E tests are written in [Go](https://go.dev/) using [Ginkgo](https://onsi.github.io/ginkgo/) and [Gomega](https://onsi.github.io/gomega/) frameworks to cover Red Hat AppStudio.
It is recommended to install AppStudio in E2E mode, but the E2E suite can be also usable in [development and preview modes](https://github.com/redhat-appstudio/infra-deployments#preview-mode-for-your-clusters).

# Features

* Instrumented tests with Ginkgo 2.0 framework. You can find more information in [Ginkgo documentation](https://onsi.github.io/ginkgo/).
* Uses client-go to connect to OpenShift Cluster.
* Ability to run the E2E tests everywhere: locally([CRC/OpenShift local](https://developers.redhat.com/products/openshift-local/overview)), OpenShift Cluster, OSD...
* Writes tests results in JUnit XML/JSON file to a custom directory by using `--ginkgo.junit(or json)-report` flag.
* Ability to run the test suites separately.

# Running the tests
Whnen you want to run the E2E tests for AppStudio you need to have installed tools(in Requirements chapter), installed the AppStudio in E2E mode and compiled `e2e-appstudio` binary.

## Requirements
Requirements for installing AppStudio in E2E mode and running the E2E tests:

* An OpenShift 4.10 or higher Environment (If you are using CRC/OpenShift Local please also review [optional-codeready-containers-post-bootstrap-configuration](https://github.com/redhat-appstudio/infra-deployments#optional-codeready-containers-post-bootstrap-configuration))
* A machine from which to run the install (usually your laptop) with required tools:
  * A properly setup Go workspace using **Go 1.18 is required**
  * The OpenShift Command Line Tool (oc)
  * yq
  * jq
  * git
* Tokens
  * Github Token with the following permissions
    * `repo`
    * `delete_repo`
  * Valid quay token where to push AppStudio components images generated by the e2e framework

## Install AppStudio in E2E mode

Before executing the e2e suites you need to have deployed AppStudio in E2E Mode to your cluster.

1. Before deploying AppStudio in E2E mode you need to login to your OpenShift cluster with OpenShift Command Line Tool as `admin` (by default  `kubeadmin`):

   ```bash
    oc login -u <user> -p <password> --server=<oc_api_url>
   ```

2. Setup environment variables for GitHub and Quay.io tokens(tokens are required, you can also set more variables, see the table below):

   ```bash
    `export GITHUB_TOKEN=ghp_Iq...`
    `export QUAY_TOKEN=ewogI3...`
   ```

The following environments are used to launch the Red Hat AppStudio installation in E2E mode and the tests execution(tokens are also used for running the tests):

| Variable | Required | Explanation | Default Value |
|---|---|---|---|
| `GITHUB_TOKEN` | yes | A github token used to create AppStudio applications in github  | ''  |
| `QUAY_TOKEN` | yes | A quay token to push components images to quay.io. Note the quay token must be your dockerconfigjson encoded in base64 format, e.g. `ewogI3dJhdXRocyI6I...` | '' |
| `GITHUB_E2E_ORGANIZATION` | no | GitHub Organization where to create/push Red Hat AppStudio Applications  | `redhat-appstudio-qe`  |
| `QUAY_E2E_ORGANIZATION` | no | Quay organization where to push components containers | `redhat-appstudio-qe` |
| `E2E_APPLICATIONS_NAMESPACE` | no | Name of the namespace used for running HAS E2E tests | `appstudio-e2e-test` |
| `PRIVATE_DEVFILE_SAMPLE` | no | The name of the private git repository used in HAS E2E tests. Your GITHUB_TOKEN should be able to read from it. | `https://github.com/redhat-appstudio-qe/private-quarkus-devfile-sample` |
| `QUAY_OAUTH_USER` | no | A valid quay robot account username to make quay oauth | '' |
| `QUAY_OAUTH_TOKEN` | no | A valid quay quay robot account token to make oauth against quay.io. | '' |


3. Install dependencies:

``` bash
# Install dependencies
$ go mod tidy
# or go mod tidy -compat=1.17
# Copy the dependencies to vendor folder
$ go mod vendor
```

4. Install Red Hat AppStudio in e2e mode with install script in scripts folder. The e2e framework will use by default the `redhat-appstudio-qe` GitHub organization(you can change the GitHub organization by environment variable `GITHUB_E2E_ORGANIZATION`).

   ```bash
      make local/cluster/prepare
   ```
   or
   ```bash
       ./scripts/install-appstudio-e2e-mode.sh install
    ```

More information about how to deploy Red Hat AppStudio
are in the [infra-deployments](https://github.com/redhat-appstudio/infra-deployments) repository.

## Building and running the e2e tests
You can use scripts to build and run the tests:
   ```bash
      make local/test/e2e
   ```
Or build and run the tests without scripts:
1. Install dependencies and build the tests:

   ``` bash
   # Install dependencies
   $ go mod tidy
   # Copy the dependencies to vendor folder
   $ go mod vendor
   # Create `e2e-appstudio` binary in bin folder. Please add the binary to the path or just execute `./bin/e2e-appstudio`
   $ make build
   ```

2. Run the e2e tests:
The `e2e-appstudio` command is the root command that executes all test functionality. To obtain all available flags for the binary please use `--help` flags. All ginkgo flags and go tests are available in `e2e-appstudio` binary.

   ```bash
    `./bin/e2e-appstudio`
   ```

The instructions for every test suite can be found in the [tests folder](tests), e.g. [has Readme.md](tests/has/README.md).
You can also specify hich tests you want to run using [labels](docs/LabelsNaming.md) or [Ginkgo Focus](docs/DeveloperFocus.md).

# Red Hat AppStudio Load Tests

Load tests for AppStudio are also in this repository. More information about load tests are in [LoadTests.md](docs/LoadTests.md).

# Running Red Hat AppStudio Tests in OpenShift CI

Overview for OpenShift CI and AppStudio E2E tests is in [OpenshiftCI.md](docs/OpenShiftCI.md). How to install E2E binary is in [Installation.md](docs/Installation.md).

# Develop new tests

* Create test folder under tests folder: `tests/[<application-name>]...`, e.g. [has](tests/has).
  * `tests/has` - all tests used owned by AppStudio application service team
* Every test package should be imported to `cmd/e2e_test.go`, e.g. [has](https://github.com/redhat-appstudio/e2e-tests/blob/main/cmd/e2e_test.go#L15).
* Every new test should have correct [labels](docs/LabelsNaming.md).
* Every test should have meaningful description with JIRA/GitHub issue key.
* (Recommended) Use JIRA integration for linking issues and commits (just add JIRA issue key in the commit message). You can found more information about GitHub-JIRA integration [here](https://docs.engineering.redhat.com/display/JiraAid/GitHub-Jira+integration).

```golang
// cmd/e2e_test.go
package common

import (
	// ensure these packages are scanned by ginkgo for e2e tests
	_ "github.com/redhat-appstudio/e2e-tests/tests/common"
	_ "github.com/redhat-appstudio/e2e-tests/tests/has"
)
```

# Reporting issues
For reporting issues with e2e tests please use [RHDP JIRA project](https://issues.redhat.com/projects/RHDP) - please use labels `appstudio` and `quality`. If the issues is also relevant to CI - for example it is blocking PRs, the also use label `appstudio-e2e-tests-known-issues`.

# Debugging tests
## In vscode
There is launch configuration in `.vscode/launch.json` called `Launch demo suites`.
Running this configuration, you'll be asked for github token and then e2e-demos suite will run with default configuration.
If you want to run/debug different suite, change `-ginkgo.focus` parameter in `.vscode/launch.json`.

