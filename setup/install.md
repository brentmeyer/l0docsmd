# Install and Configure Layer0

## Prerequisites

Before you can install and configure Layer0, you must obtain the following:

*  **A "Bring Your Own Operations" ([BYOO](http://wikit/index.php/BYOO)) account.**

	If you do not already have a BYOO account, submit an [instance creation request form](http://goo.gl/forms/KAmRyO1yzU).
	If you are not able to access the form, send an email to [xfra@us.imshealth.com](mailto:xfra@us.imshealth.com). In your email, include your team name and the name of the team owner.

* **A d.ims.io token.**

	This token grants you read access to the IMS private Docker Registry. You can use the same token for multiple Layer0 installations.
	To generate a token, go to [https://d.ims.io/token](https://d.ims.io/token). Log in using your IMS Health domain credentials.

* **An EC2 Key Pair.** This allows SSH access into the EC2 instances running your Services. 
You can use an existing key pair if you've already made one.
Otherwise, create an EC2 key pair in AWS following their [instructions](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair).
Choose any name for the key pair and make note of it. 


## Part 1: Download and extract Layer0

1. In the [Downloads section of the home page](/index.html#download), select the appropriate installation file for your operating system. Extract the zip file to a directory on your computer.
2. Add both the directory that contains the **l0** application, as well as the entire l0-setup directory, to your system path. The l0-setup directory contains files that are necessary for the **l0-setup** application to work properly, so you must add the entire directory to your path. <br />For more information about adding directories to your system path, see the following resources:
	* (Windows) [How to Edit Your System PATH for Easy Command Line Access in Windows](http://www.howtogeek.com/118594/how-to-edit-your-system-path-for-easy-command-line-access/)
	* (Linux/Mac) [Adding a Directory to the Path](http://www.troubleshooters.com/linux/prepostpath.htm)

## Part 2: Create an Access Key
This step will create an Identity & Access Management (IAM) access key from your BYOO instance. You will use the credentials created in this section when installing, updating, or removing Layer0 resources. 

**To create an Access Key:**

1. In a web browser, go to your team's BYOO login page. Log in using your IMS Health domain credentials. 
2. Under **Security and Identity**, click **Identity and Access Management**.
3. Click **Groups**.
4. Click **Create New Group** and enter the name: **Administrators**. Click **Next Step**.
5. Click on **AdministratorAccess** to attach the Administator policy to your new group. Click **Next Step** to review and **Create Group** to create your group.
6. After creating your group, click on **Users**.
7. Click **Create New Users** and enter a unique user name you will use for Layer0. This user name can be used for multiple Layer0 installations. Ensure the box **Generate an Access Key for each user** is checked, and click **Create**.
8. After your user is created successfully, you'll be presented with new user security credentials. Click **Download Credentials**, which will save your new access key to a CSV file. Once your credentials have been downloaded, click **Close**.
9. Click **Create New Users** and enter a unique user name you will use for Layer0. This user name can be used for multiple Layer0 installations. Ensure the box **Generate an Access Key for each user** is checked, and click **Create**.
10. Find your newly created user in the Users list, and select it. Under **User Actions**, select **Add User to Groups**.
11. Select the group **Administrators** and click **Add to Groups**. This will make your newly created user an administrator for your BYOO account, so be sure to keep your security credentials safe!

## Part 3: Configure your Layer0
Now that you have downloaded Layer0 and configured your AWS instance, you can create your Layer0.

**To configure Layer0:**
 
1. At a command prompt, navigate to the **l0-setup** subdirectory in the folder in which you extracted the Layer0 files.
2. Type the following command, replacing *prefix* with a unique name for your Layer0:
<ul>
  <li class="command">**l0-setup apply** *prefix*</li>
</ul>
3. When prompted, enter the following information:
	* **AWS Access Key ID**: The access key ID contained in the credential file that you downloaded in step 5 of the previous section.
	* **AWS Secret Access Key**: The secret access key contained in the credential file that you downloaded in step 5 of the previous section.
	* **d.ims.io Token**: The Docker token that you created in the Prerequisites section.
	* **Key Pair**: The name of the key pair that you created in the Prerequisites section.

The first time you run the **apply** command, it may take around 15 minutes to complete. If the **apply** command fails to complete successfully, it is safe to run it again until it succeeds.

## Part 4: Configure environment variables
Once the **apply** command has run successfully, you can configure the Layer0 environment variables using the **endpoint** command.

To view the environment variables for your Layer0, type the following command, replacing *prefix* with the Layer0 prefix you created in Part 2: 
<ul>
  <li class="command">**l0-setup endpoint --insecure** *prefix*</li>
</ul>

Alternatively, you can view the environment variables and apply them to your shell using a single command:

<!--Uncomment this for 0.6.4-->
<!--* (Windows Command Prompt): **l0-setup.exe endpoint --insecure --syntax cmd** _prefix_ **> tmp.cmd && call tmp.cmd && del tmp.cmd** (Note: This command will only work in Layer0 version 0.6.4 or later)-->
* (Windows PowerShell): **l0-setup endpoint --insecure --powershell** *prefix* **| Out-String | Invoke-Expression**
* (Linux/Mac): **eval "$(l0-setup endpoint --insecure** _prefix_**)"**

## Part 5: Configure DNS Endpoint

!!! note
	The procedures in this section are optional, but are highly recommended for production use.

Layer0 is configured to use SSL with the imshealthlabs.net domain. Follow the procedures in this section to associate a subdomain of imshealthlabs.net with your Layer0 endpoint.

**To associate a subdomain with your Layer0 endpoint:**

1. Contact the [DevOps team](mailto:nexxusops@imshealth.com) and request a user account on the DevOps AWS account.
2. Once you have received the DevOps AWS account credentials, sign in to the [AWS Console](https://imsappature-dev.signin.aws.amazon.com/console).
3. Under **Networking**, click **Route 53**.
4. In the menu on the left side of the screen, click **Hosted zones**.
5. In the list of domains, click **imshealthlabs.net**.
6. Click **Create Record Set**.
7. In the **Create Record Set** panel on the right side of the screen, configure the record set:
	* In the **Name** field, type the subdomain. The subdomain name you specify must be unique.
	* From the **Type** menu, select **CNAME - Canonical name**.
	* In the **Value** field, enter the endpoint URL for your Layer0. You can find the endpoint for your Layer0 by using the **endpoint** command; see part 4, above, for more information.
	* Click **Create**.

