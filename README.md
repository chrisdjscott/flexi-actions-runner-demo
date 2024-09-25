# Flexi actions runner demo

This repository is an example of how you could run a self-hosted github
actions runner on NeSI's Flexi-HPC cloud platform.
You may want to do this if your build is very resource instensive (e.g. it takes
very long, needs a lot of memory of compute power or disk space) and won't fit
within a normal GitHub-hosted actions runner. Another reason could be if your CI
pipeline requires special resources, such as GPUs, to run.

The [terraform](terraform) directory contains the code required to deploy the actions
runner to flexi. Code in that directory is used in the CI workflow defined in the
[.github/workflows/build.yml](.github/workflows/build.yml) file. The workflow has three
main steps:

- Step 1: use terraform to provision a VM on Flexi and run a GitHub actions self-hosted runner on it (this steps runs on a normal GitHub-hosted runner)
- Step 2: run the user-defined build step on the self-hosted runner created in Step 1 and upload the output as an artifact on the GitHub build (if we're building a tag, also create a release and upload the output to the release too)
- Step 3: destroy the resources that were provisioned on Flexi in Step 1 (this step runs on a normal GitHub-hosted runner)

The [src](src) directory contains placeholder script that gets run during step 2 above.
This could be replaced with something real.

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
