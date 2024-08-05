## Management Plane Foundational Stack Deployment

It assembles bootstrap, IAM, governance, security and observability configuration files in a single stack. 

### Typically Deployed By

Management plane administrators.

### Deployment Sequence

1. **Mgmt Plane Foundational - IAM, Logging, Governance (this stack)**
2. [Mgmt Plane Networking 1st stage - Mgmt Plane VCNs](./MPLANE-NETWORKING.md#network-stack-1st-stage)
3. [Mgmt Plane Networking - Firewall](./MPLANE-FIREWALL.md)
4. [Mgmt Plane Networking 2nd stage - Network routing post firewall deployment](./MPLANE-NETWORKING.md#network-stack-2nd-stage)
5. [Customer Onboarding](./CUSTOMER-ONBOARDING.md)
6. [Mgmt Plane Networking 3rd stage - Network routing post customer onboarding](./MPLANE-NETWORKING.md#network-stack-3rd-stage)

**Stacks #5 and #6 must be repeated for each NEW customer onboarding.**

### Stack Configuration

Input Configuration Files | Input Dependency Files | Generated Output
--------------------------|------------------------|------------------
[bootstrap.json](../mgmt-plane/bootstrap/bootstrap.json), [iam_config.json](../mgmt-plane/iam/iam_config.json), [budgets_config.json](../mgmt-plane/governance/budgets_config.json), [cloud_guard_config.json](../mgmt-plane/security/cloud_guard_config.json)\*, [security_zones_config.json](../mgmt-plane/security/security_zones_config.json), [scanning_config.json](../mgmt-plane/security/scanning_config.json), [observability_config.json](../mgmt-plane/observability/observability_config.json) | None | iam/output/compartments_output.json

\* cloud_guard_config.json is not automatically included in the stack below. When creating the stack, add *https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/security/cloud_guard_config.json* to the set of configuration files only if a Cloud Guard target does not exist in the Root compartment.

### Stack Creation

Click the button to create the stack with all variables pre-filled:

[![Deploy_To_OCI](../images/DeployToOCI.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/oracle-quickstart/terraform-oci-landing-zones-orchestrator/archive/refs/heads/main.zip&zipUrlVariables={"input_config_files_urls":"https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/bootstrap/bootstrap.json,https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/iam/iam_config.json,https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/governance/budgets_config.json,https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/observability/observability_config.json,https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/security/scanning_config.json,https://raw.githubusercontent.com/andrecorreaneto/oci-landing-zone-configuration/test/mgmt-plane/security/security_zones_config.json","url_dependency_source_oci_bucket":"isv-terraform-runtime-bucket","url_dependency_source":"ocibucket","save_output":true,"oci_object_prefix":"iam/output"})

### What Gets Deployed

![isv-pod-architecture-mgmt-plane-foundational](../images/foundational.png)