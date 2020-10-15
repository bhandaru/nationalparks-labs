## Deploying Your First Container Image

In this lab, we’re going to deploy the web component of the ParksMap application which is also called parksmap and uses OpenShift’s service discovery mechanism to discover the backend services deployed and shows their data on the map.
![Application architecture](https://github.com/bhandaru/nationalparks-labs/blob/master/images/roadshow-app-architecture-parksmap-1.png)

## Exercise: Deploying your First Image

Let’s start by doing the simplest thing possible - get a plain old Docker-formatted image to run on OpenShift. This is incredibly simple to do. With OpenShift it can be done directly from the web console.

Return to the web console:

http://console-openshift-console.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com  
___  
:information_source: In the following steps, replace userXY with the project provided to you.
___  

1. Find your **userXY** project and click it. On the next page, click **"Add to Project"**. There are several options, but we are only concerned with "Container Image". Click **"Container Image"** in the **"Add to Project"** drop down.  

![Add to Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/post-login.png)

2. This will open a dialog that will allow you to specify the information for the image you want to deploy. We will learn more about image streams and image stream tags later. For now, select the "Image Name" option, and copy/paste the following into the box:  

```bhandaru/parksmap-katacoda:1.0.4```

3. Your screen will end up looking something like this:  
![Explore Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/container-deployment.png)

4. Either press enter or click on the magnifying glass. OpenShift will then go out to the Docker registry specified and interrogate the image. You then are presented with some options to add things like environment variables, labels, and etc. — which we will learn about later.

5. Make sure to have the correct application name:

6. Let’s scroll to the bottom of this page and add some labels to better identify this deployment later. Labels will help us identify and filter components in the web UI and in the command line. We will add 3 labels:

```
    app=workshop (the name we will be giving to the app)

    component=parksmap (the name of this deployment)

    role=frontend (the role this component plays in the overall application)
```
![Explore Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/container-labels.png)

7. Hit the blue "Create" button at the bottom of the dialog and then click the "Continue to the project overview." link or the "Close" button. Take a moment to look at the various messages that you now see on the overview page.

![Explore Project](/images/initialtopology.png)

___
:information_source:  
WINNING! These few steps are the only ones you need to run to get a container image deployed on OpenShift. This should work with any container image that follows best practices, such as defining an EXPOSE port, not needing to run specifically as the root user or other user name, and a single non-exiting CMD to execute on start.  

Providing appropriate labels is desired when deploying complex applications for organization purposes. OpenShift uses a label app to define and group components together in the Overview page. OpenShift will create this label with some default if the user don’t provide it explicitly.
___  

## Background: Containers and Pods

Before we start digging in, we need to understand how containers and Pods are related. We will not be covering the background on these technologies in this lab but if you have questions please inform the instructor. Instead, we will dive right in and start using them.

In OpenShift, the smallest deployable unit is a Pod. A Pod is a group of one or more OCI containers deployed together and guaranteed to be on the same host. From the official OpenShift documentation:

    Each pod has its own IP address, therefore owning its entire port space, and containers within pods can share storage. Pods can be "tagged" with one or more labels, which are then used to select and manage groups of pods in a single operation. 

Pods can contain multiple OCI containers. The general idea is for a Pod to contain a "main process" and any auxiliary services you want to run along with that process. Examples of containers you might put in a Pod are, an Apache HTTPD server, a log analyzer, and a file service to help manage uploaded files.


## Exercise: Examining the Pod

1. In the web console’s overview page you will see that there is a single Pod that was created by your actions.
Pod overview

![Explore Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/CheckPod.png)

2. You can also get a list of all the pods created within your project, by navigating to "Applications → Pods"
Pod list

This Pod contains a single container, which happens to be the parksmap application - a simple Spring Boot/Java application.

3. You can also examine Pods from the command line:

```
oc get pods
```

You should see output that looks similar to:

```
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-hx0kv   1/1       Running   0          2m
```

4. The above output lists all of the Pods in the current Project, including the Pod name, state, restarts, and uptime. Once you have a Pod's name, you can get more information about the Pod using the oc get command. To make the output readable, I suggest changing the output type to YAML using the following syntax:
	
Make sure you use the correct Pod name from your output.

```
oc get pod parksmap-1-hx0kv -o yaml
```

You should see something like the following output (which has been truncated due to space considerations of this workshop manual):

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/created-by: |
      {"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"explore-00","name":"parksmap-1","uid":"f1b37b1b-e3e2-11e6-81a2-0696d1181070","apiVersion":"v1","reso
urceVersion":"36222"}}
    kubernetes.io/limit-ranger: 'LimitRanger plugin set: cpu, memory request for container
      parksmap; cpu, memory limit for container parksmap'
    openshift.io/deployment-config.latest-version: "1"
    openshift.io/deployment-config.name: parksmap
    openshift.io/deployment.name: parksmap-1
    openshift.io/generated-by: OpenShiftWebConsole
    openshift.io/scc: restricted
  creationTimestamp: 2017-01-26T16:17:36Z
  generateName: parksmap-1-
  labels:
    app: parksmap
    deployment: parksmap-1
    deploymentconfig: parksmap
  name: parksmap-1-bvaz6
...............
```

5. The web interface also shows a lot of the same information on the Pod details page. If you click in the Pod circle, you will find the details page. You can also get there by clicking "Applications", then "Pods", at the left, and then clicking the Pod name.
Pod list

Getting the parksmap image running may take a little while to complete. Each OpenShift node that is asked to run the image has to pull (download) it, if the node does not already have it cached locally. You can check on the status of the image download and deployment in the Pod details page, or from the command line with the oc get pods command that you used before.


## Background: Customizing the Image Lifecycle Behavior

Whenever OpenShift asks the node’s CRI (Container Runtime Interface) runtime (Docker daemon or CRI-O) to run an image, the runtime will check to make sure it has the right "version" of the image to run. If it doesn’t, it will pull it from the specified registry.

There are a number of ways to customize this behavior. They are documented in specifying an image as well as image pull policy.
Background: Services

Services provide a convenient abstraction layer inside OpenShift to find a group of similar Pods. They also act as an internal proxy/load balancer between those Pods and anything else that needs to access them from inside the OpenShift environment. For example, if you needed more parksmap instances to handle the load, you could spin up more Pods. OpenShift automatically maps them as endpoints to the Service, and the incoming requests would not notice anything different except that the Service was now doing a better job handling the requests.

When you asked OpenShift to run the image, it automatically created a Service for you. Remember that services are an internal construct. They are not available to the "outside world", or anything that is outside the OpenShift environment. That’s okay, as you will learn later.

The way that a Service maps to a set of Pods is via a system of Labels and Selectors. Services are assigned a fixed IP address and many ports and protocols can be mapped.

There is a lot more information about Services, including the YAML format to make one by hand, in the official documentation.

Now that we understand the basics of what a Service is, let’s take a look at the Service that was created for the image that we just deployed. In order to view the Services defined in your Project, enter in the following command:

```
oc get services
```

You should see output similar to the following:

```
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
parksmap   ClusterIP   172.30.169.213  <none>        8080/TCP   3h
```

In the above output, we can see that we have a Service named parksmap with an IP/Port combination of 172.30.169.213/8080TCP. Your IP address may be different, as each Service receives a unique IP address upon creation. Service IPs are fixed and never change for the life of the Service.

In the web console, service information is available by clicking on "Administrator View" and then clicking "Networking -> Services".
Services list

You can also get more detailed information about a Service by using the following command to display the data in YAML:
```
oc get service parksmap -o yaml
```

You should see output similar to the following:
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    openshift.io/generated-by: OpenShiftWebConsole
  creationTimestamp: 2016-10-03T15:33:17Z
  labels:
    app: parksmap
  name: parksmap
  namespace: userXY
  resourceVersion: "6893"
  selfLink: /api/v1/namespaces/userXY/services/parksmap
  uid: b51260a9-897e-11e6-bdaa-2cc2602f8794
spec:
  clusterIP: 172.30.169.213
  ports:
  - name: 8080-tcp
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    deploymentconfig: parksmap
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
Take note of the selector stanza. Remember it.

Alternatively, you can use the web console to view information about the Service.
Service

It is also of interest to view the YAML of the Pod to understand how OpenShift wires components together. For example, run the following command to get the name of your parksmap Pod:

```
oc get pods
```

You should see output similar to the following:
```
NAME               READY     STATUS    RESTARTS   AGE
parksmap-1-hx0kv   1/1       Running   0          3h
```
Now you can view the detailed data for your Pod with the following command:
```
oc get pod parksmap-1-hx0kv -o yaml
```
Under the metadata section you should see the following:
```
  labels:
    app: workshop
    component: parksmap
    deployment: parksmap-1
    deploymentconfig: parksmap
    role: frontend
```
The Service has selector stanza that refers to deploymentconfig=parksmap.

The Pod has multiple Labels:
```
app=workshop
component=parksmap
role=frontend
deploymentconfig=parksmap
deployment=parksmap-1
```
Labels are just key/value pairs. Any Pod in this Project that has a Label that matches the Selector will be associated with the Service. To see this in action, issue the following command:
```
oc describe service parksmap
```
You should see something like the following output:
```
Name:                   parksmap
Namespace:              userXY
Labels:                 app=workshop
                        component=parksmap
                        role=frontend
Selector:               deploymentconfig=parksmap
Type:                   ClusterIP
IP:                     172.30.169.213
Port:                   8080-tcp        8080/TCP
Endpoints:              10.1.2.5:8080
Session Affinity:       None
Events:                 <none>
```
You may be wondering why only one endpoint is listed. That is because there is only one Pod currently running. In the next lab, we will learn how to scale an application, at which point you will be able to see multiple endpoints associated with the Service.


<p align="center">
  <a href="/04%20-%20Application%20Architecture.MD">Previous Exercise</a> &nbsp;|
  &nbsp;<a href="/README.md">Table of Contents</a> &nbsp;|
  &nbsp;<a href="/06%20-%20Scaling%20and%20Healing.MD">Next Exercise</a>
</p>
