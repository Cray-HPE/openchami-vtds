# OpenCHAMI vTDS Integration Testing

This repository both drives integration testing of OpenCHAMI using GitHub actions and vTDS, and contains the infrastructure used to do that. It also monitors and manages, through github actions, the resources used for integration testing.

The repository is split into three general areas:

- Test results, in the `test_results` directory tree
- Infrastructure (scripts and other files needed for testing) in the `infrastructure` directory tree.
- Github actions in the `.github` directory tree

The `test_results` directory is automatically populated and should not need to be modified by developers. The `infrastructure` and `.github` directories contain developer modifiable code and data.

