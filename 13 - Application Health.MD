# Application Health

## Background: Readiness and Liveness Probes

As we have seen before in the UI via warnings, there is a concept of application health checks in OpenShift. These come in two flavors:
* Readiness probe
* Liveness probe

From the Application Health section of the documentation, we see the definitions:

**Liveness Probe**

	A liveness probe checks if the container in which it is configured is still running. If the liveness probe fails, the kubelet kills the container, which will be subjected to its restart policy. Set a liveness check by configuring the template.spec.containers.livenessprobe stanza of a pod configuration.

**Readiness Probe**

    A readiness probe determines if a container is ready to service requests. If the readiness probe fails a container, the endpoints controller ensures the container has its IP address removed from the endpoints of all services. A readiness probe can be used to signal to the endpoints controller that even though a container is running, it should not receive any traffic from a proxy. Set a readiness check by configuring the template.spec.containers.readinessprobe stanza of a pod configuration.

It sounds complicated, but it really isn’t. We will use the web console to add these probes to our nationalparks application.

## Exercise: Add Health Checks

As we are going to be implementing a realistic CI/CD pipeline, we will be doing some testing of the "development" version of the application. However, in order to test the app, it must be ready. This is where OpenShift’s application health features come in very handy.

We are going to add both a readiness and liveness probe to the existing nationalparks deployment. This will ensure that OpenShift does not add any instances to the service until they pass the readiness checks, and will ensure that unhealthy instances are restarted (if they fail the liveness checks).

Click **Developer View → Topology** on the left-side bar. Click ```nationalparks```. Click on Configuration. You will see not any health check configuration, with a link to click in order to add them.

Click "Edit Deployment" from the menu option 

Add below to the container spec of the deployment using the built in YAML Editor.

___
:information_source: Hint: Use control space for prompts to add.
___

Readiness Probe

    Path: /ws/healthz/

    Initial Delay: 20

    Timeout: 1

Liveness Probe

    Path: /ws/healthz/

    Initial Delay: 120

    Timeout: 1
<!--
This is shown in the following image:
Readiness and Liveness Probe
-->
___
:information_source: Note the trailing slash in the URL
___
![HealthCheck](https://github.com/bhandaru/nationalparks-labs/blob/master/images/healthcheck-1.png)  

Once you have entered in the values show above, click Save and then click the Overview button in the left navigation. You will notice that these changes caused a new deployment — they counted as a configuration change.
<!--
You will also notice that the circle around the new deployment stays light blue for a while. This is a sign that the pod(s) have not yet passed their readiness checks — it’s working!
Application Health
-->

