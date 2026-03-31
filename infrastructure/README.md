# Integration Testing Infrastruture

This directory contains the files that provide the OpenCHAMI
integration testing infrastructure. The contents break down into data
and scripts. Data files are:

- Dockerfile
- config.yaml
- config-test_vtds.yaml

Scripts are:

- start_test.sh
- run_test_deployment.sh

To understand how all these pieces work together it is helpful to
understand how test deployments are run from this repo. Here is the
basic procedure. First of all, the players are:

- a GitHub action run from this repository
- a test platform running as a compute instance in a project under
  Google Cloud Platform
- a Podman container that is created as needed on the test platform

With that in mind, here is the basic workflow from the point of
triggering the GitHub action to the point of test completion:

1) The GitHub action uses SSH to make a new test directory from which
   all operations will run on the test platform.
1) The GitHub action then uses SCP to copy the `start_test.sh` script
   from here to the test directory.
1) The GitHub action finally uses SSH to log into the test platform and
   run the `start_test.sh` script providing the OpenCHAMI Release version
   under test in the environment as `OPENCHAMI_VERSION`.
1) The `start_test.sh` script clones this repository under the test
   directory, then makes a new container image using the `Dockerfile`
   found here and starts a container in daemon mode to run the test
   deployment.
1) Once the container is started, the `start_test.sh` script removes its
   copy of this repo from the test directory and exits.
1) The GitHub action then uses SSH to remove the test directory on the
   test platform as it is no longer needed.
1) The entrypoint to the container is the `run_test_deployment.sh`
   script, which clones this repository inside the container and does
   the following:
   1) create a new `test-run/<version>-<timestamp>` branch within the
      repository.
   1) create a new `test_results/<version>-<timestamp>` deployment
      directory within the branch.
   1) create a file named `status` in the deployment directory and
      begin recording state changes in that file.
   1) compose a vTDS core configuration file in the deployment directory
      using the `config.yaml` found here as a template.
      
      NOTE: the `config-test_vtds.yaml` found here is used for testing
            the vTDS test infrastructure, and is only used by
            developers.  To use it instead of `config.yaml` set the
            `VTDS_CONFIG_TEMPLATE` environment variable to
            `config-test_vtds.yaml` before initiating the test run.

   1) run the `vtds deploy` command in the background from the
      deployment directory. This creates a new OpenCHAMI system and runs
      the vTDS provided OpenCHAMI tests in it.
   1) monitor the progress of the `vtds deploy` both by capturing its
      output in a file in the deployment directory and by updating the
      `status` file.
   1) periodically commit and push the changes to status and the
      collected output back to the branch on GitHub.
   1) assuming a successful deployment, collect the test results and
      commit / push them to the branch on GitHub
   1) run the `vtds remove` command in the background to remove the
      OpenCHAMI test system
   1) monitor the progress of the `vtds remove` both by capturing its
      output in a file in the deployment directory and by updating the
      `status` file.
   1) periodically commit and push the changes to status and the
      collected output back to the branch on GitHub.
