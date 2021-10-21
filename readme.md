# Azure Design Technologies for Cloud Configuration

I have created this repository for my self learning and reference for expert Cloud Architect certifications

## Deployment cycle for application

- **Create a VM and a virtual network**
    - Specify inbound ports - The RDP is usually automatically selected, you can also add port 80
    - You will get an OS level disk as well as a temporary D drive, you can also add an additional disk for database
    - Select default encryption at rest
    - The VM can have a public IP address if it is hosting a web application that will be accessed by the public
        - A public IP address can also be added at a later time
    - Create a **virtual network** as you creat a VM 
        - Primary virtual network interface - has its own IP address, used for internal communication
    - An NSG is also created the controls permissions for the ports

- **Create a Storage account**
    - Azure blob service - The storage capacity automatically grows
    - While you can store data on your local VM - the storage account offers the flexibility of increasing capacity over time, limited options with the VM
    - Isolation of compute and the storage environment - data or the application are not dependent on eachother - especialy if the VM crashes, you can always create a new VM and connect it to the same storage account
    - Specifications
        - Account name, Geo-redundant storage, locally redundant storage is the least expensive
    - Go to the `Storage account resource || Under data storage: Containers (blob service), fileshares (across VMs), Queues (messaging), tables (No SQL like data)`

- **Creating an Azure SQL database** Platform as a service (PaaS)
    - Note that the SQL server, Database and the storage account must be in the same region
    - When you use AZURE SQL, you do not have to manually manage the underlying VM hosting the database, which means that you do not have to install the Azure SQL service engine, no security or patching of the database
    - You can create the SQL database from the main resources as usual
        - Specifications
            - `Create a server `- This is for logging into the SQL database server. It has 2 components, the server that hosts the database and the databse itself
                - Server name- must be unique
                - Admin login details
                - Select pricing models for database - `basic,standard, premium` - These are based on DTUs, data transfer units - costs are estimated per month
                - Select backup storage redundancy
                - Specify the connectivity options - a firewall is connected by default - options: No access, public endpoint, private endpoint
                - You can add your current computer IP address to the firewall rules as well as permit Azure service/resources to access the server
                - You can use **SQL server management studio** as you work with SQL databases - install this tool on your local machine to access the server 
    - Once created , you can locate the database in resources by its name, you can run the SQL server management studio
        - The server name is the webaddress of the SQL server which you can locate from the database resource homepage that you created and you use the login credentials in order to connect

    - **Building an appliation**
        - Will be hosted on an Azure VM and Azure webapp
        - The first part is setting up MYSQL server and a storage account and creating the required tables in the MYSQL database
            - **Setting up tables in the SQL database**
                - Use SQL server management studio to create tables and insert values
                - Select the database by name right click - new query and then you can paste your commands > click execute
                - Note that in the database, only the image links are stored, but you can store the images in the Azure storage account
            - **Uploading the images into the storage account**
                - Go to the `storage account || containers > create a new container and call it images > select Blob under public access level`
                - Open images and upload the file. You will get the image url
        - **Writing the code for application**
            - You can use various backend languages like .net, node, java. This program will have a frontend and a backend
            - The backend portion will contain the queries for database
        - **Setting up a webserver on VM to publish application**
            - Go to the `server dashboard > roles and features> select role based> check Webserver IIS > next and then check management services under management tools to allow solutions from virtual studio`
            - Open internet information services manager
                - Select management service and then click enable remote connections. Then click apply and start - this ensures that the webserver will take requests from virtual studio
                - Turn off enhanced security configuration from the main dashboard
            - Then you can download and install the .Net core 3.1 to configure the server to run a .net application - you have to select the `windows hosting bundle` - There is a different installation file based on the type of OS - for example there is a different one for linux
            - Next you have to install another software `web deploy` which deploys the application on to the internet information services
                - **Configure a DNS name**
                    - Go to the VM resource > click not configured under DNS name > type a DNS name label
                    - Go to the VM resource || networking > Add inbound rule to allow port 8172, the port that you assigned with the management service when configuring IIS on the VM
        - Once the installations are complete. You then have to create a `webapp resource` in Azure
            - **Create webapp from main resources > Create a resorce**
                - Give it a name
                - Select code vs container for now and select runtime stack as .Net core 3.1(LTS)
                - Select a region, and it will automatically select a new `app service plan`, you can also `select the sku such as the basic app service plan`
                - For the monitoring of your app you can create a new resource called `application insights` - give name and specify the region
                - Once the deployment is complete, go to the resource, you will find the url of the webapp. If you paste it in the browser, it will show a default webpage
                - Once all these steps are complete, the app will be deployed on the Vm as a webapp

        - **Application Monitoring**
            - Finally after the app has been configured and deployed. You can then setup monitoring, which will assess logs and you can configure alerts etc
            - You can select between static and dynamic routes
            - You can also create action groups, dictating what to do when a threshold is met. You can configure to e-mail and administrator or you can run scripts in response to an alert
        - **Configure logs** 
            - **Create Log analytics workspace** and connect the VM to it
            - Install an analytics agent on the VM this is what 'connect' does
            - You can then configure the agent to specify what type of logs are you looking to transfer to the workspace like events performance etc
            - Logs can be grouped into various categories

        - **NSG flow logs**
            - Captures logs for traffic - IP flows - useful for the security team
            - Useful information like source and destination IP and ports and the outcome of the address
            - You can locate the NSG group by goting to the `VM resource || networking`
            - When you click on the network security group, you will see there is an option for `flow logs` - turn it on, select the version- version 2 has additional information about the packaging of bit size etc
            - Logs are stored in the storage account - so a storage account must be linked with flow log settings. Once the account has been linked, you can verify it by going to the storage account and you will notice a new flow logs container
                - **Network watcher**
                    - This is another service that analyzes the traffic
                    - Search for network watcher from general resources `Network watcher || NSG flow logs > turn on traffic analytics status`
                    - Then also select the log analytics workspace
                    - You can specify the processing interval to create visualizations
                    - To view these visualizations go to `Network watcher || traffic analytics`
                    - There are options for malacious traffic and blocked traffic for a more granular analysis
                    - **Service Map**
                        - This feature offers a more in depth assessment of what processess are running on the VMs - Also provides the external ips and port numbers the VMs are connected to - in the form of a 'service map'
                        - Go to `workspace resource || Service map` to issue an agent. This can be done with a command that will install an Azure monitoring dependency agent as an extension
                        - Note that when you are first running the powershell, you may have to select a storage account for filesharing before you can run the commands
                        - Below is the command to install this agent
                        - `Set-AzVMExtension -ExtensionName "Microsoft.Azure.Monitoring.DependencyAgent" -ResourceGroupName "new-grp" -VMName "demovm" -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" -ExtensionType "DependencyAgentWindows" -TypeHandlerVersion 9.5 -Location NorthEurope`
                        - The service map now created, needs to be added to the workspace
                            - `workspace resource || workspace summary under general > add > in th search type service map > Create service map solution`
                            - Once created, under the `workspace resource || summary` you will be able to see the service map
            
            - **Azure SQL diagnostics**
                - GO to the `SQL DB resource || disgnostics under monitoring > Add diagnostic setting`
                - Then you can stream various types of logs like errors, timeouts, basic metrics, by selecting their category and then also selecting the destination details by checking 'send to log analytics workspace' Then you can select the log analytics workspace reseource
                - This setting will forward all the SQL diagnostics data to log analytics workspace

