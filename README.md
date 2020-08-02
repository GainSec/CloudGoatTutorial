# Complete CloudGoat Setup Guide #
## Introduction ##
### WARNING: Following this tutorial in its entirety will result in vulnerable AWS cloud resources being deployed publicly on the internet...just kidding ###

## Launching the Management VM ##
First, we start by downloading something to run our VM.
![VBox download](/images/image_1.png)
Select the host option that is appropriate to you (windows in this example).
Next download the Ubuntu Server iso.
 
Click New to start a new Ubuntu server instance.
 
Name your VM and choose the local location to store its filesystem.
 
2048 MB or 2GB is enough for this application.
 
Create a new virtual hard disk to store your filesystem.
 
VirtualBox Disk Image will work fine here.
 
Choose dynamically allocated so you only use the space you have to.
 
The default 10GB configuration is fine for this application.
 
Finally, we are done with the main configuration.
 
Before booting up the image click the settings button with the CloudGoat VM selected (highlighted).
Go to the **Storage** tab, select the **Empty** slot on the **IDE Controller**.
Check the box for **Live CD/DVD** and then click the **blue disk** image button to the right of the dropdown.
 
Next, click on **Choose a disk file…**
 
Select the **iso image** that was downloaded from Ubuntu earlier.
Click **Open**, and then **OK** on the **settings** window.
 
Now click the **Start** button and follow the prompts to install ubuntu server.
 
For most applications, the default DHCP configuration will be sufficient.
 
Default proxy configuration unless you have a proxy.
Default Ubuntu archive mirror for downloading packages and updates.
Use the entire drive for installing the filesystem.
 	
Select the option to download OpenSSH, skip all the additional snaps.
You should then see the system being installed.
 
After the installation is complete you can unmount the live-boot iso by clicking Devices > Optical Drives > **Remove disk from virtual drive**
 
Next, enter your **username** and **password**. If everything was successful, you should be presented with a terminal like this.
 
The VirtualBox terminal is less than ideal at this point and we should have a more permanent method for connecting to remote Linux hosts, so let’s give Windows and upgrade.
Tap the Windows button and type **Apps & Features** to pull up the settings submenu.
Select the **optional features** link.
Then search for OpenSSH server in the list.
After you click on it, it will disappear from the list. Click the back arrow on top of the menu and it should be installing as seen in the picture below.
 
Next press the Windows button again and type **PowerShell**.
 
Click the top-left menu button and select **properties**
 
Adjust the colors, fonts, or whatever else to your liking and click OK.
Now that our shell looks awesome, let’s try connecting to our VM over SSH (secure shell in-case you didn’t know).
```
ssh <username>@<vm’s-ip>
```
 
Enter your username and password for the Ubuntu account that you set up when installing the VM. A successful login should look something like this (with a bunch of extra info about the server):
 
Next, we need to grab pip and make sure that we can run python easily.
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
 

Now go to Github and grab the link for the CloudGoat repository.
https://github.com/RhinoSecurityLabs/cloudgoat
 
In the terminal of ubuntu server enter:
```
git clone https://github.com/RhinoSecurityLabs/cloudgoat.git ./cloudgoat
```
Because I don’t like pressing extra keys, I’ll make the cloudgoat directory all lower case.
Now we get the terraform install:
```
wget https://releases.hashicorp.com/terraform/0.12.29/terraform_0.12.29_linux_amd64.zip
```
 
Grab unzip from apt, unzip terraform and then move it to a location that is configured to your PATH environment variable.
```
sudo apt-get install unzip
unzip terraform_0.12.29_linux_amd64.zip
sudo cp terraform /usr/local/bin/
rm get-pip.py & rm terraform & rm terraform_0.12.29_linux_amd64.zip
```
 
Now that we’ve cleaned up, we can download our required libraries using pip and change the permissions for cloudgoat.py so we can execute it with a single command.
```
cd cloudgoat
pip install -r ./core/python/requirements.txt
chmod u+x cloudgoat.py
```

 
Once that’s complete, we will need to download the AWS Cli (command-line interface) for managing our accounts.
```
curl “https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip” -o “awscliv2.zip”
unzip awscliv2.zip
sudo ./aws/install
rm awscliv2.zip
aws –version
```
 

Configure the CloudGoat profile:
```
./clougoat.py config profile
./cloudgoat.py config whitelist --auto
aws configure --profile cloudgoat_manager
```
Enter the **Access Key**, **Secret Key** and **Default Region** of your admin account that was set up on the **AWS Console** web portal. 



 
Create the vulnerable instance:
```
./cloudgoat.py create rce_web_app profile cloudgoat_manager
```
 
When installation is complete you will be provided with a set of credentials for two separate user accounts. Lets configure those accounts with **AWS Cli**.
```
aws configure --profile Lara
aws configure --profile McDuck
```
 
That’s it! You’ve now successfully configured and deployed CloudGoat with the vulnerable RCE ec2 instance.

Make sure to save all of your keys and credentials in a password manager. 
