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
            - You can also configure smart alerts that utilize machine learning 