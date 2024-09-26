# Flexi actions runner demo

This repository is an example of how you could run a 
[self-hosted github actions runner](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
on NeSI's Flexi-HPC cloud platform.
You may want to do this if your build is very resource intensive (e.g. it takes
very long; needs a lot of memory, compute power or disk space; etc.) and won't fit
within a normal GitHub-hosted actions runner. Another reason could be if your CI
pipeline requires special resources, such as GPUs, to run.

The [terraform](terraform) directory contains the code required to deploy the actions
runner to flexi. Code in that directory is used in the CI workflow defined in the
[.github/workflows/build.yml](.github/workflows/build.yml) file. The workflow has three
main steps:

- Step 1: use terraform to provision a VM on Flexi and run a GitHub actions self-hosted runner on it (this steps runs on a normal GitHub-hosted runner)
- Step 2: run the user-defined build step on the self-hosted runner created in Step 1 and upload the output as an artifact on the GitHub build (if we're building a tag, also create a release and upload the output to the release too)
- Step 3: destroy the resources that were provisioned on Flexi in Step 1 (this step runs on a normal GitHub-hosted runner)

The [src](src) directory contains a placeholder script that gets run during step 2 above.
This could be replaced with something real.

## Using the actions runner in your repo

You could copy the *terraform* directory and *build.yml* script into your own repository and
edit step 2 in *build.yml* to build/run your own code. Some configuration is required as
described in [terraform/README.md](terraform/README.md). You may also want to edit other
[terraform variables](terraform/variables.tf), such as `runner_volume_size`. 

## Connecting to the actions runner VM

By default it is not possible to connect to the actions runner VM. No floating ip is attached and
the security group does not allow inbound connections. There is a [terraform variable](terraform/variables.tf)
called `enable_debugging` that will add the floating ip and allow incoming SSH connections if set to true,
which can be useful for debugging.

## LUMASS example

A real example of how this pipeline could be used can be found in the *build-flexi* branch
of this [LUMASS fork](https://github.com/chrisdjscott/LUMASS/tree/build-flexi). In that repo we have:

- the [utils/build](https://github.com/chrisdjscott/LUMASS/tree/build-flexi/utils/build) directory containing:
  - the *terraform* directory (used in steps 1 and 3 above)
  - LUMASS build scripts that will be used in the CI pipeline (step 2 above)
- the *.github/workflows/build.yml* file defining the CI pipeline with the three steps as described above
  - step 2 has been modified to run the LUMASS build scripts mentioned above

An example of the CI pipeline can be found [here](https://github.com/chrisdjscott/LUMASS/actions/runs/10929797640):
- the first step provisions the self-hosted runner on flexi
- the second step builds lumass and uploads the *.AppImage* as an artifact of the build (for tags the workflow would create a release and upload the *.AppImage* file to that)
- the third step destroys the resources that were created in the first step

## Security

Self-hosted runners could be a security risk on public repositories if you allow pull requests to run the pipeline automatically
as mentioned in the [GitHub docs](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#self-hosted-runner-security).
In this case, as with the GitHub hosted runners, we are also using a clean isolated virtual machine that is destroyed at the end of the job execution.
For extra security you could set up approval for workflow runs from public forks as described
[here](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/approving-workflow-runs-from-public-forks).
