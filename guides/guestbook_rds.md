# Deployment guide: Guestbook with RDS Database
The Guestbook application that you deployed in the [Guestbook deployment guide](/guides/guestbook) was a very simple application that stored its data in memory (also known as a "stateful" application). If you were to re-deploy the Guestbook service, all of the data previously entered into the application would be lost permanently.

A stateless application, on the other hand, does not record data generated in one session for use in subsequent sessions. In order to prevent data loss, web applications that you deploy using Layer0 should be stateless.

This guide will show you how to make a stateless Guestbook application that stores data in an Amazon Relational Database Service (RDS) database.

---

## Before you start
In order to complete the procedures in this section, you must install and configure Layer0 v0.7.0 or later. If you have not already configured Layer0, see the [installation guide](/setup/install). If you are running an older version of Layer0, see the [upgrade instructions](/setup/update#upgrading-older-versions-of-layer0).

This guide expands upon the [Guestbook deployment guide](/guides/guestbook) deployment guide. You must complete the procedures in that guide before you can complete the procedures listed here. After completing the procedures in the Guestbook guide, your Layer0 should contain a service named "guestbooksvc", running a deploy named "guestbook", behind a load balancer named "guestbooklb", all within an environment named "demo".

## Part 1: Create an RDS instance
The updated Guestbook application described in this guide stores its data in an Amazon Relational Database Service (RDS) database. These steps describe the process of creating a new RDS instance for this purpose.

**To create a new RDS database:**

1. Download the [RDS Cloudformation script](https://gitlab.imshealth.com/xfra/layer0-samples/blob/master/rds/cloudformation.json) and save it to your computer.
2. Go to [the AWS Console page for Layer0](https://imshealth.auth0.com/samlp/dD95hSDud43w6tu0KiQTanigF2FzJccA). Log in using your IMS Health domain credentials.
3. Under **Management Tools**, click **CloudFormation**.
4. Click **Create Stack**.
5. Next to **Choose a Template**, select **Upload a template to Amazon S3**. Click the **Browse** button. On the file selection window, select the file that you downloaded in step 1 (**cloudformation.json**), and then click **Next**.
6. Under **Parameters**, enter the following details:
    * **Stack Name**: _Layer0Prefix_**-guestbook-rds**
    * **Database**: **guestbook**
    * **EnvironmentSG**: Select the item that starts with _Layer0Prefix_**-demo**
    * **Password**: a password that you specify. The password must contain at least 8 characters. It cannot contain spaces or any of the following characters: /, @, "
    * **Port**: 3306
    * **Username**: a username that you specify.
    * **Subnets**: Select the items that end with **(layer0-**_Layer0Prefix_**-priA)** and **(layer0-**_Layer0Prefix_**-priB)**
    * **VPC**: Select the item that ends with **(layer0-**_Layer0Prefix_**-vpc)**
7. Click **Next** until you arrive at the Review page. Click **Create**. The process of creating the stack requires about 10 minutes to complete; wait until this process is complete before proceeding to the next section.

## Part 2: Create a deploy
After the RDS database is ready, you can create a new deploy for the service.

**To create a new deploy:**

1. Download the [RDS Guestbook Task Definition](https://gitlab.imshealth.com/xfra/layer0-samples/blob/master/rds/Dockerrun.aws.json) and save it to your computer as **GuestbookRDS.Dockerrun.aws.json**.
2. In the AWS Console, under **Management Tools**, click **CloudFormation**, and then click the name of the stack you created in the previous section. Click **Outputs** to see a list of the variables you specified in the previous section.
3. In a text editor, open the GuestbookRDS.Dockerrun.aws.json file. This file uses environment variables for configuration. Edit the file to use the appropriate values for your stack. When you are finished, the **environment** section should resemble the following example:
<pre>
...
"environment": [
	{
		"name": "GB_DB_NAME",
		"value": "guestbook"
   	},
   	{
        "name": "GB_DB_HOST",
        "value": "guestbook.abc123.us-west-2.rds.amazonaws.com"
   	},
   	{
        "name": "GB_DB_PORT",
        "value": "3306"
   	},
   	{
    	"name": "GB_DB_USER",
		"value": "admin"
   	},
   	{
        "name": "GB_DB_PASSWORD",
    	"value": "password"
	}
]
...
</pre>
4. Save the changes you made to **GuestbookRDS.Dockerrun.aws.json**.
5. At the command prompt, run the following command:

<span style="padding-left:2em">**l0 deploy create GuestbookRDS.Dockerrun.aws.json guestbook**</span>

You will see the following output:
```
DEPLOY ID   	DEPLOY NAME  VERSION
1guestbook:2    guestbook    2
```
The number 2 in the **Version** column indicates that this is the second iteration of this deploy (the deploy from the previous guestbook example was the first iteration).

## Part 3: Apply the deploy
At this point, you can use the **deploy** command to deploy the latest version of the deploy named "guestbook" to our existing service ("guestbooksvc"). Using this command will instruct the service to run the new version of the Guestbook container, along with the environment variables you specified in the task definition in the previous section.

To apply the deploy, run the following command:

<span style="padding-left:2em">**l0 deploy apply guestbook:2 demo:guestbooksvc**</span>

You will see the following output:
```
Successfully Applied Deploy
```

## Part 4: Test the application

### Check the status of the service

After you create a service, it may take several minutes for that service to completely finish deploying. You can check the status of a service using the **service get** command.

To check the status of the guestbook service, run the following command:
<ul>
  <li class="command">**l0 service get demo:guestbooksvc**</li>
</ul><br />
Initially, you will see an asterisk (\*) next to the name of the "guestbook:1" deploy; this indicates that the service is in a transitional state. In this phase, if you execute the **service get** command again, you will see the following output:
```
SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER		DEPLOYS       SCALE
1guestbooksvc   guestbook     demo         guestbooklb   	guestbook:2*  1/1
                                                     		guestbook:1
```

In the next phase of the deployment, you will see "(1)" in the Scale column; this indicates that 1 copy of the service is transitioning to an active state. In this phase, if you execute the **service get** command again, you will see the following output:
```
SERVICE ID		SERVICE NAME  ENVIRONMENT  LOADBALANCER		DEPLOYS       SCALE
1guestbooksvc   guestbook     demo         guestbooklb   	guestbook:2*  1/1 (1)
                                                     		guestbook:1
```

The system will wait until the guestbook:2 deploy is running before it stops the currently-running guestbook:1 deploy. In this phase, if you execute the **service get** command again, you will see the following output:
```
SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER		DEPLOYS       SCALE
1guestbooksvc   guestbook     demo         guestbooklb  	guestbook:2   2/1
                                                     		guestbook:1
```

Once the guestbook:2 deploy is running, the guestbook:1 deploy will be stopped.
The Deploys and Scale columns will indicate that only one deploy (the guestbook:2 deploy) is running. In this phase, if you execute the **service get** command again, you will see the following output:
```
SERVICE ID  	SERVICE NAME  ENVIRONMENT  LOADBALANCER 	DEPLOYS      SCALE
1guestbooksvc   guestbook     demo         guestbooklb		guestbook:2  1/1
```

### Get the application URL
Once the service has been completely deployed, you can obtain the URL for your application and launch it in a browser.

**To test the RDS guestbook application:**

<ol>
  <li>At the command line, type the following command:
    <ul>
      <li class="command"><strong>l0 loadbalancer get demo:guestbooklb</strong></li>
    </ul><br />
  You will see the following output:
<pre class="code"><code>LOADBALANCER ID  LOADBALANCER NAME  ENVIRONMENT  SERVICES            PORTS       PUBLIC  URL
1guestbooklb     guestbooklb        demo         1guestbooksvc99706  80:80/HTTP  true    <em>(url)</em>
</code></pre>
  </li>
  <li>Copy the value shown in the <strong>URL</strong> column and paste it into a web browser. The RDS-backed guestbook application will appear.</li>
</ol>