- You can review all the sign ins to the Azure AD by selecting the Azure Active directory from the main left panel and then you can select Users
- You can click the `default directory` from top left and then select `sign-ins` under monitoring - It will show all the sign in activity
- You can add a diagnostic setting - where you can select various type of logs such as `audit logs, signin logs` etc to a certain destination including analytics workspace or storage account or an event hub
    - Once this setting has been enabled. You can verify it by going to `work space resource || log management`

## Costing Estimates
 - Pricing calculator - you can get to it by searching for `Azure pricing calculator` 
    - There are various categories for which you can estimate costs like VM, storage accounts, SQL etc
    - **VM**
        - Windows VM cost more than linux due to licensing, almost double the price/mo
        - Region also effects price
        - Disk size and the duration 
        - Compute and Disks costs are separate. The compute costs are the running costs - even if the VM is shut off, you will still be charged for disks
        - Storage transactions also cost
        - Transfer of data on to different region can be costly
    - **Storage account**
        - Capacity
        - Write operations
        - Container and read operations
- Likewise there are other components which you can look into such as database, Kubernetes, Azure functios
- You can also look at cost-analysis by subscription
- You can select you `subsciption || cost analysis` which will give you break down according to service name, location, resource group
- Optimizing costs
    - It the resourse is not being used, then delete it
    - You can look at it by going to the resource homepage and under the `monitoring` tab you can review whether a particular resource is being used
    - If the CPU usage is less than 6% then it can tell you that you can possibly reduce the instance size 
    - This way you can determine what the sizing of the resource can be, the catch is that there is always some downtown with changing the size - so this should be tested in the staging environment rather than production
