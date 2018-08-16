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