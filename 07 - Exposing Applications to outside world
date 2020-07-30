# Exposing Your Application to the Outside World

In this lab, we’re going to make our application visible to the end users, so they can access it.
![Application architecture](https://github.com/bhandaru/nationalparks-labs/blob/master/images/exposing%20app%20arch.png)

## Background: Routes

While **Services** provide internal abstraction and load balancing within an OpenShift environment, sometimes clients (users, systems, devices, etc.) **outside** of OpenShift need to access an application. The way that external clients are able to access applications running in OpenShift is through the OpenShift routing layer. And the data object behind that is a **Route**.

The default OpenShift router (HAProxy) uses the HTTP header of the incoming request to determine where to proxy the connection. You can optionally define security, such as TLS, for the **Route**. If you want your Services, and, by extension, your Pods, to be accessible from the outside world, you need to create a **Route**.

## Exercise: Creating a Route

Fortunately, creating a Route is a pretty straight-forward process. You simply expose the Service via the command line. Or, via the web console, just click the "Create Route" button associated with the service and choose the defaults as highlighted in the following image:
![Create Route](https://github.com/bhandaru/nationalparks-labs/blob/master/images/Routes.png)

When creating a route, some options can be provided, like the hostname for the route or the TLS configuration for the Route.
Create Route editor

When using the command line, we can first verify that we don’t already have any existing route:

```
oc get routes
```
```
No resources found.
```

Now we need to get the Service name to expose:

```
oc get services
```
```
NAME       CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
parksmap   172.30.169.213   <none>        8080/TCP   5h
```

Once we know the Service name, creating a Route is a simple one-command task:

```
oc expose service parksmap
```
```
route "parksmap" exposed
```

Verify the Route was created with the following command:
```
oc get route
```
```
NAME       HOST/PORT                            PATH      SERVICES   PORT       TERMINATION
parksmap   pa
rksmap-userXY.apps.osevg.openshiftworkshop.com             parksmap   8080-tcp
```

You can also verify the Route by looking at the project in the OpenShift web console:
Route created

Pretty nifty, huh? This application is now available at the URL shown in the web console. Click the link and you will see:
Empty map

