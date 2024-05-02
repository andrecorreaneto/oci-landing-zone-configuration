# SaaS Landing Zone Architecture POD Model Blueprint

## 
### Overview

This repository contains the configuration files for OCI SaaS (Software-as-a-Service) Vendor Landing Zone Architecture Pod Model blueprint based on OCI Core Landing Zone (formerly known as OCI CIS Landing Zone) principles. This Blueprint is designed for SaaS vendors and managed service providers to onboard OCI in a streamlined manner that aligns with the CIS OCI Benchmark and best practices for SaaS vendors and managed services vendors.  The Pod model is designed so that each customer gets a copy or application stack.  This pattern can be seen in SaaS and managed services industries where each customer's environment is independent of another, and the only parts shared are the management plane. 

The following architecture diagram depicts a single tenancy blueprint, separating the management plane (red color) from the customer plane (blue and grey) compartments.

### Enclosing Compartment

The enclosing compartment (grey) represents the environment of the SaaS offering or a single SaaS offering.  The former would separate the production environment from the test environment, and the latter would be SaaS Application 1 and SaaS Application 2. It achieves this by representing the scope for OCI IAM and guard rails used in the SaaS respective environment.  


### Management Plane

The management plane compartment (red) is designed to house the OCI resources that provide the functions and processes that determine which path the data and OCI resources that make up the application are managed.  A management plane is similar to a cloud provider's control plane and they are shared resources that are used by all application stacks running in the customer plane compartment (blue).  There are three types of functions and processes defined in this blueprint.

#### Management Groups

**Security Administrators:** manage security services and resources including Vaults, Keys, Logging, Vulnerability Scanning, Web Application Firewall, Bastion, Service Connector Hub.
**Network Administrators:** manage OCI network family, including VCNs, Load Balancers, DRGs, VNICs, IP addresses.

**Shared Services Administrators:** 

#### Networking Compartment
The network compartment (green) is designed to house the shared Virtual Cloud Networks (VCNs) used to secure and route network traffic to and from the application stack including the 


### Customers Plane Compartment


#### Customer Groups

**Customer Administrators:** manage application stacks deployments and related resources in the Customer Compartment.  OCI resources include Compartments, Network, Compute images, Database OCI Functions, Kubernetes clusters, Streams, Object Storage, Block Storage, File Storage, Keys, and Budgets.

**Snowflake Customer Administrators:** manage application stacks deployments and related resources in the Snowflake Customer Compartment.  OCI resources include Compartments, Network, Compute images, Database OCI Functions, Kubernetes clusters, Streams, Object Storage, Block Storage, File Storage, Keys, and Budgets.

### Tenancy Level

#### Tenancy Level Groups

**IAM Administrators:** manage IAM services and resources including compartments, groups, dynamic groups, policies, identity providers, authentication policies, network sources, tag defaults. However, this group is not allowed to manage the out-of-box Administrators and Credential Administrators groups. It's also not allowed to touch the out-of-box Tenancy Admin Policy policy.

**Credential Administrators:** manage users capabilities and users credentials in general, including API keys, authentication tokens and secret keys.

**Cost Administrators:** manage budgets and usage reports.

**Auditors:** entitled with read-only access across the tenancy and the ability to use cloud-shell to run the cis_reports.py script.

**Announcement Readers:** for reading announcements displayed in OCI Console.

#### Tenancy Level Resources

**Oracle Cloud Guard:** activity, configuration, and threat detector recipes are defined at the tenancy level to examine your resources for security weaknesses and to monitor operators and users for risky activities.

**Policies:**

**Events:** 

**Budgets:**

**Subscriptions:**

![SaaS-pod-architecture](images/SaaS-pod-architecture.png)

[Click here](./images/SaaS-pod-architecture.drawio) to download the drawio version.

## Configuration Files

The blueprint is described in configuration files (JSON documents) that can be deployed as-is or changed based on specific requirements. Changes must be done following the requirements and specifications of the respective underlying Terraform module. The idea behind describing Landing Zones as JSON documents is making them fully customizable and declarable in a standard and widespread document format (JSON), while leveraging Terraform's Infrastructure as Code paradigm.

The configuration files are organized in different folders, according to the broader area they belong. The folder structure is the following:

