# ansible-3tier-app
This project creates a CI/CD pipeline using Ansible Tower Workflow Template to provision 2 environments and deploy the 3 Tier Application code to both environments.
The QA environment is using a private OpenStack cloud and the PROD environment is using AWS.

## The QA environment contains 3 playbooks:
  * Provision_OSP.yml: Provisions OpenStack infrastructure and instances as required by the 3 Tier Application
  * Configure_3TA_OSP.yml: Deploys and configures the 3 Tier Application
  * Cleanup_OSP.yml: Removes OpenStack instances used by 3 Tier Application

The OpenStack environment is behind a firewall and as such the inventory is composed of the jumphost only. The required ssh configuration to allow connectivity by means of the jumphost is provided in ssh.cfg file.

### Provision_OSP.yml
* This playbook takes all its variables from osp_vars.yml file and creates all the OpenStack infrastructure needed by the 3 Tier Application:
  * OSP Networks
  * OSP SSH Keypair
  * OSP Security Groups
  * OSP Nova Flavor
  * OSP Instances

### Configure_3TA_OSP.yml
* This playbook uses in-memory dynamic inventory to provision 3 Tier Application:
* HAProxy on the frontend server to load balance between application servers.
* Tomcat on the application servers.
* PostgreSQL on the database server.

### Cleanup_OSP.yml
* This playbook is used in the even that the Provision_OSP.yml or Configure_3TA_OSP.yml playbooks fail and it removes:
  * OSP Instances
  * OSP Nova Flavor
* This playbook takes all its variables from osp_cleanup_vars.yml file.

## The PROD environment contains 2 playbooks:
  * Provision_AWS.yml: Provisions OpenStack infrastructure and instances as required by the 3 Tier Application
  * Configure_3TA_AWS.yml: Deploys and configures the 3 Tier Application

### Provision_AWS.yml
* This playbook takes all its variables from aws_vars.yml file and creates all the AWS infrastructure needed by the 3 Tier Application by requesting an Opentlc predefined service called PROD_THREE_TIER_APP.
* The playbook is using Ansible Vault to provide secure credentials for authenticating to Opentlc to request the service.
* The playbook wait 10 min for the environment to be provisioned.

### Configure_3TA_AWS.yml
* This playbook uses Ansible Tower AWS EC2 dynamic inventory feature to provision 3 Tier Application:
  * HAProxy on the frontend server to load balance between application servers.
  * Tomcat on the application servers.
  * PostgreSQL on the database server.
* The AWS EC2 dynamic inventory creates groups in the format tag_AnsibleGroup_<group>, where group is replaced by the actual groups defined in the service during provisioning: apps, appdbs, frontends, bastions
