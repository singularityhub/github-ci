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
2. [docker image](.github/workfolws/container.yml) discovers Singularity* changed files, and builds in a [docker image](https://quay.io/repository/singularity/singularity), also deploys to GitHub packages.
3. [manual deploy](.github/workfolws/manual-deploy.yml) takes a list of manually specified Singularity recipes (that aren't required to be changed), builds Singularity 3.x natively, and deploys to GitHub packages.

While the "build in a container" option is faster to complete and a more simple workflow, it should be noted that docker runs with
`--privileged` which may lead to issues with the resulting container in a non privileged situation. Also note that you
are free to mix and match the above recipes to your liking, or [open an issue](https://github.com/singularityhub/github-ci/issues) if you want to ask for help!

**Why should this be managed via Github?**

Github, by way of easy integration with **native** continuous integration, is an easy way
to have a workflow set up where multiple people can collaborate on a container recipe,
the recipe can be tested (with whatever testing you need), discussed in pull requests,
and tested on merge to master. If you add additional steps in the [build workflow](.github/workflows/native-install.yml)
you can also use [Singularity Registry Client](http://singularityhub.github.io/sregistry-cli) to push your container to a 
[Singularity Registry Server](https://singularityhub.github.io/sregistry) or other
cloud storage.

**Why should I use this instead of a service?**

You could use a remote builder, but if you do the build in a continuous integration
service you get complete control over it. This means everything from the version of
Singularity to use, to the tests that you run for your container. You have a lot more
freedom in the rate of building, and organization of your repository, because it's you
that writes the configuration.

## Quick Start

### 0. Enable Packages

If you want to use the [GitHub package registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
you'll need to follow the instructions there to enable packages for your organization, specifically "public" and "internal" packages should be allowed to be created.
You'll also want to add a username associated with your GitHub organization to the repository secret `GHCR_USERNAME`

### 1. Add Your Recipes

Add your Singularity recipes to this repository, and edit the [build workflow](.github/workflows/native-install.yml)
section where the container is built. The default will look for a recipe file called
"Singularity" in the base of the respository, [as we have here](Singularity).
For example, here is the default:

```yaml
    - name: Build Container
      env:
        SINGULARITY_RECIPE: Singularity
        OUTPUT_CONTAINER: container.sif
      run: |
       ls 
       if [ -f "${SINGULARITY_RECIPE}" ]; then
            sudo -E singularity build ${OUTPUT_CONTAINER} ${SINGULARITY_RECIPE}
       else
           echo "${SINGULARITY_RECIPE} is not found."
           echo "Present working directory: $PWD"
           ls
       fi
```

And I could easily change that to build as many recipes as I like, and 
even disregard the environment variable.

```yaml
    - name: Build Container
      run: |
        sudo -E singularity build smokey.sif Singularity.smokey
        sudo -E singularity build toasty.sif marshmallow/Singularity.toasty
```

### 2. Test your Container

Importantly, then you should test your container! Whether that's running it,
exec'ing a custom command, or invoking the test command, there is more than
one way to eat a reeses:

```yaml
    - name: Test Container
      run: |
        singularity exec smokey.sif python run_tests.py
        singularity test smokey.sif
        singularity run toasty.sif
```

### 3. Check Triggers

The workflow files each have a section at the top that indicates when the workflow will
trigger. By default, we will do builds on pull requests, and deploys on pushes to a main
branch. If you want to change this logic, edit the top of the recipe files.

### 4. Push to a registry

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
Remember that the example workflow is intended to run on push to master, so you might want to have
a similar one (without deployment) that runs on pull_request, or other events.
See [here](https://help.github.com/en/articles/about-github-actions#core-concepts-for-github-actions)
for getting started with GitHub actions, and [please open an issue](https://www.github.com/singularityhub/github-ci/issues)
if you need any help.


## Other Options

You can customize this base recipe in so many ways! For example:

 - If you are building a Docker container, you can start with the docker base, build the container, and then pull it down into Singularity and test it. Successful builds can be pushed to Docker Hub, and then you know they will pull okay to a Singularity container.
 - The action can be configured with a Matrix to run builds on multiple platforms.
 - You can also do the same, but test multiple versions of Singularity.

Have fun!
