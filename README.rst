**********************
CentOS Container Index
**********************
This is the master Index used by the `CentOS Container Pipeline Service <https://github.com/CentOS/container-pipeline-service>`_ to build and deliver containers at https://registry.centos.org

Quick-start
===========

1. Start with the repository containing the source of your container, such as a **Dockerfile**.
2. In the same directory, as the Dockerfile, create a `YAML <https://en.wikipedia.org/wiki/YAML>`_ 
   file containing *job-id: yourcontainername*. For example if you want your container to be tagged
   as foo/bar:latest, then you will add *job-id: bar*. The file name should be cccp.yml or .cccp.yml.
3. Now create your **own fork** of this repository, and clone it as per **standard git workflow**. The 
   container index is located in **index.d** with multiple yaml files.
4. Create a **foo.yml**, if it does **not already exist**, following the format in **index_template.yml**,
   **foo** being the **container namespace** and add an entry to it as is below. Assuming that the docker 
   file is in **my-container** directory in repository. Standard yml rules apply.

index.d/foo.yml:

.. code-block:: yaml

   Projects:
           - id: 1
             app-id: foo
             job-id: bar
             desired-tag: latest
             git-url: https://github.com/foo/bar.git
             git-path: mycontainer
             git-branch: master
             target-file: Dockerfile
             notify-email: you@example.com
             depends-on:
                   - centos/centos:latest
     
After this of course, all that remains is raising a git pull request and and working on getting it merged.

**IMPORTANT NOTES** : 

1. If your **Dockerfile** is at the **root of the repository**, then your **git-path** will be **/**, as in 
   *git-path: /* .
2. If you are running your own instance of the container pipeline service for development purposes,
   the way you can setup your index Container Pipeline Service Documentation. The format however, will remain
   the same, including the placement of index files in index.d.

If you need help with this, you can contact us on the #centos-devel channel on freenode, or the centos devel 
mailing list.   

The Details
===========

Mechanics
---------

The index.d directory in this git repository contains yaml formatted files with lists of all container applications included in the pipeline. In order to have your container application be included, tested and delivered via the Community Pipeline, it must be listed in this index. We are making resources behind the pipeline available to anyone on the internet who wishes to use them, provided what they are doing isn't illegal and is licensed in a way to be open source compatible. Note that the pipeline, its contents, all build and post build artifacts as well as delivery destinations should be considered publicly available.

The index.d files are processed hourly, so any new inclusions will be picked up fairly quickly. Once in the system, we will poll your git repository for changes every hour and initiate build runs as needed. Details for the build are included in the cccp.yml file that is hosted inside your git repository. Metadata from this file can be used to control the build, request changes to the delivery path, opt-in or out of the testing options available etc.

The *index.d* Directory
^^^^^^^^^^^^^^^^^^^^^^^

The index.d contains yaml formatted files, which must include :

 - *ID (id)*: Required: This should ideally be a number that uniquely identifies each entry in a file.
 - *App ID (app-id)* : Required: This will be namespace of your containers. For example postgresql containers will be under postgresql namespace so all the app-id's should be postgresql. 
 - *Job ID (job-id)* : Required: This will be the name of your container. In fact the final name of your container will be app-id/job-id:desired-tag.
 - *Git URL (git-url)* : Required: The complete URL to your git repository ( eg. https://github.com/projectatomic/atomicapp ). The Git repository can reside anywhere on the public internet. 
 - *Git Path (git-path)* : Required: The qualified path within the git repository to the cccp.yml file ( )
 - *Git Branch(git-branch)* : Required: Branch from the git repository you want processed, optional. Defaults to 'master'
 - *Target File (target-file)* : Required: The actual file from where the build will start. This would typically be Dockerfile.
 - *Notify Email (notify-email)*: The email id to which emails will be sent, upon success or failure of builds.
 - *Desired Tag (desired-tag)* : Required: The tag for container, such as latest.
 - *Depends On (depends-on)* : This would be a list of containers, already in the index that this container depends on. The list should container names in the form of "namespace/job-id:tag" of the containers the current entry relies on. This includes build time dependency containers, even if they are specified in the target file such as FROM in dockerfile. So even if your docker file mentions "FROM foo/bar:latest", the depends on list should explicitly include foo/bar:latest as well.
 
*Note :* The name of the file is going to be part of the container name (i.e the namespace). In fact your container will be based on the filename, job-id and desired-tag

For Example: File name :  foo.yml, and appi-id as well job-id: bar, desired-tag: latest, then container name will be foo/bar:latest

The *cccp.yml* File
^^^^^^^^^^^^^^^^^^^

Every build that we process is required to host a container pipeline control file, called the cccp.yml. You can host it as either .cccp.yml ( and then its just out of the way ), or as cccp.yml. An example of what this file might look like is included in this git repository. Feel free to use that as a template. This file is a standard yaml formatted file and includes the following information:

 - *Job ID (job-id)* : Required: This must match the Job ID that you insert into the index file.
 - *Nulecule File (nulecule-file)* : Optional - *Currently unusable*: Indicate a nulecule pattern file.
 - *Test Skip (test-skip)* : Optional (True or False): Indicate if you want to skip the test phase of the pipeline. Note, this only skips user scripts and not the standard tests that we run on every container.
 - *Test Script (test-script)* : Optional: Use to specify the path of the test script relative to the location of your target-file. This test script must use a non-zero exit code to indicate failure and can get to know the intermediate container tag with which to reference the image via the CONTAINER_NAME environment variable which is injected into the workers. Ensure that test-skip above is explicitly reset to False, as otherwise the test script will not be run (if test-skip is not specified, it is assumed to be True)
 - *Build Script (build-script)* : Optional - *Currently unusable*: Custom build script
 - *Delivery script (delivery-script)* : Optional: This would be where you can specify a custom delivery script.
 - *Docker Index (docker-index)* : Optional - *Currently unusable* : If true, then container is delivered to docker hub.
 - *Custom Delivery (custom-delivery)* : Optional - *Currently unusable* : Specify a script for your own delivery mechanisms
 - *Local Delivery* : Optional - *Currently unusable* : This flag can be used to disable delivery to r.c.o
 - *Upstreams* : Optional - *Currently unusable* : This can be used to specify upstreams to track and rebuild based on.

