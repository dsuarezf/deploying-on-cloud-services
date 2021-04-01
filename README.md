# deploying-on-cloud-services

This repository contains a very simple HTTP server coded in Java to be used
as a example project to deploy on Cloud Services as [Openshift Online] or
[Google Cloud App Engine].

The service itself is a tiny HTTP Server listening on port 8080.

## Working with Openshift

### Deploying on Openshift

First of all we need to have the OpenShift CLI installed. It can be downloaded
from the OpenShift site.

The first step is to login by typing:

    oc login

To create the project in OpenShift:

    oc new-project my-java-project

We'll use the S2I to deploy our application:

    oc new-app redhat-openjdk18-openshift~https://github.com/dsuarezf/s2i-ultra-tiny-http-service.git

And this is the output:

    --> Found image 5331d25 (4 months old) in image stream "openshift/redhat-openjdk18-openshift" under tag "latest" for "redhat-openjdk18-openshift"
    
        Java Applications
        -----------------
        Platform for building and running plain Java applications (fat-jar and flat classpath)
    
        Tags: builder, java
    
        * A source build using source code from https://github.com/dsuarezf/s2i-ultra-tiny-http-service.git will be created
          * The resulting image will be pushed to image stream "s2i-ultra-tiny-http-service:latest"
          * Use 'start-build' to trigger a new build
        * This image will be deployed in deployment config "s2i-ultra-tiny-http-service"
        * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "s2i-ultra-tiny-http-service"
          * Other containers can access this service through the hostname "s2i-ultra-tiny-http-service"
    
    --> Creating resources ...
        imagestream "s2i-ultra-tiny-http-service" created
        buildconfig "s2i-ultra-tiny-http-service" created
        deploymentconfig "s2i-ultra-tiny-http-service" created
        service "s2i-ultra-tiny-http-service" created
    --> Success
        Build scheduled, use 'oc logs -f bc/s2i-ultra-tiny-http-service' to track its progress.
        Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
         'oc expose svc/s2i-ultra-tiny-http-service'
        Run 'oc status' to view your app.

As we can see within the output, OpenShift uploads the code and creates a builder
container based on the image *redhat-openjedk18-openshift*.

The builder's log can be seen by typing:

    oc logs -f bc/s2i-ultra-tiny-http-service

The container is deployed within a Kubernetes' pod and run as a service, we can
see the service status by typing:

    oc get service
    
    NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    s2i-ultra-tiny-http-service   ClusterIP   172.30.137.215   <none>        8080/TCP,8443/TCP,8778/TCP   6m

The just deployed service run within a cluster with IP 172.30.137.215 and ports
8080, 8443 and 8778.

The service can be exposed using:

    oc expose svc/s2i-ultra-tiny-http-service --port=8080

To delete the application:

    oc delete all -l app=s2i-ultra-tiny-http-service

To delete the project we can use the following command:

    oc delete project <project-name>

## Working with Google Cloud App Engine

## Installation

Google Cloud App Engine use the [Google Cloud SDK] (GKS) to ease the interaction
with all the Google Cloud products. To install the SDK just follow the instructions
[[3]].

To configure de SDK execute:

    gcloud init

This will authenticate with a Google Account, create or specify the default project
and so on.

To verify everything is correctly installed we can execute:

    gcloud components list

To see current configuration:

    gcloud config list

## Deploying on Google Cloud App Engine

As Openshift's S2i, Google App Engine is designed to easely deploy applications
by creating container images depending on the language used, for the Java case
it is mandatory to use Maven, and therefore, have a proper `pom.xml` project
file in place.

To deploy the application just type:

    gcloud app deploy

This is more or less the output:

    Services to deploy:

    descriptor:      [/home/dsuarez/projects/github/deploying-on-cloud-services/pom.xml]
    source:          [/home/dsuarez/projects/github/deploying-on-cloud-services]
    target project:  [java-app-307616]
    target service:  [default]
    target version:  [20210401t191429]
    target url:      [https://java-app-307616.ew.r.appspot.com]

    Do you want to continue (Y/n)?  Y

    Beginning deployment of service [default]...
    ╔════════════════════════════════════════════════════════════╗
    ╠═ Uploading 14 files to Google Cloud Storage               ═╣
    ╚════════════════════════════════════════════════════════════╝
    File upload done.
    Updating service [default]...done.                                                                          
    Setting traffic split for service [default]...done.                                                         
    Deployed service [default] to [https://java-app-307616.ew.r.appspot.com]

    You can stream logs from the command line by running:
    $ gcloud app logs tail -s default

    To view your application in the web browser run:
    $ gcloud app browse

As indicated, you can browse the application by just typing:

    gcloud app browse

[Google Cloud App Engine]: https://cloud.google.com/appengine
[Google Cloud SDK]: https://cloud.google.com/sdk
[Openshift Online]: https://cloud.redhat.com/openshift/
[1]: https://access.redhat.com/documentation/en-us/red_hat_jboss_middleware_for_openshift/3/html-single/red_hat_java_s2i_for_openshift/index
[2]: https://docs.openshift.com/container-platform/3.5/dev_guide/builds/build_inputs.html#source-secrets-ssh-key-authentication
[3]: https://cloud.google.com/sdk/docs/install