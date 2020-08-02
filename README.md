# Complete CloudGoat Setup Guide #
## Introduction ##
### WARNING: Following this tutorial in its entirety will result in vulnerable AWS cloud resources being deployed publicly on the internet...just kidding ###
---
## AWS Registration and Configuration ##

Navigate to the **[AWS Console](https://aws.amazon.com/console/)** to begin account set-up
Click the **Create an AWS Account** button
![AWS Landing](/images/image_a1.png)

Enter your information to sign up
![AWS Registration](/images/image_a2.png)

Next go to the services page and search for **IAM management** and go there to set up the user account
![IAM Service](/images/image_a3.png)

Select the **Users** page on the left menu and then the **Add user** button
![Users Page](/images/image_a4.png)

Now choose a **user name** for the account that will manage your AWS resources.
![AWS Landing](/images/image_a5.png)

Next, we will be creating a user group to associate the necessary permissions with.
![AWS Landing](/images/image_a6.png)

First give your group a name – we chose **cloudgoat_admin** but it could be anything.
For this application we will need to associate the following policies (you can search for them quickly):
```
AdministratorAccess
AmazonRDSFullAccess
IAMFullAccess
AmazonS3FullAccess
CloudWatchFullAccess
AmazonDynamoDBFullAccess
```
![AWS Landing](/images/image_a7.png)

Finish up by clicking the **Create group** button.
![AWS Landing](/images/image_a8.png)

Now click through the **Next** button until you are finished with the user creation wizard.
### NOTE ### When you get to the final screen make sure to grab the **AccessKey** and **SecretKey**!
![AWS Landing](/images/image_a9.png)

If you missed the **SecretKey** you can create a new key-pair under the user settings and delete the old one.
![AWS Landing](/images/image_a10.png)

Finally, navigate to the services menu and click on S3. If you see the following message it probably means that you need to finish verifying your account info. Note that the final steps of this tutorial (creating the actual instances) will not work until your account is fully registered and verified.
![AWS Landing](/images/image_a11.png)

## Launching the Management VM ##

First, we start by downloading something to run our VM.
Select the host option that is appropriate to you (**windows in this example**).
![VBox download](/images/image_1.png)

Next download the **[Ubuntu Server iso](https://ubuntu.com/download/server)**.
![Ubuntu download](/images/image_2.png)

Click New to start a new **Ubuntu Server** instance.
![VBox Manager menu](/images/image_3.png)

Name your VM and choose the local location to store its filesystem.
![Create Virtual Machine](/images/image_4.png)

2048 MB or 2GB is enough for this application.
![Memory Allocation](/images/image_5.png)

Create a new virtual hard disk to store your filesystem.
![Virtual Filesystem Creation](/images/image_6.png)

**VirtualBox Disk Image** (.vdi) will work fine here.
![VDI Creation](/images/image_7.png)

Choose **dynamically allocated** so you only use the space you have to.
![Allocation](/images/image_8.png)

The default **10GB** configuration is fine for this application.
![Storage Size](/images/image_9.png)

Finally, we are done with the main configuration.
![Complete VM Profile](/images/image_10.png)

Before booting up the image click the **settings** button with the CloudGoat VM selected (highlighted).
Go to the **Storage** tab, select the **Empty** slot on the **IDE Controller**.
Check the box for **Live CD/DVD** and then click the **blue disk** image button to the right of the dropdown.
![Storage Settings](/images/image_11.png)

Next, click on **Choose a disk file…**
![Disk File](/images/image_12.png)

Select the **iso image** that was downloaded from Ubuntu earlier.
Click **Open**, and then **OK** on the **settings** window.
![File Selector](/images/image_13.png)

Now click the **Start** button and follow the prompts to install ubuntu server.

## Installing Ubuntu Server ##
Choose your default language.
![Ubuntu Install Languages](/images/image_14.png)

For most applications, the default DHCP configuration will be sufficient.
![DHCP Configure](/images/image_15.png)

Default proxy configuration unless you have a proxy.
Default Ubuntu archive mirror for downloading packages and updates.
Use the entire drive for installing the filesystem.
Note: **Your server's name** is your **vm's hostname**
![Profile Setup](/images/image_16.png)

Select the option to download OpenSSH, skip all the additional snaps.
You should then see the system being installed.
![Ubuntu Filesystem](/images/image_17.png)

After the installation is complete you can unmount the live-boot iso by clicking **Devices** > **Optical Drives** > **Remove disk from virtual drive**
![Remove Image](/images/image_18.png)

Next, enter your **username** and **password**. If everything was successful, you should be presented with a terminal like this.
![VM Landing](/images/image_19.png)

## Configure Windows SSH ##

The VirtualBox terminal is less than ideal at this point and we should have a more permanent method for connecting to remote Linux hosts, so let’s give Windows an **upgrade**.
Tap the Windows button and type **Apps & Features** to pull up the settings submenu.
Select the **optional features** link.
Then search for OpenSSH server in the list.
After you click on it, it will disappear from the list. Click the back arrow on top of the menu and it should be installing as seen in the picture below.
![OpenSSH Install](/images/image_20.png)

Next press the Windows button again and type **PowerShell**.
![Powershell Start](/images/image_21.png)

Click the top-left menu button and select **properties**
![Powershell Properties](/images/image_22.png)

Adjust the colors, fonts, or whatever else to your liking and click OK.
Now that our shell looks awesome, let’s try connecting to our VM over SSH (secure shell in-case you didn’t know).
```
ssh <username>@<vm’s-ip>
```

## Connecting to the VM via SSH ##

Enter your username and password for the Ubuntu account that you set up when installing the VM.
![Powershell SSH](/images/image_23.png)

A successful login should look something like this (with a bunch of extra info about the server):
![SSH Landing](/images/image_24.png)

## Configuring CloudGoat on the VM ##

Next, we need to grab **[pip](https://bootstrap.pypa.io/get-pip.py)** (command below) and make sure that we can run python easily.
First, we are using wget to download the web resource, then we are creating symbolic links for the python3 binary.
```
wget https://bootstrap.pypa.io/get-pip.py
sudo ln -s /usr/bin/python3 /usr/bin/python
sudo ln -s /usr/bin/python3 /usr/bin/py
```
Now to get pip.
```
py get-pip.py
```
![VM getpip](/images/image_25.png)

Now go to Github and grab the link for the **[CloudGoat repository](https://github.com/RhinoSecurityLabs/cloudgoat)**.
![CG Github](/images/image_26.png)

In the terminal of ubuntu server enter:
```
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git ./cloudgoat
```
Because I don’t like pressing extra keys, I’ll make the cloudgoat directory all lower case.
Now we get the terraform install:
```
wget https://releases.hashicorp.com/terraform/0.12.29/terraform_0.12.29_linux_amd64.zip
```
![Installs](/images/image_27.png)

Grab unzip from apt, unzip terraform and then move it to a location that is configured to your PATH environment variable.
```
sudo apt-get install unzip
unzip terraform_0.12.29_linux_amd64.zip
sudo cp terraform /usr/local/bin/
rm get-pip.py & rm terraform & rm terraform_0.12.29_linux_amd64.zip
```
![Apt Install](/images/image_28.png)

Now that we’ve cleaned up, we can download our required libraries using pip and change the permissions for cloudgoat.py so we can execute it with a single command.
```
cd cloudgoat
pip install -r ./core/python/requirements.txt
chmod u+x cloudgoat.py
```
![Pip Install Requirements](/images/image_29.png)

Once that’s complete, we will need to download the **AWS Cli (command-line interface)** for managing our accounts.
```
curl “https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip” -o “awscliv2.zip”
unzip awscliv2.zip
sudo ./aws/install
rm awscliv2.zip
aws –version
```
![AWS Cli Install](/images/image_30.png)

Configure the CloudGoat profile:
```
./clougoat.py config profile
./cloudgoat.py config whitelist --auto
aws configure --profile cloudgoat_manager
```
Enter the **Access Key**, **Secret Key** and **Default Region** of your admin account that was set up on the **AWS Console** web portal. 
![CloudGoat Profile Configure](/images/image_31.png)

## Creating Instances and Wrapping Up ##
Create the vulnerable instance:
```
./cloudgoat.py create rce_web_app profile cloudgoat_manager
```
![CloudGoat Instace Creation](/images/image_32.png)

When installation is complete you will be provided with a set of credentials for two separate user accounts. Lets configure those accounts with **AWS Cli**.
```
aws configure --profile Lara
aws configure --profile McDuck
```
![AWS Instnace Accounts](/images/image_33.png)

That’s it! You’ve now successfully configured and deployed CloudGoat with the vulnerable RCE ec2 instance.

**Make sure to save all of your keys and credentials in a password manager.**
