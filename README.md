# CI Unit

Generate JUnit XML for CI job steps to allow for analysis using existing testing oriented tools.

## Approach

Tools exist to detect flaky tests and handle resolution workflows, but CI often contains non-testing steps that are also prone to being flaky. Rather than build a tool specific to a CI vendor CI Unit provides a wrapper shell that generates JUnit XML files to provide _dummy_ test results for each step in a CI workflow. These XML files may then be processed by existing tools just like any tests.

The initial implementation focused on providing basic functionality without adding dependencies in order to facilitate usage in a variety of language environments.

## Docker

Docker images can be built using the provided `Dockerfile` by specifying the base image as a build argument.

```
docker build --build-arg BASE_IMAGE=something/cool .
```

## CircleCI

The key ingredient is changing the `shell` for a job to be the `ci-unit-shell` and upload the XUniT XML files. For use in multiple jobs defining an executor is likely the best route.

```yaml
executors:
  ci-unit-shell:
    docker:
      - image: image-with-ci-unit
    shell: /bin/ci-unit-shell

jobs:
  build-and-stuff:
    executor: ci-unit-shell
    steps:
      - run:
          name: do stuff
          command: ./stuff.sh
      - store_test_results:
          path: /tmp/ci-unit
```

If you want to upload the JUnit files elsewhere (if your tool does not use CircleCI API) then you may want to create a command to do both.

```yaml
commands:
  test-result-upload:
    description: upload test results to CircleCI and third-party
    parameters:
      path:
        type: string
        default: /tmp/ci-unit
    steps:
      - store_test_results:
          path: << parameters.path >>
      - other-service-upload:
          path: << parameters.path >>
```

Which could then be used in job definition in place of `store_test_results`.
