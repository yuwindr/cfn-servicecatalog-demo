# CloudFormation Templates for TP Penetration Testing Labs

## Deployment Steps
1. Deploy `vpc-stack.yaml`. This will create the VPC where the EC2 instances will be deployed into. Take note of the stack name (will be used in `instances-stack.yaml`).
2. Upload `instances-stack.yaml` to an S3 bucket and take note of the S3 path (will be used in `service-catalog-stack.yaml`).
3. Deploy `service-catalog-stack.yaml`. This will create the Service Catalog resources (Portfolio & Product) to launch `instances-stack.yaml`. Remember to key in the S3 path to the `instances-stack.yaml` template as part of the stack parameters.
4. In the Service Catalog console, go to the deployed Portfolio and assign users, groups or roles that should be allowed to launch the product.

## Components
There are 3 CloudFormation templates in this repository:
1. `vpc-stack.yaml` - deploys a VPC where the instances will be deployed into
    - Resources and configurations:
        - VPC with CIDR `10.0.0.0/16` with 1 public subnet (`10.0.0.0/24`) and 1 private subnet (`10.0.1.0/24`)
        - 1 Internet Gateway (IGW)
        - 1 Route Table for public subnet (with path to IGW) & 1 Route Table for private subnet
        - No NAT Gateways
        - Kali Linux instance will be deployed into the public subnet, and the web server instance will be deployed into the private subnet
    - Parameters needed:
        - `VPCName` - name of the VPC to be deployed

2. `instances-stack.yaml` - deploys 1 web server instance and 1 Kali Linux instance and security groups
    - Resources and configurations:
        - Web server SG rules:
            - 1 inbound rule that allows all TCP from Kali Linux SG
            - 1 outbound rule that blocks all outbound access
        - Kali Linux SG rules:
            - Inbound rules that allow ports 80, 8443 (TCP) and 8443 (UDP) from the parameter InstanceConnectLocation (e.g. 0.0.0.0/0)
            - 1 outbound rule that allows all TCP to web server SG
    - Parameters:
        - `KeyName` - name of an existing EC2 key pair
        - `InstanceType` - lists of all instance types available (can be restricted further via Service Catalog Template Constraint)
        - `InstanceConnectLocation` - The CIDR address range that is allowed to connect to the Kali Linux instance
        - `WebServerAmiId` - AMI for the Web Server instance
        - `KaliLinuxAmiId` - AMI for the Kali Linux instance
        - `NetworkStackName` - Name of the VPC CloudFormation stack. This value will be used to read the subnet IDs created in the baseline VPC stack, so that all instances are deployed into that VPC
3. `service-catalog-stack.yaml` - deploys Service Catalog resources
    - Resources and configurations:
        - Service Catalog Portfolio
        - Service Catalog Product that points to the S3 path where `instances-stack.yaml` template is stored
        - Template Constraint that allows only certain t3 instances to be chosen
        - IAM Role that is used as the Service Catalog Launch Constraint
    - Parameters:
        - `TemplateS3Bucket` & `TemplateS3Path` - S3 location where the `instances-stack.yaml` template is stored
        - `RoleName` - name of the IAM role to be created
        - Portfolio details - Name, Description, Provider
        - Product details - Name, Description, Distributor, Owner, SupportDescription, SupportEmail, SupportUrl
    