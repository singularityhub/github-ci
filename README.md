# Singularity Builder GitHub CI

![img/sregistry-github-small.png](img/sregistry-github-small.png)

This is a simple example of how you can achieve:

 - version control of your recipes
 - build of associated container and
 - push to a storage endpoint

for a reproducible build workflow! By default, we will build on all pull requests and deploy
on push to main. The containers will go to an enabled GitHub package registry thanks to
the Singularity oras endpoint.

**updated** August 2021, we can now push containers to the GitHub package registry! Woohoo!

There are three workflows configured as examples to build and deploy Singularity containers:

1. [native install](.github/workflows/native-install.yml) discovers Singularity* changed files, and builds Singularity 3.x (with GoLang) natively, deploys to GitHub packages.
2. [docker image](.github/workflows/container.yml) discovers Singularity* changed files, and builds in a [docker image](https://quay.io/repository/singularity/singularity), also deploys to GitHub packages.
3. [manual deploy](.github/workflows/manual-deploy.yml) takes a list of manually specified Singularity recipes (that aren't required to be changed), builds Singularity 3.x natively, and deploys to GitHub packages.

While the "build in a container" option is faster to complete and a more simple workflow, it should be noted that docker runs with
`--privileged` which may lead to issues with the resulting container in a non privileged situation. Also note that you
are free to mix and match the above recipes to your liking, or [open an issue](https://github.com/singularityhub/github-ci/issues) if you want to ask for help!

**Why should this be managed via Github?**

Github, by way of easy integration with **native** continuous integration, is an easy way
to have a workflow set up where multiple people can collaborate on a container recipe,
the recipe can be tested (with whatever testing you need), discussed in pull requests,
and tested on merge to master. Further, now with GitHub packages we can push our containers
directly to the GitHub package registry!

**Why should I use this instead of a service?**

You could use a remote builder, but if you do the build in a continuous integration
service you get complete control over it. This means everything from the version of
Singularity to use, to the tests that you run for your container. You have a lot more
freedom in the rate of building, and organization of your repository, because it's you
that writes the configuration.

## Quick Start

### 1. Enable Packages

If you want to use the [GitHub package registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
you'll need to follow the instructions there to enable packages for your organization, specifically "public" and "internal" packages should be allowed to be created. You'll also want to add a username associated with your GitHub organization to the repository secret `GHCR_USERNAME`. If a package already exists for your repo, you may need to [follow instructions](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-ghcrio) to updade a package from using personal access tokens to the safer `GITHUB_TOKEN`.

### 2. Add Your Recipes

Add your Singularity recipes to this repository, which should be named `Singularity.<tag>` 
or just `Singularity` to follow the previously published convention. You can then choose your file in [.github/workflows](.github/workflows).
If you choose the `manual-deploy.yml` you can manually specify recipes in the matrix variable "recipe."
If you choose either of the other two workflows, changed files that start with Singularity.* will
be automatically detected and built.

### 3. Test your Container

Importantly, we suggest that you add some steps to test your container! Whether that's running it,
exec'ing a custom command, or invoking the test command, there is more than
one way to eat a reeses:

```yaml
    - name: Test Container
      run: |
        singularity exec smokey.sif python run_tests.py
        singularity test smokey.sif
        singularity run toasty.sif
```

This step is not provided in the workflows, but it's recommended that you think about it and add if necessary.

### 4. Check Triggers

The workflow files each have a section at the top that indicates when the workflow will
trigger. By default, we will do builds on pull requests, and deploys on pushes to a main
branch. If you want to change this logic, edit the top of the recipe files.

### 5. Push to a registry

If you are good with GitHub packages, then you are good to go! Otherwise,
if you want to push to other kinds of storage, you can install the [Singularity Registry Client](http://singularityhub.github.io/sregistry-cli) and push to your cloud storage of choice! You will want to add python and python-dev to the dependency
install:

```yaml
    - name: Install Dependencies
      run: |
        sudo apt-get update && sudo apt-get install -y \
          build-essential \
          libssl-dev \
          uuid-dev \
          libgpgme11-dev \
          squashfs-tools \
          libseccomp-dev \
          pkg-config \
          python-dev python python3-pip
```

And then install and use sregistry client. Here are many examples:

```yaml
    - name: Deploy Container
      run: |
        sudo pip3 install sregistry
        SREGISTRY_CLIENT=google-storage sregistry push --name username/reponame smokey.sif
        SREGISTRY_CLIENT=s3 sregistry push --name username/reponame smokey.sif
        SREGISTRY_CLIENT=registry sregistry push --name username/reponame smokey.sif
        SREGISTRY_CLIENT=dropbox sregistry push --name username/reponame smokey.sif
```

See the [clients page](https://singularityhub.github.io/sregistry-cli/clients) for all the options.
If you want to change the recipe triggers, see [here](https://help.github.com/en/articles/about-github-actions#core-concepts-for-github-actions)
for getting started with GitHub actions, and [please open an issue](https://www.github.com/singularityhub/github-ci/issues)
if you need any help.

### 6. Pull Your Container!

The example container here is published to [singularithub/github-ci](https://github.com/singularityhub/github-ci/pkgs/container/github-ci)
and can be pulled as follows:

```bash
$ singularity pull oras://ghcr.io/singularityhub/github-ci:latest
INFO:    Downloading oras image
$ ls
github-ci_latest.sif  img  README.md  Singularity

$ ./github-ci_latest.sif 
Hold me closer... tiny container :) :D
```

## Other Options

You can customize this base recipe in so many ways! For example:

 - If you want to build a Docker container and pull down to Singularity, that's a good approach too! We have a [container-builder-template](github.com/autamus/container-builder-template) to help you authenticate with several popular registries.
 - The action matrix can be extended to run builds on multiple platforms.
 - You can also do the same, but test multiple versions of Singularity.

Have fun!