- **Azure advisor service** from the main dashboard 
    - It can provide recommendations based on cost
    - For example it can inform about a reserve instance to save on cost over a VM that is running for a long time, but not being utilized
- **Budgets**
    - You can go to your `subscription || budgets under cost management` You can define a monthly amount, if that is exceeded, you will get an email notification
- **Azure Reserve Pricing**
    - It is a huge saving opportunity - if you buy a 1-3 year commitment to reserve instance
    - This is bascally a discount that is offered with a commitment to use that reserve for a specific duration of period
    - You do have the ability to cancel a reservation

- **Reviewing subscriptions based on tags**
    - Tags can be used to mark certain departments within a company
    - You can then filter costs for a particular department based on tags
    - Select a particular `subscription || cost analysis under cost management > under top bars with filter`, select the tag for the department you are looking to analyze costs
    - Tags work better at the level of an individual reseource rather then at the level of a resource group

- **Application insights**
    - Allows integration of application insights service with Azure webapp service can let you track live stats about the website
    - This offers infomation about how the users interact with the application e-g page views
    - If you work on a project through visual studio, there is an option to work with `application insights telemetry`
    - You will be able to see th GET requests, response code and response time
    - Visual studio can send the telemetry application insights data to a resource on Azure
    - On **Azure portal**
        - Create a webap resource from general resources
        - While creating the application insights resource, under monitoring you can add **Application insights** resource
            - you can then search the `application insights resource from resources || live metrics`
            - Before you will see anything on the live metrics you will have to publish your project from visual studio into the webap
            - You can also review performace of the appliation by going to the `application insights resource || performance`. Likewise other usefule features include `failures` and `users`
            - You can also configure `smart alerts `that utilize machine learning 
                - You can go to the `application insights resource || smart detection > settings - you can find smart detection rules`
            - You can also use the `continues expert feature` for data analysis
                - You can go to the `application insights resource || continues resource > add` Where you can specify what type of data you want to send to your storage account
 - **Azure Sentinel Service**
    - This is an Azure Cybersecurity Instrument that can perform SEIM (Security Information Event Management) and SOAR (Security Orchestration Automatied Response)
    - It consists of `connectors` that connects the data to services. For example Azure sentinel service can be connected to the Azure analytics workspace via the connectors
    - Data connectors also allow you to connect to the external services
    - Cybersecurity cycle: Visibility, Analytics, hunting, Incidents, Automation
    - **Azure Security Center** allows you to look at the visibiity and analytics phase of the cybersecurity cycle
    - Azure sentinel allows you to look at all the phases of the cybersecurity cycle - It is a separate resource than Azure sentinel
    - Then there are builtin workbooks and queries
    - You cannot enable the log analytic workspace to Azure Sentinel if you have already linked it with Azure Security center
    - `Steps`
        - Create a log analytics workspace
            - Select resource, location, name (e-g `sentinelworkspace`)
        - Select Azure Sentinel from the main resources || data connectors
            - Among the connectors you can select `Azure Active Directory` Then open up that connector 
            - There you can check mark the type of logs that can be sent to Azure Sentinel from the Azure Active Directory - these can incude sign in logs, audit logs etc
            - Likewise there are multiple connectors that you can link to Azure Sentinel, such as you can add `Azure activity` This will send all of Azure activity logs from Azure subscription to Azure Sentinel
            - Another useful connector is `Security events` from windows VM machines. This will require installation of an agent onto the windows VM
                - If you want to add Security events then you will also have to add that particular VM to Sentinel workspace
                - `Sentinel workspace || virtual machines > Select the VM from list > click connect` The connect here will install an agent on the VM that will manage transfering event logs to the sentinel workspace
                - You can manually look at the security events on a windows Vm by RDP to the VM and then opening `event viewer` > windows log > security there you can see all the recorded security events
        - **Workbooks**
            - If you go to `Azure sentinel resource || data connectors`, Once you select a data connector then under the `next` tab you will see a set of recommended workbooks > view templates - you can visualize a lot of the data that way
            - The benifit of looking at these workbooks is that you do not have to look at the `event viewer` by manually going to a machine

