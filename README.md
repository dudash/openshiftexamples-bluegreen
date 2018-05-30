# OpenShift Examples - Blue/Green Deployments
A blue/green deployment is a software deployment strategy that relies on two identical production configurations that alternate between active and inactive. The idea is that you can minimize the downtime of your app during the release process and reduce the chance for failures. If you want to read more, see [martinfowler.com](2). OpenShift has platform provided capabilities for doing advanced software deployment strategies including blue/green.

This git repo contains an intentionally simple example of how you could do a blue/green deployment. You might have already seen some other OpenShift deployment strategies use the deployment configuration features. But this particular example uses the router features in order to patch a specific route to the application.

Here's what it looks like:

![Screenshot](./.screens/bluegreen.gif)

###### :information_source: This example is based on OpenShift Container Platform version 3.9.  It should work with older versions but has not been tested.


## Why blue/green?
Some commonly referenced reasons for leveraging blue/green deployments are:
* Reduce down-time
* More predictable release results
* “Prime” your app before release
* Can quickly rollback if you experience problems in the new release


## How to run this?
First off, you need access to an OpenShift cluster. Don't have an OpenShift cluster? That's OK, download the CDK for free here: https://developers.redhat.com/products/cdk/overview/.

Once you're logged into the cluster with oc, run the following commands to create your webapps:
> `oc new-app openshift/deployment-example:v1 --name=example-blue`

> `oc new-app openshift/deployment-example:v2 --name=example-green`

Create a route to the blue service, which is version 1
> `oc expose svc/example-blue --name=bluegreen-example`

Now you have a running webapp and incoming users are pointed to the blue service.  And you also have a green version 2 of that app which hasn't been exposed to the world yet. The idea is that green will go through testing and get vetted for release. Once that process has occured, you can flip the route from blue to green with the following command:
> `oc patch route/bluegreen-example -p '{"spec":{"to":{"name":"example-green"}}}'`

And now the router will be sending incoming traffic to the green service.


## About the code / software architecture
The parts in action here are:
* An example containerized web app
* Key platform components that enable this example
  - container replication and service layer
	- integrated router (HAProxy)
	- oc CLI tool


## References and other links to check out
* http://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/
* https://martinfowler.com/bliki/BlueGreenDeployment.html
* https://docs.openshift.com/container-platform/3.9/dev_guide/deployments/advanced_deployment_strategies.html#advanced-deployment-strategies-blue-green-deployments
* https://www.quora.com/What-is-blue-green-deployment


## Challenges for Blue/Green deployments (all [borrowed from Christian Posta](1))
* Long running transactions in the green environment. When you switch over to blue, you have to gracefully handle those outstanding transactions as well as the new ones. This also can become troublesome if your DB backends cannot handle this (see below)
* Enterprise deployments are not typically amenable to “microservice” style deployments – that is, you may have a hybrid of microservice style apps, and some traditional, difficult-to-change-apps working together. Coordinating between the two for a blue-green deployment can still lead to downtime
* Database migrations can get really tricky and would have to be migrated/rolledback alongside the app deployments. There are good tools and techniques for doing this, but in an environment with traditional RDBMS, NoSQL, and file-system backed DBs, these things really need to be thought through ahead of time; blindly saying you’re doing Blue Green deployments doesn’t help anything – actually could hurt.
* You need to have the infrastructure to do this
* If you try to do this on non-isolated infrastructure (VMs, Docker, etc), you run the risk of destroying your blue AND green environments


## License
Under the terms of the MIT.

[1]: http://blog.christianposta.com/deploy/blue-green-deployments-a-b-testing-and-canary-releases/
[2]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[3]: https://docs.openshift.com/container-platform/3.9/dev_guide/deployments/advanced_deployment_strategies.html#advanced-deployment-strategies-blue-green-deployments
[4]: https://www.quora.com/What-is-blue-green-deployment
