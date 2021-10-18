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
    - Once created you can run the SQL server management studio
        - The server name is the webaddress of the SQL server you created and you use the login credentials in order to connect

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