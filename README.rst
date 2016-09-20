This is the master Index used by the CentOS Container Pipeline to build and deliver containers at https://registry.centos.org

Usage Quickstart
================

 1. Fork and clone this git repository.
 2. Take the cccp.yml file included in this repo and drop it into the same directory as the container build file such as a Dockerfile.
 3. Populate the template as per your needs.
 4. Get into the index.d, and if there isnt already a yml file present for your container namespace, you can copy and paste the index_template.yml file and edit it. All values are required.
 5. Note that the file name should compulsorily be namespace.yml while all entries will have app-id as again namespace. For example if you are building openshift containers containers, then you will have an openshift.yml and and all app-id's must also be openshift.yml.
 6. Send a PR to this repository and wait for it to be merged, following any instructions, provided by the maintainers.
 7. Once the PR is merged, and the pipeline has picked up your artifacts, your container will be built, tested and delivered to registry.centos.org. You should recieve a mail indicating the status of the build(s) once completed.

If you need help with this, you can contact us on the #centos-devel or #nulecule channel on freenode, or the centos devel mailing list.

Mechanics
=========

The index.d directory in this git repository contains yaml formatted files with lists of all container applicatons included in the pipeline. In order to have your container application be included, tested and delivered via the Community Pipeline, it must be listed in this index. We are making resources behind the pipeline available to anyone on the internet who wishes to use them, provided what they are doing isnt illegal and is licensed in a way to be open source compatible. Note that the pipeline, its contents, all build and post build artifacts as well as delivery destinations should be considered publicly available.

The index.d files are processed hourly, so new inclusions will be picked up fairly quickly. Once in the system, we will poll your git repository for changes every hour and initiate build runs as needed. Details for the build are included in the cccp.yml file that is hosted inside your git repo. Metadata from this file can be used to control the build, request changes to the delivery parth, opt-in or out of the testing options available etc.

The *index.d* Directory
-----------------------

The index.d contains yaml formatted files, which must include :

 - *ID (id)*: Required: This should ideally be a number that uniquely identifies each entry in a file.
 - *App ID (app-id)* : Required: This will be namespace of your containers. For example postgresql containers will be under postgresql namespace so all the app-id's should be postgresql. 
 - *Job ID (job-id)* : Required: This will be the name of your container. Infact the final name of your container will be app-id/job-id:desired-tag.
 - *Git URL (git-url)* : Required: The complete url to your git repo ( eg. https://github.com/projectatomic/atomicapp ). The Git repo can reside anywhere on the public internet. 
 - *Git Path (git-path)* : Required: The qualified path within the git repo to the cccp.yml file ( )
 - *Git Branch(git-branch)* : Required: Branch from the git repo you want processed, optional. Defaults to 'master'
 - *Target File (target-file)* : Required: The actual file from where the build will start. This would typically be Dockerfile.
 - *Notify Email (notify-email)*: The email id to which emails will be sent, upon success or failure of builds.
 - *Desired Tag (desired-tag)* : Required: The tag for container, such as latest.
 - *Depends On (depends-on)* : This would be a list of containers, already in the index that this container depends on. The list should container container names in the form of "namespace/job-id:tag" of the containers the current entry relies on.
 
*Note :* : The name of the file is going to be part of the container name (i.e the namespace). Infact your container will be based on the filename, jobid and desired-tag 

For Example: File name :  hello.yml, job-id: mycontainer, desired-tag: latest, then container name will be hello/mycontainer:latest

The *cccp.yml* File
-------------------

Every build that we process is required to host a container pipeline control file, called the cccp.yml. You can host it as either .cccp.yml ( and then its just out of the way ), or as cccp.yml. An example of what this file might look like is included in this git repo. Feel free to use that as a template. This file is a standard yaml formatted file and includes the following information:

 - *Job ID (job-id)* : Required: This must match the Job ID that you insert into the index file.
 - *Nulecule File (nulecule-file)* : Optional - Currently unusable: Indicate a nulecule pattern file.
 - *Test Skip (test-skip)* : Optional (True or False): Indicate if you want to skip the test phase of the pipeline. Note, this only skips user scripts and not the standard tests that we run on every container.
 - *Test Script (test-script)* : Optional: Use to specify the path of the test script relative to the location of your target-file from index.yml. This test script must use a non-zero exit code to indicate failure.
 - *Build Script (build-script)* : Optional - Currently unusable:
 - *Delivery script (delivery-script)* : Optional: This would be where you can specify a custom delivery script.
