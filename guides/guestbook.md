# Deployment guide: Guestbook sample application
In this example, you will learn how different Layer0 commands work together to deploy web applications to the cloud. The sample application in this guide is a guestbook&mdash;a web application that acts as a simple message board.

## Before you start
In order to complete the procedures in this section, you must install and configure Layer0 v0.7.2 or later. If you have not already configured Layer0, see the [installation guide](/setup/install). If you are running an older version of Layer0, see the [upgrade instructions](/setup/update#upgrading-older-versions-of-layer0).

Once Layer0 is configured on your computer, download the [Guestbook Task Definition](https://gitlab.imshealth.com/xfra/layer0-samples/blob/master/guestbook/app/Dockerrun.aws.json); save the resulting file as "Guestbook.Dockerrun.aws.json". The instructions in this section assume that this file is located in the directory in which Layer0 is installed; if you place the file elsewhere, you will need to provide the complete path to the file when you reference it in [Part 4](#part-4-create-the-service).

---

## Part 1: Create the environment

The first step in deploying an application in Layer0 is to create an environment.
An environment is a dedicated space in which one or more services can reside. In this example, you will create an environment named "demo".

**To create the environment**
<ul>
  <li>At the command prompt, run the following command to create a new environment named **demo**:
    <ul>
      <li class="command"><strong>l0 environment create demo</strong></li>
    </ul>
  <br />You will see the following output:
<pre class="code"><code>ENVIRONMENT ID  ENVIRONMENT NAME  SERVICE ID
1demo           demo</code></pre>
  </li>
</ul>

## Part 2: Create the load balancer

In order to expose a web application to the public internet, you must create a load balancer. A load balancer listens for web traffic at a specific address, and directs that traffic to a Layer0 service.

By default, Layer0 load balancers listen for web traffic on port 80 and forward it on port 80 using the TCP protocol. You can modify the way in which ports are forwarded, as well as the protocol used, using the **--port** option.

In this example, you will create a new load balancer called "guestbooklb" in the environment named "demo". The load balancer will listen on port 80, and forward traffic to port 80 in the Docker container using the HTTP protocol.

**To create the load balancer:**
<ul>
  <li>At the command prompt, run the following command to create a load balancer named **guestbooklb** that forwards traffic on port 80 using the HTTP protocol:
    <ul>
      <li class="command">**l0 loadbalancer create --port 80:80/http demo guestbooklb**</li>
    </ul>
  <br />You will see the following output:
<pre class="code"><code>LOADBALANCER ID  LOADBALANCER NAME  ENVIRONMENT  SERVICES  PORTS       PUBLIC  URL
1guestbooklb     guestbooklb        demo                   80:80/http  true</code></pre>

  </li>
</ul>

The following is an summary of the arguments passed in the above command:

* **loadbalancer create**: creates a new load balancer
* **--port 80:80/http**: instructs the load balancer to forward requests from port 80 on the server to port 80 in the Docker container using the HTTP protocol
* **demo**: the name of the environment in which you are creating the load balancer
* **guestbooklb**: a name for the load balancer itself

## Part 3: Deploy the Docker task definition

The **deploy** command is used to specify the Docker task definition that refers to a web application. In this section, you will create a new deploy called "guestbook" that refers to the Guestbook.Dockerrun.aws.json file you created earlier.

**To deploy the Guestbook task definition:**
<ul>
  <li>At the command prompt, run the following command:
    <ul>
      <li class="command">**l0 deploy create Guestbook.Dockerrun.aws.json guestbook**</li>
    </ul>
  <br />You will see the following output:
<pre class="code"><code>DEPLOY ID       DEPLOY NAME  VERSION
1guestbook:1    guestbook    1</code></pre>
  </li>
</ul>

The following is an summary of the arguments passed in the above command:

* **deploy create**: creates a new deployment and allows you to specify a Docker task definition
* **Guestbook.Dockerrun.aws.json**: the file name of the Docker task definition (use the full path of the file if it is not in the same directory as l0-setup)
* **guestbook**: a name for the deploy, which you will use later when you create the service

The Deploy Name and Version are combined to create a unique identifier for a deploy.
If you create additional deploys named "guestbook,"  they will be assigned different version numbers.

## Part 4: Create the service
The final part of the deployment process involves using the **service create** command to create a new service and associate it with the environment, load balancer and deployment you created in the previous sections. The service will execute the containers described in the deployment. In this example, you will create a new service called "guestbooksrv".

**To create the service:**
<ul>
  <li>At the command prompt, run the following command:
    <ul>
      <li class="command">**l0 service create --loadbalancer demo:guestbooklb demo guestbooksvc guestbook:latest**</li>
    </ul>
  <br />You will see the following output:
<pre class="code"><code>SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER  DEPLOYS      SCALE
1guestbooksvc   guestbooksvc  demo         guestbooklb   guestbook:1  0/1</code></pre>
  </li>
</ul>

The following is an summary of the arguments passed in the above command:

* **service create**: creates a new service
* **--loadbalancer demo:guestbooklb**: the fully-qualified name of the load balancer; in this case, the load balancer named "guestbooklb" in the environment named "demo." It is not strictly necessary to use the fully qualified name of the load balancer, unless another load balancer with exactly the same name exists in a different environment.
* **demo**: the name of the environment you created in [Part 1](#part-1-create-the-environment)
* **guestbooksvc**: a name for the service you are creating
* **guestbook**: the name of the deploy that you created in [Part 3](#part-3-deploy-the-docker-task-definition)

## Part 5: Test the application

### Check the status of the service

After you create a service, it may take several minutes for that service to completely finish deploying. You can check the status of a service using the **service get** command.

**To check the status:**
<ul>
  <li>At the command prompt, type the following command to check the status of the "guestbooksvc" deploy:
    <ul>
      <li class="command">**l0 service get demo:guestbooksvc**</li>
    </ul></li>
</ul>

Initially, you will see an asterisk (\*) next to the name of the "guestbook:1" deploy; this indicates that the service is in a transitional state. In this phase, if you execute the **service get** command again, you will see the following output:
```
SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER  DEPLOYS       SCALE
1guestbooksvc   guestbooksvc  demo         guestbooklb   guestbook:1*  0/1
```

In the next phase of the deployment, you will see "(1)" in the Scale column; this indicates that 1 copy of the service is transitioning to an active state. In this phase, if you execute the **service get** command again, you will see the following output:

```
SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER  DEPLOYS      SCALE
1guestbooksvc   guestbooksvc  demo         guestbooklb   guestbook:1  1/1
```

### Get the application URL
Once the service has been completely deployed, you can obtain the URL for your application and launch it in a browser.

**To test your web application:**
<ol>
  <li>At the command prompt, type the following command:
    <ul>
      <li class="command">**l0 loadbalancer get demo:guestbooklb**</li>
    </ul><br />
  You will see the following output:
  <pre class="code"><code>LOADBALANCER ID  LOADBALANCER NAME  ENVIRONMENT  SERVICES            PORTS       PUBLIC  URL
1guestbooklb     guestbooklb        demo         1guestbooksvc99706  80:80/HTTP  true    &lt;url&gt;</code></pre></li>
  <li>Copy the value shown in the **URL** column and paste it into a web browser. The guestbook application will appear.</li>
</ol>
