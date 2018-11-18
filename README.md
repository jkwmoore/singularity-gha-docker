# Singularity Builder GitLab

![.gitlabci/sregistry-gitlab.png](.gitlabci/sregistry-gitlab.png)

This is a simple example of how you can achieve:

 - version control of your recipes
 - versioning to include image hash *and* commit id
 - build of associated container and
 - push to a storage endpoint (or GitLab artifact)

for a reproducible build workflow.

**Why should this be managed via GitLab?**

GitLab, by way of easy integration with continuous integration, is an easy way
to have a workflow set up where multiple people can collaborate on a container recipe,
the recipe can be tested (with whatever testing you need), discussed in pull requests,
and then finally pushed to be a GitLab artifact, to your storage of choice 
or to Singularity Registry.

**Why should I use this instead of a service?**

You could use a remote builder, but if you do the build in a continuous integration
service you get complete control over it. This means everything from the version of
Singularity to use, to the tests that you run for your container. You have a lot more
freedom in the rate of building, and organization of your repository, because it's you
that writes the configuration. Although the default would work for most, you can 
edit the build, setup, and circle configuration file in the 
[.gitlabci](.gitlabci) folder to fit your needs.

## Quick Start

Add your Singularity recipes to this repository, and edit the build commands in
the [build.sh](.gitlabci/build.sh) file. This is where you can specify endpoints 
(Singularity Registry, Dropbox, Google Storage, AWS) along with container names
(the uri) and tag. You can build as many recipes as you like, just add another line!

```yaml
                               # recipe relative to repository base
  - /bin/bash .gitlabci/build.sh Singularity
  - /bin/bash .gitlabci/build.sh --uri collection/container --tag tacos --cli google-storage Singularity
  - /bin/bash .gitlabci/build.sh --uri collection/container --cli google-drive Singularity
  - /bin/bash .gitlabci/build.sh --uri collection/container --cli globus Singularity
  - /bin/bash .gitlabci/build.sh --uri collection/container --cli registry Singularity
```

For each client that you use, required environment variables (e.g., credentials to push,
or interact with the API) must be defined in the (encrypted) Travis environment. To
know what variables to define, along with usage for the various clients, see
the [client specific pages](https://singularityhub.github.io/sregistry-cli/clients)

## Detailed Started

### 0. Fork this repository

You can clone and tweak, but it's easiest likely to get started with our example
files and edit them as you need.

### 1. Get to Know GitLab

We will be working with [GitLab](https://www.gitlab.com) repositories, which have
a built-in Continuous Integration service. Note that if you aren't using GitLab.com
or need to configure or enable runners, you can do this by going to your project page,
and clicking on the Settings --> CI/CD and selecting your "Runners." Typically a runner
is some external service, or the GitLab shared runners. I used GitLab.com so the 
shared runners were already active for me.
 
### 2. Add your Recipe(s)

For the example here, we have a single recipe named "Singularity" that is provided 
as an input argument to the [build script](.gitlabci/build.sh). You could add another 
recipe, and then of course call the build to happen more than once. 
The build script will name the image based on the recipe, and you of course
can change this. Just write the path to it (relative to the repository base) in
your [.gitlab-ci.yml](.gitlab-ci.yml).


### 3. Configure Singularity

The basic steps to [setup](.gitlabci/setup.sh) the build are the following:

 - Install Singularity, we use the release 2.6 branch as it was the last to not be written in GoLang. You could of course change the lines in [setup.sh](.gitlabci/setup.sh) to use a specific tagged release, an older version, or development version.
 - Install the sregistry client, if needed. The [sregistry client](https://singularityhub.github.io/sregistry-cli) allows you to issue a command like "sregistry push ..." to upload a finished image to one of your cloud / storage endpoints. By default, the push won't happen, and you will just build an image using the CI.

### 4. Configure the Build

The basic steps for the [build](.gitlabci/build.sh) are the following:

 - Running build.sh with no inputs will default to a recipe called "Singularity" in the base of the repository. You can provide an argument to point to a different recipe path, always relative to the base of your repository.
 - If you want to define a particular unique resource identifier for a finished container (to be uploaded to your storage endpoint) you can do that with `--uri collection/container`. If you don't define one, a robot name will be generated.
 - You can add `--uri` to specify a custom name, and this can include the tag, OR you can specify `--tag` to go along with a name without one. It depends on which is easier for you.
 - If you add `--cli` then this is telling the build script that you have defined the [needed environment variables](https://circleci.com/docs/2.0/env-vars/) for your [client of choice](https://singularityhub.github.io/sregistry-cli/clients) and you want successful builds to be pushed to your storage endpoint. See [here](https://singularityhub.github.io/sregistry-cli/clients) for a list of current client endpoints, or roll your own!

See the [.gitlab-ci.yml](.gitlab-ci.yml) for examples of this build.sh command (commented out). If there is some cloud service that you'd like that is not provided, please [open an issue](https://www.github.com/singularityhub/sregistry-cli/issues).

### 5. Pull your Container
If you want to pull from the artifacts folder in GitLab, check out the 
[Singularity Registry Client](https://www.github.com/singularityhub/sregistry-cli) 
(at the time of writing has a work in progress to pull images from the GitLab artifacts.)
Also remember that for different Singularity Registry Client Endpoints, you may need to configure environment
variables to help authenticate you. Instructions can be found [here](https://code.stanford.edu/help/ci/variables/README#variables).
