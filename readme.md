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
    - When you use AZURE SQL, you do not have to manually manage the underlying VM hosting the database, which means that you do not have to install the Azure SQL service engine, no security or patching of the database
    - You can create the SQL database from the main resources as usual
        - Specifications
            - Create a server - This is for logging into the SQL database server. It has 2 components, the server that hosts the database and the databse itself