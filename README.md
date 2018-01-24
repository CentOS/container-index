# CentOS Container Index

This is the master index used by the [CentOS Community Container Pipeline Service](https://github.com/CentOS/container-pipeline-service) to build and deliver containers at [registry.centos.org](https://wiki.centos.org/ContainerPipeline)

# Quickstart

In this tutorial we will learn to deploy a Docker container to the [CentOS Registry](https://registry.centos.org).

__First, we push changed to the same directory as our Dockerfile:__

1. Change to the directory container your Dockerfile
```sh
cd yourdirectory
```
2. Download the example `cccp.yml` file from https://github.com/CentOS/container-index
```sh
wget https://raw.githubusercontent.com/CentOS/container-index/master/cccp.yml
```
3. Customize the `cccp.yml` file as needed
```sh
vim cccp.yml
```
4. Push the changes to your (public) Git repository
```
git add .
git commit -m "Added cccp.yml"
git push
```

__Second, we will open a PR request to add your container to the CentOS registry:__

1. Fork and then git clone the container index
```sh
git clone https://github.com/yourusername/container-index
cd container-index
```
2. Create a new namespace by copying the `index_template.yml`
```sh
cp index.d/index_template.yml index.d/yourgitname.yml
```
3. Add your container details to the index file, customizing it to your needs. Note that the app-id must be the same as the namespace filename containing your project.
```sh
vim index.d/yourgitname.yml
```
4. Commit your changes and open a pull request against https://github.com/CentOS/container-index
```
git add .
git commit -m "Added cccp.yml"
git push
```

**Notes:**

 - Even if CI has passed your pull request, please go through the CI build logs for warnings against your changes. Unless there is an explicit and appropriate reason why they should be ignored, the maintainers wont ignore them.
 - Preferably, add comments to the index to give more information, such as if you want to group containers together, based on function. For example : If you have 3 mariadb containers under the CentOS namespace, then group them together with something like
  ``` 
  # My MariaDB Containers
  Your entries here
  # My MariaDB Containers ends
  ```
 - Once the PR is merged, and the pipeline has picked up your artifacts, your container will be built, tested and delivered to registry.centos.org. You should receive a mail indicating the status of the build(s) once completed.

# File Reference

## `index.d` directory

The index.d contains yaml formatted files, which must include :

| Key              | Type           | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|------------------|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id               | int            | Yes      | This should ideally be a number that uniquely identifies each entry in a file.                                                                                                                                                                                                                                                                                                                                                                                            |
| app-id           | string         | Yes      | This will be namespace of your containers. For example postgresql containers will be under postgresql namespace so all the app-id's should be postgresql.                                                                                                                                                                                                                                                                                                                 |
| job-id           | string         | Yes      | This will be the name of your container. Infact the final name of your container will be app-id/job-id:desired-tag.                                                                                                                                                                                                                                                                                                                                                       |
| git-url          | string         | Yes      | The complete url to your git repo ( eg. https://github.com/username/repo ). The Git repo can reside anywhere on the public internet. If this is a Gitlab URL, url must end with .git.                                                                                                                                                                                                                                                                                     |
| git-path         | string         | Yes      | The qualified path within the git repo to the Dockerfile, and cccp.yml should be on the same directory as the dockerfile.                                                                                                                                                                                                                                                                                                                                                 |
| git-branch       | string         | No       | Defaults to 'master'. Branch from the git repo you want processed.                                                                                                                                                                                                                                                                                                                                                                                                        |
| target-file      | string         | Yes      | The actual file from where the build will start. This would typically be Dockerfile.                                                                                                                                                                                                                                                                                                                                                                                      |
| notify-email     | string         | Yes      | The email  to which status emails will be sent, upon success or failure of builds.                                                                                                                                                                                                                                                                                                                                                                                        |
| desired-tag      | string         | Yes      | The tag for container, such as latest.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| build-context    | string         | Yes      | A path, relative to the git path, which needs to be included during the build. For Dockerfiles, it would be what you give as last parameter to a `Docker build` command. The default value is ".".                                                                                                                                                                                                                                                                        |
| prebuild-script  | string         | No       | This would be the path, relative to the repository root, of the script, you wish to run, before the build happens. For example, it could a script that generates the artifacts required during the image build.                                                                                                                                                                                                                                                           |
| prebuild-context | string         | No       | This value needs to be specified with prebuild script. It is the path, relative to repository root, where you want to be, when you run the prebuild script.                                                                                                                                                                                                                                                                                                               |
| depends-on       | string / array | Yes      | This would be a list of containers, already in the index that this container depends on. The list should container names in the form of "namespace/job-id:tag" of the containers the current entry relies on. This includes build time dependency containers, even if they are specified in the target file such as FROM in dockerfile. So even if your docker file mentions "FROM foo/bar:latest", the depends on list should explicitly include foo/bar:latest as well. |


**Note:** 

The name of the YAML file will be part of the container name. For example, if your username was `foo` and your container `bar`. The container would be accessible at `registry.centos.org/foo/bar`.

Example:

```
foo.yml, job-id: bar, desired-tag: latest, then container name will be foo/bar:latest
```

## `cccp.yml`

Every build that we process is required to contain `cccp.yml` (or `cccp.yaml` if you prefer). An example is located at: https://raw.githubusercontent.com/CentOS/container-index/master/cccp.yml

| Key             | Type     | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-----------------|----------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| job-id          | string   | Yes      | This must match the Job ID that you insert into the index file.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| test-skip       | boolean  | No       | Indicate if you want to skip the test phase of the pipeline. Note, this only skips user scripts and not the standard tests that we run on every container.                                                                                                                                                                                                                                                                                                                                                       |
| test-script     | string   | No       | Use to specify the path of the test script relative to the location of your target-file from index.yml. This test script must use a non-zero exit code to indicate failure and can get to know the intermediate container tag with which to reference the image via the CONTAINER_NAME environment variable which is injected into the workers. Ensure that test-skip above is explicitly reset to False, as otherwise the test script will not be run (if test-skip is not specified, it is assumed to be True) |
| build-script    | _unused_ | No       | Custom build script                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| delivery-script | _unused_ | No       | This would be where you can specify a custom delivery script.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| docker-index    | _unused_ | No       | If true, then container is delivered to docker hub.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| custom-delivery | _unused_ | No       | Specify a script for your own delivery mechanisms                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| local-delivery  | _unused_ | No       | This flag can be used to disable delivery to r.c.o                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| upstreams       | _unused_ | No       | Specify upstreams to track and rebuild based on                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

# How does it work?

The [index.d](https://github.com/CentOS/container-index/tree/master/index.d) directory in this git repository contains yaml formatted files with lists of all container applicatons included in the pipeline. In order to have your container application be included, tested and delivered via the Community Pipeline, it must be listed in this index. We are making resources behind the pipeline available to anyone on the internet who wishes to use them, provided what they are doing is not illegal and is licensed in a way to be open source compatible. Note that the pipeline, its contents, all build and post build artifacts as well as delivery destinations should be considered publicly available.

The index.d files are processed frequently, so new inclusions will be picked up fairly quickly. Once in the system, we will poll your git repository for changes every hour and initiate build runs as needed. Details for the build are included in the cccp.yml file that is hosted inside your git repo. Metadata from this file can be used to control the build, request changes to the delivery path, opt-in or out of the testing options available, etc.