# Azure Security and Identity

- Two main steps: Authentication - verifying your username and password, the next step is authorization for a resource - this occurs at the subscription level and is user specific. 
- Hierarchy -> Resources > Resource groups > Subscriptions > Management groups (group subscriptions together) > Tenant root group
- Note that the permissions get inherited as you move down the hierarchy

## Azure Active directory

- You can create users, groups, roles and administrators
- Azure AD connect lets you connect your on premises users with the Azure active directory
- You can look at all the features available with the Azure active directory onto the Azure active directory pricing webpage
- The premium version of active directory also offers
    - Self-service password reset
    - Conditional access policies
    - Identity governanace, access reviews and privilidged idendity
    - The pricing is per user per month
- **Trust relation b/w Azure active directory and subscription**
- The subscription must trust the user in the Azure active directory, before it will allow access to the resources
- Go to Azure subscriptions > change directory - by default it will select your default directory - however you can creat addtional directories and then you can tell the subscription to trust the new directory
- There are pre-built roles that you can assign. 
    - 4 high level roles: Contributor (Cannot allow permissions for additional roles), owner, reader, User access Administrator
- **Creating a new user in Azure Active Directory**
    - `Active directory || users > new user` Its basically like creating a new e-mail based on the website that is linked to your Azure account
        - While you create the user you give it a username and password
        - The first time log in will require the user to change the password
        - Then you can sign in with this new username from the same Azure portal
            - This user will not be able to see any of the resources
    - You can grant acces to a user to a particular VM
        - Go to the `VM resource || Access control > Add > Add role assignment > reader role > Under select search a user`
        - Once this access has been granted, if you log in with the same user account. You will now be able to see the VM on the dashboard
        - This user will only be able to have limied access to this VM as we previously only permitted `read` level access - for example the user will not be able to see the IP address and subnet associated with the VM - these are separate resources and the user did not have access to these
        - To overcome this you can grant access to the user at the resource group level in addition to the VM level to which the VM is part of
            - Once this is scuccessfull, then when you log in with this user you will now be able to see all the resources listed on the dashboard if you log on to the resource with this user
            - This user will not be able to stop the VM as this user was not granted with `contributor` level access
- **Azure Active Directory roles that can be assigned to the users**
- Note that the Subscription based roles (Role based access control - subscription level) is different from Azure Active directory roles
- Role based access control is more of an administrative level permission as to what level of permissions this user can further assign, where as Azure active directory roles are limited to the resource group access
        - **Dedicating some administrative roles to the users**
            - While logged in as the root admin Azure active directory || Roles and administrators
                - There you can see what role is currently assigned to the account - for the root admin its `global administrator`
                - Then you can select a particular user from the search and assign it a role
                - From the default root user account `Azure AD || Users > find the user > Assigned roles > Add assignment > Select role of User administrator` - This will allow the user to manage the users - including creating new users
                - You can also decide the assignment type elligible vs active (immediate)
