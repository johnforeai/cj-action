# Critical Journey Github Action

This GitHub Action runs the critical journey script inside a Docker container.

## Inputs

- `test_id`: ID of the test to be run. Either this or `test_suite_id` should be provided.
- `test_suite_id`: ID of the test suite to be run.
- `service_account_key`: Your service account key to access fore ai Critical Journey.
- `wait_timeout_seconds`: (Optional) Maximum number of seconds to wait for the test to complete. Default is 300 seconds. Must be between 30 and 900 seconds (inclusive).
- `website_url_override`: (Optional) Allows overriding the base website URL used during test execution.  
- `params_override`: (Optional) Allows overriding default parameter values defined in the test suite, so that tests can be run with custom parameter values. This should be a valid json string and all keys and values are also strings.
- `browser_type_override`: (Optional) Browser engine to run the test with: 'chromium', 'firefox', or 'webkit'. Defaults to 'chromium' if not specified.
- `create_issue_on_failure`: (Optional) If `true`, automatically creates a GitHub issue when the test run fails. The issue includes step traces, error details, test configuration, and a screenshot from the last executed step. Requires `GITHUB_TOKEN` to be available. Default is `false`.

## Outputs

- `result`: A message that includes information about the status of the run.

## Example Usage for running a single test

```yaml
name: Run CJ Github Action

on: [push]

jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Test
        uses: foreai-co/cj-action@v1
        id: run_cj
        with:
          test_id: 'my-test-id'
          service_account_key: ${{ secrets.CRITICAL_JOURNEY_SERVICE_ACCOUNT_KEY }}
      
      - name: Print result
        run: echo "${{ steps.run_cj.outputs.result }}"
```

## Example Usage for running a test suite

```yaml
name: Run CJ Github Action

on: [push]

jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Test Suite
        uses: foreai-co/cj-action@v1
        id: run_cj
        with:
          test_suite_id: 'my-test-suite-id'
          service_account_key: ${{ secrets.CRITICAL_JOURNEY_SERVICE_ACCOUNT_KEY }}
          wait_timeout_seconds: 360  # = 6 minutes
          # Override the website url using this optional field.
          website_url_override: 'https://beta-dev.my-awesome.com/2'
          # Override test parameters using this optional field. Provide valid json string.
          params_override: '{ "param1" : "value1", "param2" : "value2" }'

      - name: Print result
        run: echo "${{ steps.run_cj.outputs.result }}"
```

## Example Usage with automatic GitHub issue creation on failure

When `create_issue_on_failure` is enabled, the action uses the default `GITHUB_TOKEN` provided automatically by GitHub Actions — no manual secret configuration required. If your repository uses restrictive default permissions, add `issues: write` to the job permissions.

```yaml
name: Run CJ Github Action

on: [push]

jobs:
  my-job:
    runs-on: ubuntu-latest
    permissions:
      issues: write  # Required to create GitHub issues on failure
    steps:
      - name: Run Test Suite
        uses: foreai-co/cj-action@v1
        id: run_cj
        with:
          test_suite_id: 'my-test-suite-id'
          service_account_key: ${{ secrets.CRITICAL_JOURNEY_SERVICE_ACCOUNT_KEY }}
          create_issue_on_failure: 'true'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Print result
        run: echo "${{ steps.run_cj.outputs.result }}"
```

## Jenkins Usage

### Prerequisites

- Docker must be installed on the Jenkins agent
- The `CJ_SERVICE_ACCOUNT_KEY` and `CJ_TEST_ID` credentials must be added to Jenkins as **Secret text** credentials (Manage Jenkins → Credentials → Global → Add Credentials)

> **Note:** Ensure there are no leading or trailing spaces in the credential values when saving them in Jenkins.

### Example Usage for running a single test

```groovy
pipeline {
    agent any
    stages {
        stage('Run Critical Journey Test') {
            steps {
                withCredentials([
                    string(credentialsId: 'CJ_SERVICE_ACCOUNT_KEY', variable: 'SERVICE_KEY'),
                    string(credentialsId: 'CJ_TEST_ID', variable: 'TEST_ID')
                ]) {
                    sh 'docker run --rm -e "INPUT_SERVICE_ACCOUNT_KEY=$SERVICE_KEY" -e "INPUT_TEST_ID=$TEST_ID" foreai/cj-action:latest'
                }
            }
        }
    }
}
```

### Example Usage for running a test suite

```groovy
pipeline {
    agent any
    stages {
        stage('Run Critical Journey Test Suite') {
            steps {
                withCredentials([
                    string(credentialsId: 'CJ_SERVICE_ACCOUNT_KEY', variable: 'SERVICE_KEY'),
                    string(credentialsId: 'CJ_TEST_SUITE_ID', variable: 'SUITE_ID')
                ]) {
                    sh 'docker run --rm -e "INPUT_SERVICE_ACCOUNT_KEY=$SERVICE_KEY" -e "INPUT_TEST_SUITE_ID=$SUITE_ID" -e "INPUT_WAIT_TIMEOUT_SECONDS=360" -e "INPUT_WEBSITE_URL_OVERRIDE=https://staging.my-app.com" -e "INPUT_PARAMS_OVERRIDE={\"param1\":\"value1\",\"param2\":\"value2\"}" foreai/cj-action:latest' # Override test parameters using this optional field. Provide valid json string.
                }
            }
        }
    }
}
```

### All available inputs

Pass any of the following as `-e "INPUT_<NAME>=<value>"` in the `docker run` command:

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `INPUT_SERVICE_ACCOUNT_KEY` | Yes | | Your service account key to access fore.ai Critical Journey. |
| `INPUT_TEST_ID` | No* | | ID of the test to be run. Either this or `INPUT_TEST_SUITE_ID` must be provided. |
| `INPUT_TEST_SUITE_ID` | No* | | ID of the test suite to be run. |
| `INPUT_WAIT_TIMEOUT_SECONDS` | No | `300` | Maximum seconds to wait for the test to complete. Must be between 30 and 900. |
| `INPUT_WEBSITE_URL_OVERRIDE` | No | | Overrides the base website URL used during test execution. |
| `INPUT_PARAMS_OVERRIDE` | No | | Overrides default parameter values. Must be a valid JSON string. |
| `INPUT_BROWSER_TYPE_OVERRIDE` | No | `chromium` | Browser engine: `chromium`, `firefox`, or `webkit`. |
