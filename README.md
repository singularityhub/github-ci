# Singularity Builder GitHub CI

![img/sregistry-github-small.png](img/sregistry-github-small.png)

This is a simple example of how you can achieve:

 - version control of your recipes
 - versioning to include image hash *and* commit id
 - build of associated container and
 - (optional) push to a storage endpoint

for a reproducible build workflow. This recipe on master is intended to build
Singularity 3.x (with GoLang).

**Why should this be managed via Github?**

Github, by way of easy integration with **native** continuous integration, is an easy way
to have a workflow set up where multiple people can collaborate on a container recipe,
the recipe can be tested (with whatever testing you need), discussed in pull requests,
and tested on merge to master. If you add additional steps in the [build workflow](.github/workflows/go.yml)
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

### 1. Add Your Recipes

Add your Singularity recipes to this repository, and edit the [build workflow](.github/workflows/go.yml)
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

### 3. Push to a registry

You might be done there. But if not, you can install [Singularity Registry Client](http://singularityhub.github.io/sregistry-cli) and push to your cloud storage of choice! You will want to add python and python-dev to the dependency
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
