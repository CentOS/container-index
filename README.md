This is the master Index used by the [CentOS Community Container Pipeline Service](https://github.com/CentOS/container-pipeline-service) to build and deliver containers at [registry.centos.org](https://wiki.centos.org/ContainerPipeline)

# Quickstart

1. Fork and clone https://github.com/CentOS/container-index.
1. Copy ``cccp.yml`` file included in the above repo into your project's git repo directory containing the ``Dockerfile``.
1. Customize ``cccp.yml`` as needed and, commit and push the changes to git.
1. Back in the container-index repo, in the ``index.d`` directory, you can do either
    1. Put your project in a new namespace ``yml`` file, by copying ``index.d/index_template.yml``
    file in this repo to a desired file, and customizing it as needed.
    1. You can add your project details in an existing namespace ``yml`` file
    by following the example in ``index.d/index_template.yml`` file.
1. Please note that, in your project details, app-id must be the same as the namespace filename containing your project.
1. Commit your changes and send a pull request to https://github.com/CentOS/container-index and wait for a maintainer to accept it.

**Notes**

 - Even if CI has passed your pull request, please go through the CI
   build logs for warnings against your changes. Unless there is an
   explicit and appropriate reason why they should be ignored, the
   maintainers wont ignore them.
 - Preferably, add comments to the index to give more information,      
   such as if you want to group containers together, based on function  
   For example : If you have 3 mariadb containers under centos   
   namespace,    Then group them together with something like
  
          # My MariaDB Containers
          Your entries here
          # My MariaDB Containers ends
    
 - Once the PR is merged, and the pipeline has picked up your artifacts,
   your container will be built, tested and delivered to
   registry.centos.org. You should receive a mail indicating the status
   of the build(s) once completed.

# How it works?

The [index.d](https://github.com/CentOS/container-index/tree/master/index.d) directory in this git repository contains yaml formatted files with lists of all container applicatons included in the pipeline. In order to
have your container application be included, tested and delivered via the
Community Pipeline, it must be listed in this index. We are making resources
behind the pipeline available to anyone on the internet who wishes to use them,
provided what they are doing is not illegal and is licensed in a way to be
open source compatible. Note that the pipeline, its contents, all build and
post build artifacts as well as delivery destinations should be considered
publicly available.

The index.d files are processed frequently, so new inclusions will be picked up
fairly quickly. Once in the system, we will poll your git repository for
changes every hour and initiate build runs as needed. Details for the build
are included in the cccp.yml file that is hosted inside your git repo.
Metadata from this file can be used to control the build, request changes to
the delivery path, opt-in or out of the testing options available, etc.

## The *index.d* Directory

The index.d contains yaml formatted files, which must include :

- **ID (id)**: Required: This should ideally be a number that uniquely identifies each entry in a file.
- **App ID (app-id)** : Required: This will be namespace of your containers. For example postgresql containers will be under postgresql namespace so all the app-id's should be postgresql.
- **Job ID (job-id)** : Required: This will be the name of your container. Infact the final name of your container will be app-id/job-id:desired-tag.
- **Git URL (git-url)** : Required: The complete url to your git repo ( eg. https://github.com/projectatomic/atomicapp ). The Git repo can reside anywhere on the public internet. If this is a Gitlab URL, url must end with .git.
- **Git Path (git-path)** : Required: The qualified path within the git repo to the Dockerfile, and cccp.yml should be on the same directory as the dockerfile.
- **Git Branch(git-branch)** : Required: Branch from the git repo you want processed, optional. Defaults to 'master'
- **Target File (target-file)** : Required: The actual file from where the build will start. This would typically be Dockerfile.
- **Notify Email (notify-email)**: The email id to which status emails will be sent, upon success or failure of builds.
- **Desired Tag (desired-tag)** : Required: The tag for container, such as latest.
- **Build Context(build-context)** : Required: A path, relative to the git path, which needs to be included
during the build. For Dockerfiles, it would be what you give as last parameter to a `docker build` command.
The default value is ".".
- **Prebuild Script(prebuild-script)** : Optional: This would be the path, relative to the repository root, of the
script, you wish to run, before the build happens. For example, it could a script that generates the artifacts required during the image build.
- **Prebuild Context(prebuild-context)**: Required with prebuild-script: This value needs to be specified with 
prebuild script. It is the path, relative to repository root, where you want to be, when you run the prebuild
script.
- **Depends On (depends-on)** : This would be a list of containers, already in the index that this container depends on. The list should container names in the form of "namespace/job-id:tag" of the containers the current entry relies on. This includes build time dependency containers, even if they are specified in the target file such as FROM in dockerfile. So even if your docker file mentions "FROM foo/bar:latest", the depends on list should explicitly include foo/bar:latest as well.

**Note** The name of the file is going to be part of the container name (i.e the namespace). Infact your container will be based on the filename, jobid and desired-tag

For Example: File name :  hello.yml, job-id: mycontainer, desired-tag: latest, then container name will be hello/mycontainer:latest

## The *cccp.yml* File

Every build that we process is required to host a container pipeline control file, called the cccp.yml. You can host it as cccp.yml. An example of what this file might look like is included in this git repo. Feel free to use that as a template. This file is a standard yaml formatted file and includes the following information:

- **Job ID (job-id)** : Required: This must match the Job ID that you insert into the index file.
- **Nulecule File (nulecule-file)** : Optional - *Currently unusable*: Indicate a nulecule pattern file.
- **Test Skip (test-skip)** : Optional (True or False): Indicate if you want to skip the test phase of the pipeline. Note, this only skips user scripts and not the standard tests that we run on every container.
- **Test Script (test-script)** : Optional: Use to specify the path of the test script relative to the location of your target-file from index.yml. This test script must use a non-zero exit code to indicate failure and can get to know the intermediate container tag with which to reference the image via the CONTAINER_NAME environment variable which is injected into the workers. Ensure that test-skip above is explicitly reset to False, as otherwise the test script will not be run (if test-skip is not specified, it is assumed to be True)
- **Build Script (build-script)** : Optional - *Currently unusable*: Custom build script
- **Delivery script (delivery-script)** : Optional: This would be where you can specify a custom delivery script.
- **Docker Index (docker-index)** : Optional - *Currently unusable* : If true, then container is delivered to docker hub.
- **Custom Delivery (custom-delivery)** : Optional - *Currently unusable* : Specify a script for your own delivery mechanisms
- **Local Delivery** : Optional - *Currently unusable* : This flag can be used to disable delivery to r.c.o
- **Upstreams** : Optional - *Currently unusable* : This can be used to specify upstreams to track and rebuild based on.