- mgmt-plane: a top level folder for management plane, management groups and Root level resources.
    - iam: 
        - [iam_config.json](./mgmt-plane/iam/iam_config.json), with the IAM configuration, including compartments, groups and policies.
    - governance: 
        - [budgets_config.json](./mgmt-plane/governance/budgets_config.json), with budgets configuration.
    - security: 
        - [cloud_guard_config.json](./mgmt-plane/security/cloud_guard_config.json), with Cloud Guard configuration.
        - [scanning_config.json](./mgmt-plane/security/scanning_config.json), with scanning configuration.
    - observability: 
        - [observability_config.json](./mgmt-plane/observability/observability_config.json), with logging, streams, notifications, events, alarms and service connector hub configurations.
    - network: 
        - [network_initial_config.json](./mgmt-plane/network/network_initial_config.json), containing the initial network configuration, with VCNs, subnets, route tables, security lists, network security groups, VCN gateways and DRG.
        - [network_post_firewall_config.json](./mgmt-plane/network/network_post_firewall_config.json), adding route table rule to Network Load Balancer private IP and DRG attachments to the initial network configuration. This configuration is to be executed after the Firewall appliance and OCI Network Load Balancer are provisioned.
        - [network_post_each_customer_config.json](./mgmt-plane/network/network_post_each_customer_config.json), adding route table rules and DRG route table distributions for each on-boarded customer. This configuration is to be executed after each customer onboarding.
    - firewall: [firewall_config.json](./mgmt-plane/firewall/firewall_config.json), with firewall appliance and OCI Network Load Balancer configurations.

- customers: a top level folder for each customer configuration.
    - customer1: [customer1_config.json](./customers/customer1/customer1_config.json), with all resource configurations for customer 1, including IAM and networking.

## How to Deploy

The blueprint (or any customization) configurations can be deployed in separate stacks. A stack is a set of configurations that are managed by a single Terraform execution and thus reflected in a single Terraform state file. The decision factor for how many stacks depends on how the organization wants to manage the environment, especially which roles are expected to deploy which parts of the blueprint. 

As a rule of thumb, it is advisable to separate management place stacks from customer stacks. They must be executed in the following order:

1. **Management plane foundational stack**: assembled with *iam_config.json*, *budgets_config.json*, *cloud_guard_config.json*, *scanning_config.json* and *observability_config.json* configuration files. It can be further split based on specific requirements.
2. **Management plane network stack**: composed by *network_initial_config.json*, later replaced by *network_post_firewall_config.json* after firewall provisioning, and later replaced by *network_post_each_customer_config.json* after each customer stack.
3. **Management plane firewall stack**: composed by *firewall_config.json*, after which the Management plane network stack must be updated with *network_post_firewall_config.json*.
4. **Customer stacks**: composed by *customer?_config.json*, after which the Management plane network stack must be updated with *network_post_each_customer_config.json*.

These configuration files are designed to be used as templates, owned and modified at will by customers that are deploying the model. 

### The OCI Landing Zones Orchestrator

The [OCI Landing Zones Orchestrator](https://github.com/oracle-quickstart/terraform-oci-landing-zones-orchestrator) is a convenience tool for deploying JSON-based landing zones. It also supports YAML documents or Terraform tfvars files with HCL (Hashicorp Language) objects. The only requirement is that the documents/HCL objects are defined according to the requirements and specifications set forth by the OCI Landing Zone core modules, that are available in the following repositories:

- [Identity & Access Management](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam)
- [Networking](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-networking)
- [Governance](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-governance)
- [Security](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-security)
- [Observability & Monitoring](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-observability)
- [Secure Workloads](https://github.com/oracle-quickstart/terraform-oci-secure-workloads)

Next we show how the whole blueprint is deployed as a composite configuration of the aforementioned stacks.

### Management Plane Foundational Stack Deployment

The foundational stack joins IAM, Security and Observability resources in a single configuration. As mentioned before, it can be further split depending on deployment requirements. 

Click the button below to deploy the stack with the OCI Landing Zones Orchestrator. The variables are all pre-filled with configuration files available in this repository, along with all required dependencies, stored in an OCI private Object Storage bucket.

[![Deploy_To_OCI](./images/DeployToOCI.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/oracle-quickstart/terraform-oci-landing-zones-orchestrator/archive/refs/heads/urls-dep-source.zip&zipUrlVariables={"input_config_files_urls":"https://raw.githubusercontent.com/oracle-quickstart/terraform-oci-landing-zones-orchestrator/main/examples/vision/iam/config/iam-config.json","url_dependency_source_oci_bucket":"SaaS-terraform-runtime-bucket","url_dependency_source":"ocibucket","url_dependency_source_oci_objects":"bootstrap/compartments_output.json","save_output":true,"oci_object_prefix":"iam/output"})

![SaaS-pod-architecture-mgmt-plane-foundational](images/SaaS-pod-architecture-mgmt-plane-foundational.png)

### Management Plane Network Stack Deployment

### Management Plane Firewall Stack Deployment

### Customer Stacks Deployment

