Installing ASP.NET on IBM Cloud {.western lang="en-IN" style="margin-top: 0cm; margin-bottom: 0.26cm; background: #ffffff"}
===============================

\
 \

**Step 1 - provision Kubernetes Cluster**

-   -   -   

![](asp.net%20ibm%20cloud_html_46d1c04e26ba5eea.png)

-   -   -   -   -   -   -   

![](asp.net%20ibm%20cloud_html_4d3a968071544952.png)

-   -   

![](asp.net%20ibm%20cloud_html_72496e6b0b2c820d.png)

-   

other hand with Multizone it is distributed to multiple zones, thus
safer in an unforeseen

zone failure

-   

-   -   -   -   

**Step 2 Deploy IBM Cloud Block Storage plug-in**

The Block Storage plug-in is a persistent, high-performance iSCSI
storage that you can add to your apps by using Kubernetes Persistent
Volumes (PVs).

-   -   -   -   -   

**Step 3 For ASP.NET**

Following should be installed on the machine:

1.  2.  3.  4.  

\
 \

**Steps:** {.western style="margin-top: 0.11cm; margin-bottom: 0.11cm; line-height: 200%; background: #ffffff"}
----------

1.  

\

git clone https://github.com/IBM-Cloud/get-started-aspnet-core

### \
 {.western style="letter-spacing: 0.1pt; background: #ffffff"}

2.  

This step is to verify whether your app is running successfully locally
before deployment. You can start by verifying the version of dotnet as
follows:

dotnet --version

3.  

cd get-started-aspnet-core/src/GetStartedDotnet

4.  

dotnet restore

This uses NuGet to restore dependencies and project-specific tools that
are specified in the project file. By default, the restoration of
dependencies and tools are executed in parallel. For more info
visit [Docs](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-restore?tabs=netcore2x).

Now, run your application with the following command:

dotnet run

The application starts listening on port 5000. You will see the
following message.

...

Now listening on: http://localhost:5000

Application started. Press Ctrl+C to shut down.

5.  

To pack the application and its dependencies, create a new folder
named publish for deployment to a hosting system for execution. We have
to use dotnet to publish. Then it’s ready to run anywhere.

Publish the app to get a self-contained DLL using the dotnet
publish command.

dotnet publish -c Release

Running publish displays some messages with a successfully published DLL
at the end of the process. For our example, you can see the following
message.

...

GetStartedDotnet -\>
/home/get-started-aspnet-core/src/GetStartedDotnet/bin/Release/netcoreapp2.0/GetStartedDotnet.dll

6.  

Once the application is ready, we can make an image and put it inside a
container. We need a file that contains step-by-step instructions to
deploy the image inside the container to run our application anywhere.
This Dockerfile is a basic file and you may only require a few lines to
get started with your own image.

Go to the app folder (here GetStartedDotnet) and create a Dockerfile to
define the Docker image.

cd get-started-aspnet-core/src/GetStartedDotnet

