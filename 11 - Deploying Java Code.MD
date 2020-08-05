# Deploying Java Code

In this lab, we’re going to deploy a backend service, developed in Java that will expose 2 main REST endpoints to the visualizer application (parksmap web component that was deployed in the previous labs). The application will query for national parks information (including its coordinates) that is stored in a MongoDB database. This application will also provide an external access point, so that the API provided can be directly used by the end user.
![Application architecture](https://github.com/bhandaru/nationalparks-labs/blob/master/images/java-code-architecture-nationalparks-1.png)

## Background: Source-to-Image (S2I)

In a previous lab, we learned how to deploy a pre-existing image image. Now we will expand on that by learning how OpenShift builds container images using source code from an existing repository. This is accomplished using the Source-to-Image project.

[Source-to-Image (S2I)](https://github.com/openshift/source-to-image) is a open source project sponsored by Red Hat that has the following goal:

>_*"
Source-to-image (S2I) is a tool for building reproducible container images. S2I
produces ready-to-run images by injecting source code into a container image and
assembling a new container image which incorporates the builder image and built
source. The result is then ready to use with docker run. S2I supports
incremental builds which re-use previously downloaded dependencies, previously
built artifacts, etc."*_

OpenShift is S2I-enabled and can use S2I as one of its build mechanisms (in addition to building container images from Dockerfiles, and "custom" builds).

OpenShift runs the S2I process inside a special **Pod**, called a Build Pod, and thus builds are subject to quotas, limits, resource scheduling, and other aspects of OpenShift.

A full discussion of S2I is beyond the scope of this class, but you can find more information about it either in the [OpenShift S2I documentation](https://docs.openshift.com/container-platform/4.4/builds/understanding-image-builds.html#build-strategy-s2i_understanding-image-builds) or on [GitHub](https://github.com/openshift/source-to-image). The only key concept you need to remember about S2I is that it’s magic.

## Exercise: Creating a Java application

The backend service that we will be deploying as part of this exercise is called ```nationalparks```. This is a Java Spring Boot application that performs 2D geo-spatial queries against a MongoDB database to locate and return map coordinates of all National Parks in the world. That was just a fancy way of saying that we are going to deploy a webservice that returns a JSON list of places.

### Add to Project

Because the ```nationalparks``` component is a backend to serve data that our existing frontend (parksmap) will consume, we are going to build it inside the existing project that we have been working with. To illustrate how you can interact with OpenShift via the CLI or the web console, we will deploy the nationalparks component using the web console.

### Using Application Code on an Embedded Git Server

OpenShift can work with any accessible Git repository. This could be GitHub, GitLab, or any other server that speaks Git. You can even register webhooks in your Git server to initiate OpenShift builds triggered by any update to the application code!

The repository that we are going to use is already cloned in the internal Gogs repository and located at the following URL:

:information_source: In the following url(s), replace userXY with the Gogs username provided to you. Your Gogs credentials are the same as your OpenShift credentials.

```
http://gogs-ocp-workshop.apps.osevg.openshiftworkshop.com/userXY/nationalparks.git
```

Later in the lab, we want you to make a code change and then rebuild your application. This is a fairly simple Spring framework Java application.

## Build the Code on OpenShift

Similar to how we used "Add to project" before with an existing image, we can do the same for specifying a source code repository. Since for this lab you have your own git repository, let’s use it with a simple Java S2I image.
___
:information_source: In the following step(s), replace userXY with the project provided to you.
___
1. In the OpenShift web console, find your userXY project, and then click the "Add to Project" button and then the browse catalog link as highlighted in the following image:  
![Add to Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Dep1.png)  

2. This is the service catalog which allows a user to select components they want to add to their application. In this case, we are using Spring Boot so we want to select a JDK without an application server.  
Note how the platform detects the runtime needed for the deployment. Alternately, search for openjdk, and finally "Red Hat OpenJDK" as shown in the following image:  
![Service Catalog Languages](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Dep3.png)  

3. In the Git Repository field enter the base of the Git repository for your nationalparks application. This will cause the S2I process to grab that specific tag in the code repository. If you were to specify a specific branch or directory in the repository for your code, you would be able to do it in the Advanced Options.
```
https://github.com/openshift-roadshow/nationalparks.git
```  

4. After you select Red Hat OpenJDK, a dialog is presented as shown in the following image:  
![Create application](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Dep2.png)   

5. Select Java 8 from dropdown.  
___
:information_source: All of these runtimes shown are made available via Templates and ImageStreams, which will be discussed in a later lab.
___  
6. Don’t click Create, but click on Advanced Options, as there are more things we are going to configure.  
We will again set 3 labels to the deployment.  
```
app=workshop
component=nationalparks
role=backend
```  
![Service Catalog Languages](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Dep3.png)  
7. You can then hit the button labeled "Create". Then click Continue to the project overview and toggle the > icon next to the deployment config for nationalparks . You will see the build log output directly there.  
![Service Catalog Languages](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Dep4.png)  

This is a Java-based application that uses Maven as the build and dependency system. For this reason, the initial build will take a few minutes as Maven downloads all of the dependencies needed for the application. You can see all of this happening in real time!  

8. From the command line, you can also see the Builds:  
```
oc get builds
```  
You’ll see output like:  
```
NAME              TYPE      FROM          STATUS     STARTED              DURATION
nationalparks-1   Source    Git@b052ae6   Running    About a minute ago   1m2s
```  
You can also view the build logs with the following command:  
```
oc logs -f builds/nationalparks-1
```  
9. After the build has completed and successfully:  
* The S2I process will push the resulting image to the internal OpenShift registry
* The DeploymentConfiguration (DC) will detect that the image has changed, and this will cause a new deployment to happen.
* A ReplicationController (RC) will be spawned for this new deployment.
* The RC will detect no Pods are running and will cause one to be deployed, as our default replica count is just 1.

10. In the end, when issuing the oc get pods command, you will see that the build Pod has finished (exited) and that an application Pod is in a ready and running state:  
```
NAME                    READY     STATUS      RESTARTS   AGE
nationalparks-1-tkid3   1/1       Running     3          2m
nationalparks-1-build   0/1       Completed   0          3m
parksmap-1-4hbtk        1/1       Running     0          2h
```  
If you look again at the web console, you will notice that, when you create the application this way, OpenShift also creates a Route for you. You can see the URL in the web console, or via the command line:  
```
oc get routes
```  

Where you should see something like the following:

```
NAME            HOST/PORT                                                   PATH      SERVICES        PORT       TERMINATION
nationalparks   nationalparks-userXY.apps.osevg.openshiftworkshop.com             nationalparks   8080-tcp
parksmap        parksmap-userXY.apps.osevg.openshiftworkshop.com                  parksmap        8080-tcp
```

In the above example, the URL is:
___  
:information_source: In the following url(s), replace userXY with the project provided to you.  
___

http://nationalparks-userXY.apps.osevg.openshiftworkshop.com

Since this is a backend application, it doesn’t actually have a web interface. However, it can still be used with a browser. All backends that work with the parksmap frontend are required to implement a /ws/info/ endpoint. To test, the complete URL to enter in your browser is:
___  
:information_source: In the following url(s), replace userXY with the project provided to you.
___  
```
http://nationalparks-userXY.apps.osevg.openshiftworkshop.com/ws/info/
```
![Slash](https://github.com/bhandaru/nationalparks-labs/blob/master/images/slash.png)
___  
:information_source: The trailing slash is required.
___  
You will see a simple JSON string:

```
{"id":"nationalparks","displayName":"National Parks","center":{"latitude":"47.039304","longitude":"14.505178"},"zoom":4}
```

Earlier we said:

```
This is a Java Spring Boot application that performs 2D geo-spatial queries
against a MongoDB database
```

### But we don’t have a database yet!

<p align="center">
  <a href="/10%20-%20Remote%20Access%20to%20Application.MD">Previous Exercise</a> &nbsp;|
  &nbsp;<a href="/README.md">Table of Contents</a> &nbsp;|
  &nbsp;<a href="/12%20-%20Adding%20Database.MD">Next Exercise</a>
</p>
