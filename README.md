# FME Training Virtual Machines and Automation
The files in this repository are used to create virtual machines for FME training courses, and to allow students to request virtual machines on-demand.
The virtual machines are Amazon AWS EC2 machines.
The webpage used for requesting the virtual machines is a static page hosted on AWS S3.

## Prerequisites
A basic understanding of [GitHub](https://guides.github.com/activities/hello-world/), [Amazon AWS](https://aws.amazon.com/ec2/?hp=tile&so-exp=below), FME Desktop, and FME Server is required.
You will also require an AWS account and a GitHub account.

On your local machine, you will need an installation of [GitHub Desktop](https://desktop.github.com/) and FME Desktop. You may also need [Boto3 for Python](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) installed on your local machine.

Be aware that you'll probably have to request an Instance Limit increase for the EC2 instances. The default limit is 20 machines per region. At Safe Software, we have a limit of 500. Keep in mind that the Limit increase request is per region. You'll have to submit a request for each region you intend to use.

Also be aware that the default VPC limit per region is 5. If you currently have 5 VPCs in your desired region, you will have to request an increase in the VPC limit.

![EC2 Service Increase](/images/EC2Limits.png)

## Overview
1. Fork this Repository to your own account
1. Install and update AWSCredentialSupplier.fmx
1. Edit settings.json
1. Run QuickSetup.fmw
1. Create and tag AMI
1. Publish VMCreator.fmw and GitClone2S3.fmw with AWSCredentialSupplier.fmx to FME Server/Cloud
1. Create FME Server App to allow virtual machine creation

There are two files in the repository that need to be edited, and two workspaces that need to be edited and published to FME Server. The two files you will eventually edit are:
1. settings.json
1. InitialConfiguration.bat

# Steps

## Create an IAM user that the workspaces can use
[Create a new IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) for **Programatic Access**. Select **Attach existing policies to user directly** and select the **AmazonEC2FullAccess** policy. This creates an access key and secret key. Make note of those somewhere safe; you'll need them later in AWSCredentialSupplier.fmx

## Fork this Repository to your own account
Click Fork.
Once forked into your own account, click Settings.
Change the repository name if desired.
Create a branch that will be the name of the course.

## Edit settings.json
* `git.username`    The GitHub username of the account containing your repository
* `git.repository`  The GitHub repository containing this file
* `git.branch`      The GitHub repository branch for this particular virtual machine. Also the AMI Description.
---
* `aws.name_tag`    The `Name` tag that will be attached to the various items created in AWS
* `aws.region_name` The EC2 region that will be hosting the virtual Machines. Full list under [Amazon Elastic Compute Cloud (Amazon EC2)](https://docs.aws.amazon.com/general/latest/gr/rande.html)
* `aws.vpc_cidr`  Private IP address range of the VPC
* `aws.subnet_cidr` Address ranges of each subnet
---
* `ami.linux` Search term used to find Linux AMI
* `ami.windows` Search term used to find Windows AMI. Use `"Microsoft Windows Server 2016 with Desktop Experience Locale English AMI provided by Amazon"` or `"Microsoft Windows Server 2019 with Desktop Experience Locale English AMI provided by Amazon"`
---
These are used for the FlexNet license server.
* `flexnet.ec2type` EC2 type. Recommend `T3` or `T3a`
* `flexnet.ec2size` EC2 size. Recommend `nano` or `micro`
* `flexnet.volumeSize`  The drive volume size in GB. Recommend `8`
* `flexnet.securityGroup` The name of the security group used by the license server
---
These are for the FME Server machine
* `fmeserver.ec2type` EC2 type. Recommend `T3` or `T3a`
* `fmeserver.ec2size`EC2 size. Recommend `medium` or `large`
* `fmeserver.volumeSize`  The drive volume size in GB. Recommend `40`
* `fmeserver.securityGroup` Name of the security group that will be created
* `fmeserver.docker-compose.yaml` Docker yaml url. [List can be found here](https://s3-us-west-2.amazonaws.com/safe-software-container-deployments/index.html)
---
These are for the training virtual machines
* `fme.ec2type` EC2 type. Recommend `T3` or `T3a`
* `fme.ec2size` EC2 size. Recommend `large` or `xlarge`
* `fme.volumeSize` The drive volume size in GB. Recommend `80`
* `fme.securityGroup` The security group the virtual machines will belong to

* `fme.timezone`    The desired [Windows timezone](https://techsupport.osisoft.com/Documentation/PI-Web-API/help/topics/timezones/windows.html) for the virtual machine
* `fme.portForwarding` Forwards additional ports to 3389 so they can be used for Remote Desktop
* `fme.firewall`  Ports to be opened in the Widows Firewall
* `fme.password`    The desired password for the virtual machine
* `fme.license`     The IP address of the floating license server
* `fme.installApps` Apps to be installed by [Chocolatey package manager](https://chocolatey.org/)
* `fme.vm.instanceInitiatedShutdownBehaviour` What happens to the virtual machine when it is turned off
* `fme.vm.Subject`  Email subject line
* `fme.vm.fromEmail`  Email address the connection files will be sent from
* `fme.vm.CCEmail`    Additional email address used to report problems when VM is created
* `fme.vm.BCCEmail`   BCC email address. Can be used to copy emails to a CRM

---


* `fme.vm.template.email` The template used for the email containing the RDP connection files
* `fme.vm.template.rdp`   The settings for the RDP files; watch out for the domain value

## Install AWSCredentialSupplier.fmx
The /transformers/AWSCredentialSupplier.fmx custom transformer will supply the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values in other workspaces. The values are kept in private parameters. Import the custom transformer by double-clicking on it. The parameters will be set later.

## Run QuickSetup.fmw
This step creates your AWS EC2 Environment
1. Open QuickSetup.fmw
1. Right-Click on the AWSCredentialSupplier transformer and select 'edit'
1. In AWSCredentialSupplier, set your AWS Access Key ID and AWS Secret Access Key values in the Private Parameters. Save the custom transformer.
1. Run Quicksetup.fmw

### Configure FlexNet License Server
1. Follow the instructions in LicenseServerInfo.txt file to request a license.
1. Edit the safe.lic file so that the serial number is removed.
1. Save the safe.lic file into your GitHub repository and push any updates.
1. Reboot the license server machine.

## Review/Edit OnstartConfiguration.bat
OnstartConfiguration.bat is run by the Task Scheduler on the virtual machines every time the virtual machine starts (or restarts). This allows you to perform additional configuration steps at startup.

## Create and tag AMI
Once the machine is configured, create an image (AMI) where the Description value is the same as the Git Branch name. The course is the name of the image (like training, or certification). This tag is used by the VMCreator.fmw file to launch virtual machines on demand.  

### Configure FME Server
1. Using the public IP address, log into FME Server. Username and password are `admin`
1. Change the admin password
1. Activate FME Server

## Publish VMCreator.fmw and AWSCredentialSupplier.fmx to FME Server/Cloud
1. Open VMCreator.fmw, set the private parameters `git.username` and `git.repository`, and add a webconnection for the `GMAIL_NAMED_CONNECTION` private parameter. Publish to the FMETraining repository (or a repository of your choice) on FME Server.

## Create FME Server App
You can unselect the course name. That way you can re-use the same workspace for multiple courses, and just create a new FME Server App for each course after forking the GitHub repository.

## Creating additional courses
Fork the current GitHub branch, and give it a name that matches the Description tag on the AMI.

## Set Calendar Reminders
If you are not using a permanent license for FME Server, you'll have to re-license it on occasion.