Add the contents
of [Dockerfile](https://github.ibm.com/Nidhi-N-Shah/ASP.NET-CORE-App-Deployment-in-IKS/blob/master/Dockerfile) by
using your favorite editor (vim, nano, etc.) and save the file.

vi Dockerfile

FROM microsoft/aspnetcore:2.0

\

WORKDIR /app1

\

COPY ./bin/Release/netcoreapp2.0/publish .

\

ENTRYPOINT ["dotnet", "GetStartedDotnet.dll"]

The first line we added FROM microsoft/aspnetcore:2.0 will download
the aspnetcore image from the hub repository, so it actually contains
the .NET Core and you don’t need to put it inside the image. You can
find more repositories and versions in the Docker hub.

The Dockerfile line WORKDIR /app sets our working directory in the app
folder, which is inside the container that we’re building.

Now we need to copy the contents of the publish folder into the app
folder on the image COPY ./bin/Release/netcoreapp2.0/publish.

**Note:** You can find this path in the output when you run dotnet
publish -c Release.

The last line in the Dockerfile is the ENTRYPOINT statement: ENTRYPOINT
["dotnet", "GetStartedDotnet.dll"]. This line tells Docker that it
should run the dotnet command with GetStartedDotnet.dll as parameter.

7.  

You can test your dockerized app by following the steps below. This
section is optional for this tutorial, though.

First, build an image.

docker build -t get-started-aspnet

**Note:** You can choose any name for your app.

Running the build command displays the following message in the end.

Successfully tagged get-started-aspnet:latest

The following command will run an app.

docker run -d -p 8080:80 --name app get-started-aspnet

Navigate to http://localhost:8080 to access your app in a web browser.

Clean up with the following commands.

docker stop /app

docker rm /app

These above commands stop and remove the Docker container of your app,
respectively. You can use them to remove your container if you no longer
need it.

8.  

We are now ready to create our [Kubernetes
cluster](https://cloud.ibm.com/containers-kubernetes/clusters).

1.  2.  

or

ibmcloud login --sso

3.  

​i. Create a [Kubernetes
cluster](https://cloud.ibm.com/containers-kubernetes/overview) by
choosing **Cluster Type – Free**. Give a unique name to the cluster and
click **Create Cluster**.

Note: For more details, see [Creating a Kubernetes cluster in IBM
Cloud](https://cloud.ibm.com/docs/containers/container_index.html#clusters).

![](asp.net%20ibm%20cloud_html_f2948b6bf4e23894.png)

​ii. It will take some time. It is ready to use if you see the
following:

![](asp.net%20ibm%20cloud_html_fcca8f1195b0b5d4.png)

4.  5.  6.  7.  8.  

Show more

9.  

1.  

To create an IBM Cloud Cloudant Database, create a
new [Cloudant](https://cloud.ibm.com/catalog/services/cloudant) database
instance. Select **Use both legacy credentials and
IAM** under **Available authentication methods**.

![](asp.net%20ibm%20cloud_html_b1c1d061f909a41e.jpg)

2.  

![](asp.net%20ibm%20cloud_html_b936ac0ca3a95e3b.jpg)

3.  

\

4.  

For example:

kubectl create secret generic cloudant
--from-literal=url=https://username:passw0rdf@username-bluemix.cloudant.com

5.  6.  

This will display all the secrets you created in their respective
clusters.

10. 

The [IBM Cloud Container
Registry](https://cloud.ibm.com/kubernetes/catalog/registry) provides a
multi-tenant private image registry that you can use to safely store and
share your Docker images with users in your IBM Cloud account.

1.  

ibmcloud cr login

\

2.  

ibmcloud cr namespaces

\

3.  

ibmcloud cr namespace-add \<name\>

For example:

ibmcloud cr namespace-add aspnetapp-01

4.  

ibmcloud cr info

For example: registry.ng.bluemix.net

5.  

docker build . -t \<REGISTRY\>/\<NAMESPACE\>/myapp:v1.0.0

For example:

docker build . -t registry.ng.bluemix.net/aspnetapp-01/myapp:v1.0.0

It will display the following message in the end.

...

Successfully tagged registry.ng.bluemix.net/aspnetapp-01/app:v1.0.0

6.  

docker push \<REGISTRY\>/\<NAMESPACE\>/myapp:v1.0.0

7.  

ibmcloud cr image-list

You have set up a namespace in the IBM Cloud Container Registry and
pushed a Docker image to your namespace.

11. 

Once you have a running Kubernetes cluster, you can deploy your
containerized application on top of it. To do so, you create a
Kubernetes Deployment configuration. The Deployment instructs Kubernetes
on how to create and update instances of your application. Once you
create a Deployment, the Kubernetes master schedules the mentioned
application instances onto individual Nodes in the cluster. A Kubernetes
Deployment Controller continuously monitors those instances that were
created. If the Node that’s hosting an instance goes down or is deleted,
the Deployment controller replaces it. This provides a self-healing
mechanism to address machine failure or maintenance.

1.  2.  

vi deployment.yaml

\

\# Update \<REGISTRY\> \<NAMESPACE\> values before use

\# Replace app name instead of get-started-aspnet if you wish to use
different name for your app

\

apiVersion: apps/v1

kind: Deployment

metadata:

name: get-started-aspnet

labels:

app: get-started-aspnet

spec:

replicas: 2

selector:

matchLabels:

app: get-started-aspnet

template:

metadata:

labels:

app: get-started-aspnet

spec:

containers:

- name: get-started-aspnet

image: \<REGISTRY\>/\<NAMESPACE\>/myapp:v1.0.0

ports:

- containerPort: 8080

imagePullPolicy: Always

env:

- name: CLOUDANT\_URL

valueFrom:

secretKeyRef:

name: cloudant

key: url

optional: true

The deployment get-started-aspnet was created, indicated by
the .metadata.name field. The Deployment creates two replicated Pods,
indicated by the replicas field. These replicas are needed to handle the
traffic in deployment. You can keep it to 1 as well. The selector field
defines how the Deployment finds which Pods to manage. However, more
sophisticated selection rules are possible, as long as the Pod template
itself satisfies the rule. The Pods labeled app: get-started-aspnet are
using the labels field. The Pod template’s specification,
or .template.spec field, indicates that the Pods run one
container, get-started-aspnet, which runs
the \<REGISTRY\>/\<NAMESPACE\>/myapp:v1.0.0 Docker image. Open port 8080
so that the container can send and accept traffic. Set
the imagePullPolicy of the container to Always. The Secret information
has been updated in the env field, like the CLOUDANT\_URL that we
mentioned while creating our Secret for the Cloudant database.

3.  4.  

The output will display, similar to the following message.

deployment "get-started-aspnet" created

5.  

Use the NodePort 8080 to expose the deployment.

kubectl expose deployment get-started-aspnet --type NodePort --port 8080
--target-port 8080

You will see the following message.

service "get-started-aspnet" exposed

12. 

To verify that your application is running successfully, you need to
check the STATUS of your pod. It should be in a state of Running:

kubectl get pods -l app=get-started-aspnet

It will appear like the following:

NAME READY STATUS RESTARTS AGE

get-started-aspnet-68d6dc5c4-2trcl 1/1 Running 0 1m

get-started-aspnet-68d6dc5c4-qdbkt 1/1 Running 0 1m

It will show two instances as we have set two replicas in our
deployment.

For access of ASP.Net Core application:

1.  2.  3.  

In this way, deployment and access of application in the IKS environment
can be done.

13. 

Following command will be used to clean up the sample application:

kubectl delete deployment,service -l app=get-started-aspnet

kubectl delete secret cloudant

\
 \

