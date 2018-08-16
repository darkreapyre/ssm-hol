# Estate and Patch Management: Lab guide
In the cloud, you can apply the same engineering discipline that you use for application code to your entire environment. You can define your entire workload (applications, infrastructure, etc.) as code and update it with code. You can script your operations procedures and automate their execution by triggering them in response to events. By performing operations as code, you limit human error and enable consistent execution of operations activities.  

In this lab you will apply the concepts of Infrastructure as Code and Operations as Code to the following activities:
- Deployment of Infrastructure
- Estate Management
- Patch Management

Included in the lab guide are bonus sections that can be completed if you have time or later if interested.
- Creating Maintenance Windows and Scheduling Automated Operations Activities
- Create and Subscribe to a Simple Notification Service Topic

>__NOTE:__ At the end of the lab guide there is an additional section on how to remove all resources you created. Remove the resources you created. Otherwise you will be charged for any resources that are not covered in the AWS Free Tier.

---
# 1. Setup
## Requirements
You will need the following to be able to perform this lab:
- Your own device for console access.
- An AWS account that you are able to use for testing, that is not used for production or other purposes.
- IAM user with `AdministratorAccess` Policy.
- An available region within your account with capacity to add 2 additional VPCs.

>__NOTE:__ You will be billed for any applicable AWS resources used if you complete this lab that are not covered in the AWS Free Tier.

## 1.1 Log in to the AWS Management Console and create an EC2 KEy Pair

[Amazon EC2 uses public–key cryptography](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) to encrypt and decrypt login information. Public–key cryptography uses a public key to encrypt a piece of data, such as a password, then the recipient uses the private key to decrypt the data. The public and private keys are known as a key pair. To log in to the Amazon Linux instances we will create in this lab, you must create a key pair, specify the name of the key pair when you launch the instance, and provide the private key when you connect to the instance.
1. Use your administrator account to access the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2. In the IAM navigation pane under __Network & Security__, choose __Key Pairs__ and then choose __Create Key Pair__.
3. In the __Create Key Pair__ dialog box, type a __Key pair name__ such as `OELab2018` and then choose __Create__.
4. Save the `keyPairName.pem` file for optional later use [accessing](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html) the EC2 instances created in this lab.

---
# 2. Deploy an Environment Using Infrastructure as Code
## Tagging
We will make extensive use of tagging throughout the lab. The CloudFormation template for the lab includes the definition of multiple tags against a variety of resources.
AWS enables you to assign metadata to your AWS resources in the form of [tags](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html). Each tag is a simple label consisting of a customer-defined key and an optional value that can make it easier to manage, search for, and filter resources. Although there are no inherent types of tags, commonly adopted categories of tags include technical tags (e.g., Environment, Workload, InstanceRole, and Name), tags for automation (e.g., Patch Group, and SSMManaged), business tags (e.g., Owner), and security tags (e.g., Confidentiality).

Apply the following best practices when using tags:
- Always use a standardized, case-sensitive format for tags, and implement it consistently across all resource types
- Consider tag dimensions that support the following:
    - Managing resource access control with IAM
    - Cost tracking
    - Automation
    - AWS console organization
- Implement automated tools to help manage resource tags. The Resource Groups Tagging API enables programmatic control of tags, making it easier to automatically manage, search, and filter tags and resources.
- Err on the side of using too many tags rather than too few tags.
- Develop a tagging strategy.

__Remember that it is easy to modify tags to accommodate changing business requirements; however, consider the
ramifications of future changes, especially in relation to tag-based access control, automation, or upstream billing reports.__

>__NOTE:__ `Patch Group` is a reserved tag key used by Systems Manager Patch Manager that is case sensitive with a space between the two words.

## CloudFormation
AWS [CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) is a service that helps you model and set up your Amazon Web Services resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and AWS CloudFormation provisions and configures those resources for you. AWS CloudFormation enables you to use a template file to create and delete a collection of resources together as a single unit (a stack).
There is no additional charge for AWS CloudFormation. You pay for AWS resources (such as Amazon EC2 instances, Elastic Load Balancing load balancers, etc.) created using AWS CloudFormation in the same manner as if you created the resources manually. You only pay for what you use, as you use it; there are no minimum fees and no required upfront commitments.

## 2.1 Deploy the Lab Infrastructure
1. Download and save the `cfn-template.yaml` file from the GitHub repository.
2. Use your administrator account to access the CloudFormation console at https://console.aws.amazon.com/cloudformation/.
3. Choose __Create Stack__.
4. On the __Select Template__ page, choose __Upload a template to Amazon S3__.
5. Click the __Browse__ button and select the `cfn-template.yaml` an click __Next__.

A CloudFormation template is a JSON or YAML formatted text file that describes your AWS infrastructure containing both optional and required sections. In the next steps, we will provide a name for our stack and parameters that will be passed into the template to help define the resources that will be implemented.
1. In the __Specify Details__ section, define a __Stack name__, such as `OELabStack1`.
2. In the __Parameters__ section:
    - Leave __InstanceProfile__ blank as we have not yet defined an instance profile.
    - Leave __InstanceTypeApp__ and __InstanceTypeWeb__ as the default free-tier-eligible `t2.micro` value.
    - Select the EC2 KeyName you defined earlier from the list.
    - In a browser window, go to http://checkip.amazonaws.com/ to get your IP and then enter your IP address in __SSHLocation__ in CIDR notation (i.e., ending in /32).
    - Define the __Workload Name__ as `Test`.
    - Choose __Next__.