- **Azure AD privilidged Identity management** - This feature is only available in Azure premium P2 license
    - The global administrator assigns special permissions to one of the user (prvilidged user) that can intern perform global administrator like functions such as creating or deleting VM ir SQL databases
    - The problem is that If this user moves onto a different department, this privilidged access will need to be removed
    - To avoid this problem, Azure AD privilileged identity management can be used - the benifit it offers is of temporary nature
        - **Features**:Just in time and time-bound and requiring activation of privilidged roles, implement multi-factor authentication to active role and setting notifications when a privilidged role is activated, conduct access reviews
        - this feature can only be implemented with `premium access` of AD
    - Select the `privilidged identity management from general resources` 
        - There are two pathways, one is the active pathway the other is the eligibility pathway
        - In the active pathway, the privilidges go in effect right away - when setting this up you have to options of 
            - `Privilidged identity management || AZure AD roles under manage - THis leads to the default directory homepage || settings > Search for the user administrator role > edit`
                - On this page you will see the tabs of Activation, Assignment and Notification
                    - **Activation** - you can select where multi-factor identification is to be applied
                    - You can select max activation duration - this is the number of hrs the user with this privilidge have to perform certain tasks once activated
                    - **Assignment** - Here you can specify whether you want it to be a permanent elligible assignement or a permanent active assignment OR you can specify an expiration time (e-g expire after 15 days). The elligible option offers an additional layer of security as it does not go into effect right away, the user who is rendered this elligibility will have to perform the activation from his/her account
                    - **Notification** - Who should be notified when the role has been assigned
            - `Azure AD || users > search the user > click the user || assigned roles` - here you will be able to see the assignment
            - **Adding a new assignment**
                - `Azure AD || users > search the user > click the user || assigned roles > Add assignment > select user administrator role`
                    - Select assignment type as eligible - It will have a start and end date - This date range is only when this particular user will be eligible to take up this role
                    - Next if you go inside this user's account by logging in you will see that this user will now be able to activate this elligibility
                        - Home page after logging in, Azure active directory || all users - wont be able to add users yet
                        - search `privilidged identity management || my roles > there you can see the eligibility to take up the user administrator role`
                    - The elligibility will be available only during the specified dates, during which the user can activate this additional privilidge and once the privilidge is activated there will only be a set number of hrs during which the user can perform the task. However, within the eligibility period. The user will be able to reactivate for this role multiple times
- **Azure Reviews**
- Review to access whether the latest user roles are appropriate for group memberships and role assignments
- Reviews can be conducted for:
    - Security group members
    - Users assigned to an application
    - Azure AD role
    - Azure resource role
- For reviews in the AD and resource roles, the reviews must be conducted with the privilidged identity management
- Premium tasks
    - Member and guest users who are assigned as reviewers
    - Member and guest users who perform a self-review
    - Group owners who perform an access review
    - Application owners who perform an acces review
- P2 license is not required for uses with Global Administrator or User Administrator roles
- Go to the `default directory, privilidged identity management || access reviews > new`
    - There you can decide the date range and the frequency of reviews
    - Then you can select the role for which you are performing a review e-g review of all the user accounts that have `user administrator` privilidges
    - you can also select `reviewers` these are the user accounts that can conduct reviews - If you are doign the review yourself then you can add your user account
    - It will display a list of users for each of the user, you can click audit to see the logs
    - Here you will have to opportuity to `approve or deny` which will continue or take away the privilidge, e-g in this case it is user Administrator
    - Then if you go back on to `default directoy || access review > you will see an access review - click it to modify it`. 
        - If you have dinied the user then you will first have to `stop` the access review and then click `apply`

- **Discover Resources**
- `Privilidged identy management || Azure resources`
    - It will show the subscriptions e-g Test environment
    - Going to the subscription homepage also gives you a summary of Identity management activities
    - Here you will see various roles for example a basic reader role - this will let you modify it
        - You will have to options of setting up activation, assignment and notification settings

- **Azure AD identity protection**
- It is to detect problems logins
- It will analyze the day to day logins of a particular user and then determine any risk to the user as well as the organization
    - For example the login is coming form an ip address that is not really known
    - It can categorize the risk as high medium or low
    - Then you can also trigger an action for example block access, but trigger a password change
    - Utilization of these features like user risk policy and sign in risk policy require Azure AD premium subscription

- **Identity governance**
    - Search for a`ccess review in general resources || access reviews > New access review`
        - Here you can select teams and `groups` and select `scope` like all users etc
        - Next in the `reviews` tab, then you can select reviewer, select the admin account and select the frequency with which to perform the review. Then likewise you can select the days and the start date - you can also specify frequency like weekly review, monthly review etc
        - Asign it a name and then click create
        - You will get an email notification when the reciew is complete - through which you can take actions like approve or deny an access
        - You can see information about a user like their last day of signin - there will be automatic recommendations and if you agree with them you can accept of deny the recommendations or approve or deny separately

- **Azure Security Center**
    - You can go to this resource 
    - Gives an overview of the security posture of the resources
    - It provides an overall security  score
    - It scans the resources and gives you recommendations
    - Some recommendations are free, but for some more advanced recommendations you will have to pay
         - `Security center || overview || pricing` and settings will enable standard tier
         - Navigating to this tier you can see whats available for free and whats available with costs e-g with standard tier
         - E-g regulatory compliance dashboard and reports will not be available for free
         - Continues assessment and security recommendations as well as Azure secure score are fee features


