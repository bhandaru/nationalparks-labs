# Adding a Database (MongoDB)

In this lab, we’re going to deploy a Mongo database that will be used to store the data for the nationalparks application. We will also connect the nationalparks service with the newly deployed MongoDB database, so that the nationalparks service can load and query the database for the corresponding information.

Finally, we will mark the nationalparks application as a backend for the map visualization tool, so that it can be dynamically discovered by the parksmap component using the OpenShift discovery mechanism and the map will be displayed automatically.
![Application architecture](https://github.com/bhandaru/nationalparks-labs/blob/master/images/mongodb-arch.png)

## Background: Storage

Most useful applications are "stateful" or "dynamic" in some way, and this is usually achieved with a database or other data storage. In this lab we are going to add MongoDB to our ```nationalparks``` application and then rewire it to talk to the database using environment variables via a secret.

We are going to use the MongoDB image that is included with OpenShift.

By default, this will use **EmptyDir** for data storage, which means if the **Pod** disappears the data does as well. In a real application you would use OpenShift’s persistent storage mechanism to attach real-world storage (NFS, Gluster, EBS, etc) to the **Pods** to give them a persistent place to store their data.

## Exercise: Deploy MongoDB

As you’ve seen so far, the web console makes it very easy to deploy things onto OpenShift. When we deploy the database, we pass in some values for configuration. These values are used to set the username, password, and name of the database.

The database image is built in a way that it will automatically configure itself using the supplied information (assuming there is no data already present in the persistent storage!). The image will ensure that:  
* A database exists with the specified name
* A user exists with the specified name
* The user can access the specified database with the specified password

1. In the web console in your ```userXY``` project, click the **"Add to Project"** button and then **"From Container Image"**.
![Add to Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/post-login.png)

2. Enter ```centos/mongodb-36-centos7``` for the image name and ```mongodb``` for the Name.
![Add to Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/AddContainerImage.png)

3. Scroll down to select ```DeploymentConfig``` option and enter the following environment variables.
* MONGODB_USER : mongodb
* MONGODB_PASSWORD : mongodb
* MONGODB_DATABASE: mongodb
* MONGODB_ADMIN_PASSWORD : mongodb

![Add to Project](https://github.com/bhandaru/nationalparks-labs/blob/master/images/EnterMongoVars.png)

___
:information_source: Make sure to have configured the MONGODB_DATABASE parameter with the appropriate value as by default it will already have a value of sampledb.
___

4. Once you have entered in the above information, click on "Create" to deploy the MongoDB Container.
![MongoDB Deploy](https://github.com/bhandaru/nationalparks-labs/blob/master/images/final-mongo.png)

6. While the database deploys, we will fix the labels assigned to the deployment. Currenly we can not set a label when using templates from the Service Catalog, so in this case we will fix this manually.  

Like before, we will add 3 labels:
* app=workshop (the name we will be giving to the app)
* component=nationalparks (the name of this deployment)
* role=backend (the role this component plays in the overall application)

7. Execute the following command:
```
oc label dc/mongodb svc/mongodb app=workshop component=nationalparks role=database --overwrite
```

<!--
Now that the connection and authentication information is in a binding in our project, we need to add it to the nationalparks backend.
National Parks Binding

This will bring us to the binding information. Click Reveal Secret next to the word Opaque in the screen. Then click the Add to Application button.
Add binding to application

Select nationalparks application and click Save.
Add binding to application

This change in configuration will trigger a new deployment of the nationalparks application with the binding properly injected.
-->

## Exercise: Exploring OpenShift Magic

As soon as we changed the **DeploymentConfiguration**, some magic happened. OpenShift decided that this was a significant enough change to warrant updating the internal version number of the DeploymentConfiguration. You can verify this by looking at the output of ```oc get dc```:

```
NAME                    REVISION   DESIRED   CURRENT   TRIGGERED BY
mongodb-nationalparks   1          1         1         config,image(mongodb:3.2)
nationalparks           2          1         1         config,image(nationalparks:master)
parksmap                2          1         1         config,image(parksmap:1.2.0))
```

Something that increments the version of a DeploymentConfiguration, by default, causes a new deployment. You can verify this by looking at the output of oc get rc:

```
NAME                      DESIRED   CURRENT   READY     AGE
mongodb-nationalparks-1   1         1         1         24m
nationalparks-1           0         0         0         3h
nationalparks-2           1         1         1         8m
parksmap-1                0         0         0         6h
parksmap-2                1         1         1         5h
```

We see that the desired and current number of instances for the "-1" deployment is 0. The desired and current number of instances for the "-2" deployment is 1. This means that OpenShift has gracefully torn down our "old" application and stood up a "new" instance.

## Exercise: Data, Data, Everywhere

Now that we have a database deployed, we can again visit the nationalparks web service to query for data:
___
:information_source: In the following url(s), replace userXY with the project provided to you.
___  
```
http://nationalparks-userXY.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com/ws/data/all
```

And the result?

```
[]
```

Where’s the data? Think about the process you went through. You deployed the application and then deployed the database. Nothing actually loaded anything INTO the database, though.

The application provides an endpoint to do just that:
___
:information_source: In the following url(s), replace userXY with the project provided to you.
___
```
http://nationalparks-userXY.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com/ws/data/load
```

And the result?

```
Items inserted in database: 2740
```

If you then go back to ```/ws/data/all``` you will see tons of JSON data now. That’s great. Our parks map should finally work!

___
:information_source: There are some errors reported with browsers like Firefox 54 that don’t properly parse the resulting JSON. It’s a browser problem, and the application is working properly.
___
:information_source: In the following url(s), replace userXY with the project provided to you.
___

```
http://parksmap-userXY.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com/
```

Register the backend nationalparks service with parksmap frontend. Copy paste the following command into a browser window. Make sure to replace the userXY with the one assigned to you.

```
http://parksmap-userXY.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com/ws/backends/register?endpoint=nationalparks
```


Hmm…​ There’s just one thing. The main map STILL isn’t displaying the parks. That’s because the front end parks map only tries to talk to services that have the right Label.  
___
:information_source:  You are probably wondering how the database connection magically started working? When deploying applications to OpenShift, it is always best to use environment variables, secrets, or configMaps to define connections to dependent systems. This allows for application portability across different environments. The source file that performs the connection as well as creates the database schema can be viewed here:
___  
```
http://www.github.com/openshift-roadshow/nationalparks/blob/master/src/main/java/com/openshift/evg/roadshow/parks/db/MongoDBConnection.java#L44-l48
```

In short summary: By referring to bindings to connect to services (like databases), it can be trivial to promote applications throughout different lifecycle environments on OpenShift without having to modify application code.

## Exercise: Working With Labels

We explored how a Label is just a key=value pair earlier when looking at Services and Routes and Selectors. In general, a Label is simply an arbitrary key=value pair. It could be anything.
* pizza=pepperoni
* wicked=googly
* openshift=awesome

In the case of the parks map, the application is actually querying the OpenShift API and asking about the **Routes** and **Services** in the project. If any of them have a **Label** that is ```type=parksmap-backend```, the application knows to interrogate the endpoints to look for map data. You can see the code that does this here.

Fortunately, the command line provides a convenient way for us to manipulate labels. describe the nationalparks service:

```
$ oc describe route nationalparks

Name:                   nationalparks
Namespace:              userXY
Created:                2 hours ago
Labels:                 app=workshop
                        component=nationalparks
                        role=backend
Requested Host:         nationalparks-userXY.apps.osevg.openshiftworkshop.com
                        exposed on router router 2 hours ago
Path:                   <none>
TLS Termination:        <none>
Insecure Policy:        <none>
Endpoint Port:          8080-tcp

Service:                nationalparks
Weight:                 100 (100%)
Endpoints:              10.1.9.8:8080
```

You see that it already has some labels. Now, use oc label:

```
oc label route nationalparks type=parksmap-backend
```

You will see something like:

```
route "nationalparks" labeled
```

If you check your browser now:
:information_source: In the following url(s), replace userXY with the project provided to you.

http://parksmap-userXY.apps.cluster-dfw-a75a.dfw-a75a.example.opentlc.com//

![MongoDB](https://github.com/bhandaru/nationalparks-labs/blob/master/images/maps-final.png)

You’ll notice that the parks suddenly are showing up. That’s really cool!



<p align="center">
  <a href="/11%20-%20Deploying%20Java%20Code.MD">Previous Exercise</a> &nbsp;|
  &nbsp;<a href="/README.md">Table of Contents</a> &nbsp;|
  &nbsp;<a href="/13%20-%20Application%20Health.MD">Next Exercise</a>
</p>