3. On the __Options__ page under __Tags__, define a __Key__ of __Owner__, with __Value__ set to the username you choose for your administrator. You can optionally define additional keys. The CloudFormation template will create all the example tags given in the discussion on tagging above.
4. Leave all other sections unmodified. Scroll to the bottom of the page and choose __Next__.
5. On the Review page, review your choices and then choose __Create__.
6. On the CloudFormation console page, click the refresh button (circular arrow) in the top right if your stack name is not displayed.
7. Check the box next to your Stack Name to see its details. You may need to choose the refresh button again to get details.
8. Choose the __Events__ tab for your selected workload to see the activity log from the creation of your CloudFormation stack.

When the __Status__ of your stack in filter list is __CREATE_COMPLETE__ you will have just created a representation of a typical lift and shift 2-tier application migrated to the cloud.

## The impact of Infrastructure as Code
With infrastructure as code, if you can deploy one environment, you can deploy any number of copies of that environment. In this example we have created a `Test` environment. Later, we will repeat these steps to deploy a `Prod` environment. The ability to on-demand, dynamically deploy temporary environments enables parallel experimentation, development, and testing efforts; duplication of environments to recreate and analyze errors; and the cut-over deployment of production systems using blue-green methodologies. These all contribute to reduced risk and increased operations effectiveness and efficiency.

