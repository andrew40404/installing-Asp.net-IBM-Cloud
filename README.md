# Installing ASP.NET on IBM Cloud



**Step 1 - provision Kubernetes Cluster**

- Click the **Catalog** button on the top
- Select **Service** from the **Catalog**
- Search for **Kubernetes Service** and click on it

-   -   -

![](asp.net%20ibm%20cloud_html_46d1c04e26ba5eea.png)

-   -   -   -   -   -   -

- You are now at the Kubernetes deployment page. You need to specify some details about the cluster
- Choose a plan **standard** or **free** , the free plan only has one worker node and no subnet, to provision a standard cluster, you will need to upgrade your account to Pay-As-You-Go
- To upgrade to a Pay-As-You-Go account, complete the following steps:
- In the console, go to Manage > Account.
- Select Account settings; and click Add credit card.
- Enter your payment information, click Next, and submit your information
- Choose **classic** or **VPC** , read the docs and choose the most suitable type for yourself

![](asp.net%20ibm%20cloud_html_4d3a968071544952.png)

- Now choose your location settings,
- Choose **Geography** (continent)

![](asp.net%20ibm%20cloud_html_72496e6b0b2c820d.png)

-   Choose 	Single or Multizone, in single zone your data is only kept in on 	datacenter, on the

​      other hand with Multizone it is distributed to multiple zones, thus safer in an unforeseen

​      zone failure

- If you wish to use Multizone please set up your account with[VRF

- If at your current location selection, there is no available Virtual LAN, a new VLAN will be created for you
- Choose a Worker node setup or use the preselected one, set Worker node amount per zone
- Choose **Master Service Endpoint**. In VRF-enabled accounts, you can choose private-only to make your master accessible on the private network or via VPN tunnel. Choose public-only to make your master publicly accessible. When you have a VRF-enabled account, your cluster is set up by default to use both private and public endpoints.
   Give desired **tags** to your cluster, for more information visit tags
- Click **create**
   • Wait for your cluster to be provisioned
   • Your cluster is ready for usage

**Step 2 Deploy IBM Cloud Block Storage plug-in**

The Block Storage plug-in is a persistent, high-performance iSCSI storage that you can add to your apps by using Kubernetes Persistent Volumes (PVs).

- Click the **Catalog** button on the top
- Select **Software** from the catalog
- Search for **IBM Cloud Block Storage plug-in** and click on it
   • On the application page Click in the dot next to the cluster, you wish to use
   • Click on Enter or Select Namespace and choose the default Namespace or use a custom one (if you get error please wait 30 minutes for the cluster to finalize)
- Give a **name** to this workspace
- Click **install** and wait for the deployment

**Step 3 For ASP.NET**

Following should be installed on the machine: 

1. **IBM Cloud account**
2. **IBM CLoud CLI**
3. **Git**
4. **.NET Core 2.1.1 SDK 2.1.301**

1. 1. **Steps:**

   2. 1. ​	**Clone/download the IBM Cloud ASP.NET Core Getting Started Application**

   3. ```sh
      git clone https://github.com/IBM-Cloud/get-started-aspnet-core 
      ```

   4. ###  **Run the ASP.NET Core app locally**

   5. This step is to verify whether your app is running successfully locally before deployment. You can start by verifying the version of dotnet as follows:

   6. dotnet --version

   7. 1. ​	**Next, navigate to your App folder.**

   8. cd get-started-aspnet-core/src/GetStartedDotnet

   9. 1. ​	**Restore the app with the following command:**

2. 

![](asp.net%20ibm%20cloud_html_f2948b6bf4e23894.png)

ii. It will take some time. It is ready to use if you see the
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

