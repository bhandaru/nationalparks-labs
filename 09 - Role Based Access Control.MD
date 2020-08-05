# Role-Based Access Control

Almost every interaction with an OpenShift environment that you can think of requires going through the OpenShift’s control plane API. All API interactions are both authenticated (AuthN - who are you?) and authorized (AuthZ - are you allowed to do what you are asking?).

In the log aggregation lab we saw that there was an error in reference to a **Service Account**.

As OpenShift is a declarative platform, some actions will be performed by the platform and not by the end user (when he or she issues a command). These actions are performed using a **Service Account** which is a special type of user that the platform will use internally.

OpenShift automatically creates a few special service accounts in every project. The **default** service account is the one taking the responsibility of running the pods, and OpenShift uses and injects this service account into every pod that is launched. By changing the permissions for that service account, we can do interesting things.

You can view current permissions in the web UI, by navigating to Administrator View and "User Management → Servicee Account" page.
![Membership](https://github.com/bhandaru/nationalparks-labs/blob/master/images/ServiceAccount.png)

## Exercise: Grant Service Account View Permissions

The parksmap application wants to talk to the OpenShift API to learn about other Pods, Services, and resources within the Project. You’ll soon learn why!
___
:information_source: In the following command(s), replace userXY with the project provided to you.
___

1. Ensure the current project is your project. Run the following command from CLI or choose the project from  Web Console.
```
oc project userXY
```

2. Then execute the following command.

```
oc policy add-role-to-user view -z default
```

The oc policy command above is giving a defined role (view) to a user. But we are using a special flag, -z. What does this flag do? From the -h output:

```
-z, --serviceaccount=[]: service account in the current namespace to use as a user
```

The ```-z``` syntax is a special one that saves us from having to type out the entire string, which, in this case, is ```system:serviceaccount:userXY:default```. It’s a nifty shortcut.
	
___
:information_source: The ```-z``` flag will only work for service accounts in the **current** project. If you’re referring to a service account in a different project, use the ```-n <project>```switch.
___

Now that the default Service Account now has view access, so now it can query the API to see what resources are within the Project. This also has the added benefit of suppressing the error message! Although, in reality, we fixed the application.

Another way you could have done the same is by using the OpenShift console. Once you’re on the "User Adminitration -> User Managemenet -> Role Bindings" page, click on Create Binding button.
![Service account list](https://github.com/bhandaru/nationalparks-labs/blob/master/images/rolebindings.png)

![Service account list](https://github.com/bhandaru/nationalparks-labs/blob/master/images/createrolebinding.png)

## Exercise: Redeploy Application

One more step is required. We need to re-deploy the parksmap application because it’s given up trying to query the API.

This time we’ll use the web console. Find your userXY project, and then click "Applications" and then "Deployments". You’ll see your only application, parksmap, listed.

The deployment screen tells you a lot about how the application will be deployed. At the top right, there is a button labeled "Deploy". This button will cause a new deployment (which you know creates a new ReplicationController, right?).

You’ll see that a new deployment is immediately started. Return to the overview page and watch it happen. You might not be fast enough! But it will be in the list of deployments.

If you look at the logs for the application now, you should see no errors. That’s great.

## Exercise: Grant User View Permissions

If you create a project, you are that project’s administrator. This means that you can grant access to other users, too. If you like, give your neighbor view access to your project using the following command:
___
:information_source: In the following command(s), replace userXY with the user name of the person to whom you want to grant access.
___

```
oc policy add-role-to-user view userXY
```

Have them go to the project view by clicking the Projects button and verify that they can see your project and its resources. This type of arrangement (view but not edit) might be ideal for a developer getting visibility into a production application’s project.

<p align="center">
  <a href="/08%20-%20OpenShift%20Logging%20Capabilities.MD">Previous Exercise</a> &nbsp;|
  &nbsp;<a href="/README.md">Table of Contents</a> &nbsp;|
  &nbsp;<a href="/10%20-%20Remote%20Access%20to%20Application.MD">Next Exercise</a>
</p>