---
# 3. Estate Management using Operations as Code
## Management Tools: Systems Manager
[AWS Systems Manager](https://aws.amazon.com/systems-manager/features/) is a collection of features that enable IT Operations in the cloud that we will explore throughout this lab.

There are set up [tasks](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-setting-up.html) and [pre-requisites](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-prereqs.html) that must be satisfied prior to using Systems Manager to manage your EC2 instances or on-premises systems in [hybrid](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-managedinstances.html) environments.
- You must be using a supported operating system. Windows, Amazon Linux, Ubuntu Server, RHEL, and CentOS all have supported versions.
- The SSM Agent must be installed. The SSM Agent for Windows also requires PowerShell 3.0 or later for some capabilities.
- Your EC2 instances must have outbound internet access.
- You must access Systems Manager in a supported region.
- Systems Manager requires an IAM role for instances that will process commands and a separate role for users executing commands.

SSM Agent is installed, by default, on Amazon Linux base AMIs dated 2017.09 and later. The SSM Agent is installed by default on Windows Server 2016 instances and instances created from Windows Server 2003-2012 R2 AMIs published in November 2016 or later.

__There is no additional charge for AWS Systems Manager__. You only pay for your underlying AWS resources managed or created by AWS Systems Manager (e.g., Amazon EC2 instances or Amazon CloudWatch metrics). You only pay for what you use, as you use it; there are no minimum fees and no upfront commitments.

## 3.1 Setting up Systems Manager
1. Use your administrator account to access the Systems Manager console at https://console.aws.amazon.com/systems-manager/.
2. Choose __Managed Instances__ from the navigation bar. If you have not satisfied the pre-requisites for Systems Manager, you will arrive at the __AWS Systems Manager Managed Instances__ page.
    - As a user with __AdministratorAccess permissions__, you already have User Access to Systems Manager.
    >__NOTE:__ if desired you can grant access to use Systems Manager with a more [restrictive](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-access-user.html) permission set.
    - The Amazon Linux AMIs used to create the instances in your environment are dated `2017.09`. They are [supported](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-supported-oses.html) operating systems and have the [SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html) installed by default.
    - If you are in a supported region the remaining step is to configure the IAM role for instances that will process commands.
3. Navigate to the [IAM](https://console.aws.amazon.com/iam/home) console to create an Instance Profile for Systems Manager managed instances.
    - In the navigation pane, choose __Roles__, and then choose __Create role__.
    - In the __Select type of trusted entity__ section, verify that the default __AWS service__ is selected.
    - In the __Choose the service that will use this role__ section, choose __EC2__ within the field of services to select it. This will open the __Select your use case__ section.
    - In the __Select your use case__ section, choose __EC2 Role for Simple Systems Manager__ to select it.
    - Then choose __Next: Permissions__.
4. Under __Attached permissions policy__, verify that `AmazonEC2RoleforSSM` is listed, and then choose __Next: Review__.
5. In the Review section:
    - Enter a Role name, such as `ManagedInstancesRole`.
    - Accept the default in the __Role description__.
    - Choose __Create role__.
6. You must apply this role to the instances you wish to manage with Systems Manager.
    - Navigate to the EC2 Console and choose __Instances__.
    - Select the first instance and then choose __Actions__, __Instance Settings__, and __Attach/Replace IAM Role__.
    - Under __Attach/Replace IAM Role__, select `ManagedInstancesRole` from the drop down list and choose __Apply__.
    - After you receive confirmation of success, choose __Close__.
    - Repeat this process, assigning `ManagedInstancesRole` to each of the *3* remaining instances.
7. Return to the Systems Manager console and choose __Managed Instances__ from the navigation bar. Periodically choose Managed Instances until your instances begin to appear in the list. Over the next couple of minutes your instances will be populated into the list as managed instances.

## 3.2 Create a Second CloudFormation Stack
Create a second CloudFormation stack using the procedure in 2.1 with the following changes:
- In the __Specify Details__ section, define a __Stack name__, such as `OELabStack2`.
- Specify the __InstanceProfile__ using the `ManagedInstancesRole` you defined.
- Define the __Workload Name__ as `Prod`.

## Systems Manager: Inventory
You can use [AWS Systems Manager Inventory](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html) to collect operating system (OS), application, and instance metadata from your Amazon EC2 instances and your on-premises servers or virtual machines (VMs) in your hybrid environment. You can query the metadata to quickly understand which instances are running the software and configurations required by your software policy, and which instances need to be updated.

## 3.3 Using Systems Manager Inventory to Track Your Instances
1. Under __Insights__ in the AWS Systems Manager navigation bar, choose __Inventory__.
    - Scroll down in the window to the __Corresponding managed instances__ section. Inventory currently contains only the instance data available from the EC2.
    - Choose the __InstanceID__ of one of your systems.
    - Examine each of the available tabs of data under the __Instance ID__ heading.
2. Inventory collection must be specifically configured and the data types to be collected must be specified.
    - Choose __Inventory__ in the navigation bar.
    - Choose __Setup Inventory__ in the top left corner of the window.
3. In the __Setup Inventory__ screen, you can define targets for inventory. You can select all managed instances in this account, ensuring that all managed instances will be inventoried. You can constrain inventoried instances to those with specific tags, such as Environment or Workload. Or you can manually select specific instances for inventory.
    - Under __Specify targets by__, select __Specifying a tag__.
    - Under __Tags__ specify `Environment` for the key and `OELab2018` for the value.
4. You can schedule the frequency with which inventory is collected. The default and minimum period is 30 minutes.
    - For __Collect inventory data every__, accept the default *30 Minute(s)*.
5. Under parameters, you can specify what information to collect with the inventory process.
    - Review the options and select the defaults.
7. Choose __Setup Inventory__.
    >__NOTE:__ it can take up to 10 minutes to deploy a new inventory policy to an instance.
8. You can create multiple Inventory specifications. They will each be stored as __associations__ within __Systems Manager State Manager__.
    - To create a new inventory policy, from __Inventory__, choose __Setup inventory__.
    - To edit an existing policy, from __State Manager__ in the left navigation menu, select the association and choose __Edit__.

>__Optional Exercise:__ If desired, you can check the box next to __Sync inventory execution logs to an S3 bucket__ under the Advanced options and provide an S3 bucket name, and optional S3 bucket prefix, for the inventory execution logs. You will need to [create](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html) a bucket prior to proceeding.

## Systems Manager: State Manager
In State Manager, an [association](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-associations.html) is the result of binding configuration information that defines the state you want your instances to be in to the instances themselves. This information specifies when and how you want instance- related operations to run that ensure your Amazon EC2 and hybrid infrastructure is in an intended or consistent state.

An association defines the state you want to apply to a set of targets. An association includes three components:
- a document that defines the state
- target(s)
- a schedule

>__NOTE:__ Optionally you can also specify runtime parameters.

When you performed the __Setup Inventory__ actions, you created an association in State Manager.

## 3.4 Review Association Status
Under __Actions__ in the navigation bar, select __State Manager__. At this point in time the Status may reflect the fact that the inventory activity has not yet completed.
1. Choose the single Association id that is the result of your __Setup Inventory__ action.
2. Examine each of the available tabs of data under the *Association ID* heading.
3. Choose __Edit__.
4. Enter a name under __Name - optional__ to provide a more user friendly label to the association, such as `InventoryAllInstances`. 
>__NOTE:__ White space is not permitted in an association name.

Inventory is accomplished through the activities defined in the `AWS-GatherSoftwareInventory` command document. The parameters provided in the *Parameters* section are passed to the document at execution. The targets are defined in the *Targets* section. In this example there is a single target, which is a wildcard that matches all instances. The schedule for this activity is defined under *Specify schedule* and *Specify* with to use a CRON/Rate expression on a 30 minute interval. Finally there is the option to specify *Output options*. If you change the command document, the __Parameters__ section will change to be appropriate to that command document.
1. Navigate to __Managed Instances__ under __Shared Resources__ in the navigation bar.
2. Note that an __Association Status__ has been established for the inventoried instances under management.
3. Choose one of the __Instance ID__ links to go to the inventory of the instance. Note that the Inventory tab is now populated and you can track associations and their last activity under the Associations tab.
4. Navigate to __Compliance__ under __Insights__ in the navigation bar. Here you can view the overall compliance status of your managed instances in the __Compliance Summary__ and the individual compliance status of systems in the Corresponding managed instances section below.

>__NOTE:__ The inventory activity can take up to 10 minutes to complete. While waiting for it to complete you can proceed with the next section.

## Systems Manager: Compliance
You can use AWS Systems Manager [Configuration Compliance](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-compliance.html) to scan your fleet of managed instances for patch compliance and configuration inconsistencies. You can collect and aggregate data from multiple AWS accounts and Regions, and then drill down into specific resources that aren’t compliant. By default, Configuration Compliance displays compliance data about Systems Manager Patch Manager patching and Systems Manager State Manager associations. You can also customize the service and create your own compliance types based on your IT or business requirements. You can also port data to Amazon Athena and Amazon QuickSight to generate fleet-wide reports.

---
# 4. Patch Management Systems Manager: Patch Manager
AWS Systems Manager Patch Manager automates the process of [patching](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html) managed instances with security related updates. For Linux-based instances, you can also install patches for non-security updates. You can patch fleets of Amazon EC2 instances or your on-premises servers and virtual machines (VMs) by operating system type. This includes supported versions of Windows, Ubuntu Server, Red Hat Enterprise Linux (RHEL), SUSE Linux Enterprise Server (SLES), and Amazon Linux. You can scan instances to see only a report of missing patches, or you can scan and automatically install all missing patches. You can target instances individually or in large groups by using Amazon EC2 tags.

__Important Notes:__  
__- AWS does not test patches for Windows or Linux before making them available in Patch Manager.__  
__- If any updates are installed by Patch Manager the patched instance is rebooted.__  
__- Always test patches thoroughly before deploying to production environments.__  

## Patch Baselines
Patch Manager uses __patch baselines__, which include rules for auto-approving patches within days of their release, as well as a list of approved and rejected patches. Later in this lab we will schedule patching to occur on a regular basis using a __Systems Manager Maintenance Window__ task. Patch Manager integrates with AWS Identity and Access Management (IAM), AWS CloudTrail, and Amazon CloudWatch Events to provide a secure patching experience that includes event notifications and the ability to audit usage. 

>__NOTE:__ The operating systems supported by Patch Manager may vary from those supported by the SSM Agent.

## 4.1 Create a Patch Baseline
1. Under __Actions__ in the AWS Systems Manager navigation bar, choose __Patch Manager__.
2. Choose __Create patch baseline__.
3. On the __Create patch baseline page__ in the __Provide patch baseline details__ section:
    - Enter a Name for your custom patch baseline, such as `AmazonLinuxSecAndNonSecBaseline`.
    - Optionally enter a description, such as `Amazon Linux patch baseline including security and non-security patches`.
    - Select Amazon Linux from the list.
4. In the __Approval rules__ section:
    - Examine the options in the lists but leave __Product__, __Classification__, and __Severity__ at their default of __All__.
    - Leave the __Auto approval delay__ at its default of __0 days__.
    - Change the value of __Compliance level - optional__ to __Critical__.
    - Choose __Add another rule__.
    - In the new rule, change the value of __Compliance level - optional__ to __Medium__.
    - Check the box under __Include non-security updates__ to include all Amazon Linux updates when patching.

If an approved patch is reported as missing, the option you choose in Compliance level, such as `Critical` or `Medium`, determines the severity of the compliance violation reported in System Manager Compliance.
1. In the __Patch exceptions__ section in the __Rejected patches - optional__ text box, enter `system-release.*`. This will reject patches to new Amazon Linux releases that may advance you beyond the Patch Manager supported operating systems prior to your testing new releases.
2. For Linux operating systems, you can optionally define an [alternative patch source repository](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-how-it-works-alt-source-repository.html). Choose the __X__ in the __Patch sources__ area to remove the empty patch source definition.
3. Choose __Create patch baseline__ and you will go to the __Patch Baselines__ page where the AWS provided default patch baselines, and your custom baseline, are displayed.

## Patch Groups
A patch group is an optional method to organize instances for patching. For example, you can create patch groups for different operating systems (Linux or Windows), different environments (Development, Test, and Production), or different server functions (web servers, file servers, databases). Patch groups can help you avoid deploying patches to the wrong set of instances. They can also help you avoid deploying patches before they have been adequately tested.

You create a patch group by using Amazon EC2 tags. Unlike other tagging scenarios across Systems Manager, a patch group must be defined with the tag key: `Patch Group`.

>__NOTE:__ The key is case sensitive. 

You can specify any value, for example, *"web servers"*, but the key must be `Patch Group`.

>__NOTE:__ An instance can only be in one patch group.

After you create a patch group and tag instances, you can register the patch group with a patch baseline. By registering the patch group with a patch baseline, you ensure that the correct patches are installed during the patching execution. When the system executes the task to apply a patch baseline to an instance, the service checks to see if a patch group is defined for the instance.
- If the instance is assigned to a patch group, the system then checks to see which patch baseline is registered to that group.
- If a patch baseline is found for that group, the system applies that patch baseline.
- If an instance isn't configured for a patch group, the system automatically uses the currently configured default patch baseline.

## 4.2 Assign a Patch Group
1. Choose the __Baseline ID__ of your newly created baseline to enter its details screen.
2. Choose __Actions__ in the top right of the window and select __Modify patch groups__.
3. In the __Modify patch groups__ window under __Patch groups__, enter `Critical`, choose __Add__, and then choose __Close__ to be returned to the __Patch Baseline__ details screen.

## AWS-RunPatchBaseline
AWS-RunPatchBaseline is a command document that enables you to control patch approvals using patch baselines. It reports patch compliance information that you can view using the Systems Manager Compliance tools, such as which instances are missing patches and what those patches are. For Linux operating systems, compliance information is provided for patches from both the default source repository configured on an instance and from any alternative source repositories you specify in a custom patch baseline. AWS- RunPatchBaseline supports both Windows and Linux operating systems.

## AWS Systems Manager: Document
An AWS Systems Manager document defines the actions that Systems Manager performs on your managed instances. Systems Manager includes many pre-configured documents that you can use by specifying parameters at runtime, including AWS-RunPatchBaseline. Documents use JavaScript Object Notation (JSON) or YAML, and they include steps and parameters that you specify.

All AWS provided Automation and Run Command documents can be viewed in AWS Systems Manager Documents. You can [create your own documents](https://docs.aws.amazon.com/systems-manager/latest/userguide/create-ssm-doc.html) or launch existing scripts using provided documents to implement custom operations as code activities.

## 4.3 Examine AWS-RunPatchBaseline in Documents
1. In the AWS Systems Manager navigation bar under __Shared Resources__, choose __Documents__.
2. Click in the __search box__, select __Document name prefix__, and then __Equal__.
3. Enter `AWS-Run` into the text field and press enter.
4. Select AWS-RunPatchBaseline and choose __View details__.
5. Review the content of each tab in the details page of the document.

## AWS Systems Manager: Run Command
AWS Systems Manager Run Command lets you remotely and securely manage the configuration of your managed instances. Run Command enables you to automate common administrative tasks and perform ad hoc configuration changes at scale. You can use Run Command from the AWS Management Console, the AWS Command Line Interface, AWS Tools for Windows PowerShell, or the AWS SDKs.

## 4.4 Scan Your Instances with AWS-RunPatchBaseline via Run Command
1. Under __Actions__ in the AWS Systems Manager navigation bar, choose __Run Command__. In the Run Command dashboard, you will see previously executed commands including the execution of AWS- RefreshAssociation, which was performed when you set up inventory.
2. *(Optional)* choose a Command ID from the list and examine the record of the command execution.
3. Choose __Run Command__ in the top right of the window.
4. In the __Run a command__ window, under __Command document__:
    - Choose the search icon and select `Platform`, and then choose `Linux` to display all the available commands that can be applied to Linux instances.
    - Choose __AWS-RunPatchBaseline__ in the list.
5. In the __Targets__ section:
    - Under __Specify targets by__, choose __Specifying a tag__ to reveal the __Tags__ sub-section.
    - Under __Enter a tag key__, enter `Workload`, and under __Enter a tag value__, enter `Test`.
6. In the __Command parameters__ section, leave the __Operation__ value as the default __Scan__.

The remaining Run Command features enable you to:
- Specify Rate control, limiting Concurrency to a specific number of targets or a calculated percentage of systems, or to specify an Error threshold by count or percentage of systems after which the command execution will end.
- Specify Output options to record the entire output to a preconfigured S3 bucket and optional S3 key prefix.
    >__NOTE:__ Only the last 2500 characters of a command's output are displayed in the console.
- Specify SNS notifications to a specified SNS Topic on all events or on a specific event type for either the entire command or on a per-instance basis. This requires Amazon SNS to be preconfigured.
- View the command as it would appear if executed within the AWS Command Line Interface.
- Choose Run to execute the command and return to its details page.
- Scroll down to Targets and outputs to view the status of the individual targets that were selected through your tag key and value pair. Refresh your page to update the status.
- Choose an Instance ID from the targets list to view the Output from command execution on that instance.
- Choose Step 1 - Output to view the first 2500 characters of the command output from Step 1 of the command, and choose Step 1 - Output again to conceal it.
- Choose Step 2 - Output to view the first 2500 characters of the command output from Step 2 of the command. Note that this execution step PatchWindows was skipped as it did not apply to your Amazon Linux instance.
- Choose Step 1 - Output again to conceal it.

## 4.5 Review Initial Patch Compliance
1. Under __Insights__ in the the AWS Systems Manager navigation bar, choose __Compliance__.
2. On the __Compliance__ page in the __Compliance Summary__, you will now see that there are 4 systems that have critical severity compliance issues. In the __Corresponding managed instances__ list, you will see the individual compliance status and details.

## 4.6 Patch Your Instances with AWS-RunPatchBaseline via Run Command
1. Under __Actions__ in the AWS Systems Manager navigation bar, choose __Run Command__.
2. Choose __Run Command__ in the top right of the window.
3. In the __Run a command__ window, under __Command document__:
    - Choose the search icon, select `Platform`, and then choose `Linux` to display all the available commands that can be applied to Linux instances.
    Choose __AWS-RunPatchBaseline__ in the list.
4. In the __Targets__ section:
    - Under __Specify targets by__, choose __Specifying a tag__ to reveal the __Tags__ sub-section.
    - Under __Enter a tag key__, enter `Workload` and under __Enter a tag value__ enter `Test`.
5. In the __Command parameters__ section, change the __Operation__ value to __Install__.
6. In the __Targets__ section, you can choose either __Specify a tag__ using `Workload` and `Test` or __Manually selecting instances__ and use the check box at the top of the list to select all instances displayed, or select them individually.
>__NOTE:__ There are multiple pages of instances and selections on each page must be selected specifically.
7. In the __Rate control__ section:
    - For __Concurrency__, leave the default targets selected and specify __1__.
    - For __Error threshold__, leave the default errors selected and specify __1__.
    >__NOTE:__ If any updates are installed by Patch Manager, the patched instance is rebooted.
    - Limiting concurrency will stagger the application of patches and the reboot cycle, however, to ensure that your instances are not rebooting at the same time, create separate tags to define target groups and schedule the application of patches at separate times.
8. Choose __Run__ to execute the command and to go to its details page.
9. Refresh the page to view updated status and proceed when the execution is successful.

## 4.7 Review Patch Compliance After Patching
1. Under __Insights__ in the the AWS Systems Manager navigation bar, choose __Compliance__.
2. On the __Compliance__ page, change the __Compliance Type:__ to `Patch`.
3. The Compliance Summary will now show that there are 4 systems that have satisfied critical severity patch compliance.

## The Impact of Operations as Code
In a traditional environment you would have had to set up the systems and software to perform these activities. You would require a server from which to execute your scripts. You would need to manage authentication credentials across all of your systems. To access the state of your systems may require access to multiple systems.

Operations as code: reduces the resources, time, risk, and complexity of performing operations tasks and ensures consistent execution. You can take operations as code and automate operations activities by using scheduling and event triggers. Through integration at the infrastructure level you avoid "swivel chair" processes that require multiple interfaces and systems to complete a single operations activity.

---
# Creating Maintenance Windows and Scheduling Automated Operations Activities
## AWS Systems Manager: Maintenance Windows
AWS Systems Manager Maintenance Windows let you define a schedule for when to perform potentially disruptive actions on your instances such as patching an operating system (OS), updating drivers, or installing software. Each Maintenance Window has a schedule, a duration, a set of registered targets, and a set of registered tasks. With Maintenance Windows, you can perform tasks like the following:
- Installing applications, updating patches, installing or updating SSM Agent, or executing PowerShell commands and Linux shell scripts by using a Systems Manager Run Command task.
- Building Amazon Machine Images (AMIs), boot-strapping software, and configuring instances by using Systems Manager Automation.
- Executing AWS Lambda functions that trigger additional actions such as scanning your instances for patch updates.
- Running AWS Step Function state machines to perform tasks such as removing an instance from an Elastic Load Balancing environment, patching the instance, and then adding the instance back to the Elastic Load Balancing environment.
>__NOTE:__ To register Step Function tasks you must use the AWS CLI.

## 5.1 Setting up Maintenance Windows
1. Navigate to the IAM console to create a role so that Systems Manager can run tasks in Maintenance Windows on your behalf.
    - In the navigation pane, choose __Roles__, and then choose __Create role__.
    - In the __Select type of trusted entity__ section, verify that the default __AWS service__ is selected.
    - In the __Choose the service that will use this role__ section, choose __EC2__. This allows EC2 instances to call AWS services on your behalf.
    - Choose __Next: Permissions__.
2. Under __Attached permissions policy__, search for __AmazonSSMMaintenanceWindowRole__, check the box next to it in the list, and then choose __Next: Review.__
3. In the __Review__ section:
    - Enter a __Role name__, such as `SSMMaintenanceWindowRole`.
    - Enter a __Role description__, such as `Role for Amazon SSMMaintenanceWindow`.
    - Choose __Create role__. Upon success you will be returned to the Roles screen.
4. To enable the service to run tasks on your behalf, we need to edit the trust relationship for this role.
    - Choose the role you just created to enter its __Summary__ page.
    - Choose the __Trust relationships__ tab.
    - Choose __Edit trust relationship__.
    - Delete the current policy, and then copy and paste the following policy into the __Policy Document__ field:
    ```json
    {
         "Version":"2012-10-17",
         "Statement":[
          {
              "Sid":"",
              "Effect":"Allow",
              "Principal":{
                  "Service":[
                      "ec2.amazonaws.com",
                      "ssm.amazonaws.com",
                      "sns.amazonaws.com"
                  ]
              },
              "Action":"sts:AssumeRole"
              }
        ]
    }
    ```
5. Choose __Update Trust Policy__. You will be returned to the now updated Summary page for your role.
6. Copy the __Role ARN__ to your clipboard by choosing the double document icon at the end of the ARN.

When you register a task with a Maintenance Window, you specify the role you created, which the service will assume when it runs tasks on your behalf. To register the task, you must assign the IAM PassRole policy to your IAM user account. The policy in the following procedure provides the minimum permissions required to register tasks with a Maintenance Window.

1. Create the IAM PassRole policy for your Administrators IAM user group:
    - In the IAM console navigation pane, choose __Policies__, and then choose __Create policy__.
    - On the Create policy page, in the __Select a service area__, next to __Service__ choose __Choose a service__, and then choose __IAM__.
    - In the __Actions__ section, search for __PassRole__ and check the box next to it when it appears in the list.
    - In the __Resources__ section, choose "You choose actions that require the role resource type.", and then choose __Add ARN__ to restrict access. The Add ARN(s) window will open.
    - In the __Add ARN(s)__ window, in the __Specify ARN for role field__, delete the existing entry, paste in the role ARN you created in the previous procedure, and then choose Add to return to the Create policy window.
    - Choose __Review policy__.
    - On the __Review Policy__ page, type a name in the Name box, such as `SSMMaintenanceWindowPassRole` and then choose __Create policy__. You will be returned to the Policies page.
2. Assign the IAM PassRole policy to your Administrators IAM user group:
    - In the IAM console navigation pane, choose __Groups__, and then choose your __Administrators__ group to reach its Summary page.
    - Under the permissions tab, choose __Attach Policy__.
    - On the __Attach Policy__ page, search for `SSMMaintenanceWindowPassRole`, check the box next to it in the list, and choose __Attach Policy__. You will be returned to the Summary page for the group.

## Creating Maintenance Windows
To create a Maintenance Window, you must do the following:
- Create the window and define its schedule and duration.
- Assign targets for the window.
- Assign tasks to run during the window.

After you complete these steps, the Maintenance Window runs according to the schedule you defined and runs the tasks on the targets you specified. After a task is finished, Systems Manager logs the details of the execution.

## 5.2 Create a Patch Maintenance Window
First you must create the window and define its schedule and duration.
1. Open the AWS Systems Manager console.
2. In the navigation pane, choose __Maintenance Windows__ and then choose __Create a Maintenance Window__.
3. In the __Provide maintenance window details__ section:
    - In the __Name__ field, type a descriptive name to help you identify this Maintenance Window, such as `PatchTestWorkloadWebServers`.
    - *(Optionally)*, you may enter a description in the Description field.
    - Choose __Allow unregistered targets__ if you want to allow a Maintenance Window task to run on managed instances, even if you have not registered those instances as targets.
    >__NOTE:__ If you choose this option, then you can choose the unregistered instances (by instance ID) when you register a task with the Maintenance Window. If you don't choose this option, then you must choose previously registered targets when you register a task with the Maintenance Window. Additionally, if you don't choose this option, then you must choose previously-registered targets when you register a task with the Maintenance Window.
4. Specify a schedule for the Maintenance Window by using one of the scheduling options.
    - Under __Specify with__, accept the default __Cron schedule builder__.
    - Under __Window starts__, choose the third option, specify __Every Day at__, and select a time, such as `02:00`.
    - In the __Duration__ field, type the number of hours the Maintenance Window should run, such as `3 hours`.
    - In the __Stop initiating tasks__ field, type the number of hours before the end of the Maintenance Window that the system should stop scheduling new tasks to run, such as `1` hour before the window closes.
5. *(Optionally)* to have the maintenance window execute more rapidly while engaged with the lab:
    - Under __Window starts__, choose Every __30 minutes__ to have the tasks execute on every hour and every half hour.
    - Set the __Duration__ to the minimum `1` hours.
    - Set the __Stop initiation__ tasks to the minimum `0` hours.
6. Choose __Create maintenance window__. The system returns you to the __Maintenance Window__ page. The state of the Maintenance Window you just created is __Enabled__.

## 5.3 Assigning Targets to Your Patch Maintenance Window
After you create a Maintenance Window, you assign targets where the tasks will run.
1. On the __Maintenance windows__ page, choose the __Window ID__ of your maintenance window to enter its __Details__ page.
2. Choose __Actions__ in the top right of the window and select __Register targets__.
3. On the __Register target__ page under __Maintenance window target details__:
    - In the __Target Name__ field, enter a name for the targets, such as `TestWebServers`.
    - *(Optionally)* Enter a description in the Description field.
    - *(Optionally)* Specify a name or work alias in the Owner information field.
    >__NOTE:__ Owner information is included in any CloudWatch Events that are raised while running tasks for these targets in this Maintenance Window.
4. In the __Targets__ section, under __Select Targets by__:
    - Choose the default __Specifying tags__ to target instances by using Amazon EC2 tags that were previously assigned to the instances.
    - Under __Tags__, enter `Workload` as the key and `Test` as the value. The option to add and additional tag key/value pair will appear.
    - Add a second key/value pair using `InstanceRole` as the key and `WebServer` as the value.
5. Choose __Register target__ at the bottom of the page to return to the maintenance window details page.

If you want to assign more targets to this window, choose the Targets tab, and then choose __Register target__ to register new targets. With this option, you can choose a different means of targeting. For example, if you previously targeted instances by instance ID, you can register new targets and target instances by specifying Amazon EC2 tags.

## 5.4 Assigning Tasks to Your Patch Maintenance Window
After you assign targets, you assign tasks to perform during the window.
1. From the details page of your maintenance window, choose __Actions__ in the top right of the window and select __Register Run command__ task.
2. On the __Register Run command task__ page:
    - In the __Name__ field, enter a name for the task, such as `P`atchTestWorkloadWebServers`.
    - *(Optionally)* Enter a description in the Description field.
3. In the __Command document__ section:
    - Choose the search icon, select `Platform,` and then choose `Linux` to display all the available commands that can be applied to Linux instances.
    - Choose __AWS-RunPatchBaseline__ in the list.
    - Leave the __Task priority__ at the default value of __1__.
    >__NOTE:__ 1 is the highest priority.
    - Tasks in a Maintenance Window are scheduled in priority order, with tasks that have the same priority scheduled in parallel.
4. In the __Targets__ section:
    - For __Target by__, select __Selecting registered target groups__.
    - Select the group you created from the list.
5. In the __Rate control__ section:
    - For __Concurrency__, leave the default targets selected and specify __1__.
    - For __Error threshold__, leave the default errors selected and specify __1__.
6. In the __Role__ section, specify the role you defined with the `AmazonSSMMaintenanceWindowRole`.
7. In __Output options__, leave Enable writing to S3 clear.
8. In __SNS notifications__, leave Enable SNS notifications clear.
9. In the __Parameters__ section, under __Operation__, select __Install__.
10. Choose __Register Run command task__ to complete the task definition and return to the details page.

## 5.5 Review Maintenance Window Execution
1. After allowing enough time for your maintenance window to have completed:
    - Navigte to the AWS Systems Manager console.
    - Choose __Maintenance Windows__, and then select the __Window ID__ for your new maintenance window.
2. On the __Maintenance window ID__ details page, choose __History__.
3. Select a __Windows execution ID__ and choose __View details__.
4. On the __Command ID__ details page, scroll down to the __Targets and outputs__ section, select an __InstanceID__, and choose __View output__.
5. Choose __Step 1 - Output__ and review the output.
6. Choose __Step 2 - Output__ and review the output.

You have now configured a maintenance window, assigned targets, assigned tasks, and validated successful execution. The same procedures can be used to schedule the execution of any AWS Systems Manager Document.

---
# Creating a Simple Notification Service Topic
Amazon Simple Notification Service (Amazon SNS) coordinates and manages the delivery or sending of messages to subscribing endpoints or clients. In Amazon SNS, there are two types of clients: publishers and subscribers. These are also referred to as producers and consumers. Publishers communicate asynchronously with subscribers by producing and sending a message to a topic, which is a logical access point and communication channel. Subscribers (i.e., web servers, email addresses, Amazon SQS queues, AWS Lambda functions) consume or receive the message or notification over one of the supported protocols (i.e., Amazon SQS, HTTP/S, email, SMS, Lambda) when they are subscribed to the topic.
6.1 Create and Subscribe to an SNS Topic
1. Navigate to the SNS console at https://console.aws.amazon.com/sns/.
2. Choose __Create topic__.
3. In the __Create new topic__ window:
    - In the __Topic name__ field, enter `AdminAlert`.
    - In the __Display name__ field, enter `AdminAlert`.
    - Choose __Create topic__.
4. On the __Topic details: AdminAlert__ page, choose __Create subscription__.
5. In the __Create subscription__ window:
    - Select __Email__ from the __Protocol__ list.
    - Enter your email address in the __Endpoint__ field.
    - Choose __Create subscription__.
6. You will receive an email request for confirmation. Your Subscription ID will remain __PendingConfirmation__ until you confirm your subscription by clicking through the link to __Confirm subscription__ in the email.
7. Refresh the page after confirming your subscription to view the populated __Subscription ARN__.

You can now use this SNS topic to send notifications to your Administrator user.

---
# 7 Removing Lab Resources

>__NOTE:__ When the lab is complete, remove the resources you created. Otherwise you will be charged for any resources that are not covered in the AWS Free Tier.

## 7.1 Remove resources created with CloudFormation
1. Navigate to the CloudFormation dashboard at https://console.aws.amazon.com/cloudformation/:
     - Select your first stack.
     - Choose __Actions__ and choose __delete stack__.
     - Select your second stack.
     - Choose __Actions__ and choose __delete stack__.
2. Navigate to Systems Manager console at https://console.aws.amazon.com/systems-manager/:
    - Choose __State Manager__.
    - Select the association you created.
    - Choose Delete.
3. If you created an __S3 bucket__ to store detailed output, delete the bucket and associated data:
    - Navigate to the S3 console https://s3.console.aws.amazon.com/s3/.
    - Select the bucket.
    - Choose Delete and provide the bucket name to confirm deletion.
4. If you created the optional __SNS Topic__, delete the SNS topic:
    - Navigate to the SNS console https://console.aws.amazon.com/sns/.
    - Select your `AdminAlert` SNS topic from the list.
    - Choose __Actions__ and select __Delete topics__.
5. If you created a __Maintenance Window__, delete the Maintenance Window:
    - Navigate to the __Systems Manager console__ at https://console.aws.amazon.com/systems-manager/.
    - Choose __Maintenance Windows__.
    - Select the maintenance window you created.
    - Choose __Delete__.
    - In the __Delete maintenance window__ window, choose __Delete__.