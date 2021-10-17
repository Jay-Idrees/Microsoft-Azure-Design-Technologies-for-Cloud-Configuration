# Azure Design Technologies for Cloud Configuration

I have created this repository for my self learning and reference on my journey to become an expert Cloud Architect

## Deployment cycle for application

- **Create a VM and a virtual network**
    - Specify inbound ports - The RDP is usually automatically selected, you can also add port 80
    - You will get an OS level disk as well as a temporary D drive, you can also add an additional disk for database
    - Select default encryption at rest
    - The VM can have a public IP address if it is hosting a web application that will be accessed by the public
    - Create a **virtual network** as you creat a VM 
        - Primary virtual network interface - has its own IP address, used for internal communication
    - An NSG is also created the controls permissions for the ports