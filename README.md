## Usage

* First you need to adapt Ansible variables in the
  `inventory/testing/group_vars/all.yml`. They are shared between
  all individual playbooks

To be able to work with openstack an `openstacksdk` python module should be
installed on the local machine, where ansible will be triggered.

A configuration of the cloud should be added into the
`$HOME/.config/openstack/clouds.yaml`

~~~~
clouds:
  otc_demo:
    auth:
      auth_url: https://iam.eu-de.otc.t-systems.com:443/v3
      project_name: eu-de #required, since otherwise some APIs are not working
      user_domain_name: OTC00000000001000000xxx
    interface: public
    identity_api_version: 3
~~~~

`$HOME/.config/openstack/secure.yaml`

~~~~
clouds:
  otc_demo:
    auth:
      username: MY_USER_NAME
      password: MY_PASS
~~~~

### VPC creation

After the global variables are adapted the following can be done for
the VPC creation:

  `ansible-playbook -i inventory/testing playbooks/infra/main.yaml`

This command will create VPC (network, default subnetwork, router, keypair).
The new keypair will be saved in $HOME/.ssh

Note: due to the issue in shade/openstacksdk<0.14.0 library the SNAT is not
enabled on the VPC. After the playbook is complete please enable it manually

### Bastion Host

The bastion (jump) host can be created with the following command:

  `ansible-playbook -i inventory/testing playbooks/bastion/main.yaml`

This starts one server instance (ECS) and assigns floating IP to it.

After this is done and the public IP is known the SSH configuration can be
extended to access cloud through this host:

Assuming Public IP is 1.1.1.1, in the `./ssh/config` (0x600) please add the
following:

~~~~
# Bastion host
Host otc-bastion
  HostName 1.1.1.1
  User linux
  IdentityFile ~/.ssh/my-KeyPair.pem
  ControlMaster auto
  ControlPersist 5m

# App-cluster nodes access
Host 192.168.110.*
  ProxyCommand ssh -W %h:%p otc-bastion
  IdentityFile ~/.ssh/my-KeyPair.pem
~~~~

In addition for the simplification of the further infrastructure usage it is
recommended to extend inventory/.../hosts with proper IP address

### LAMP Infrastructure

Infrastructure of the LAMP can be provisioned:

  `ansible-playbook -i inventory/testing playbooks/lamp_infra/main.yaml`

This starts 2 server instances (ECS) for DB and Web and assigns proper security
groups.

After this step is completed a "configuration management" can be started.
IP addresses of the db and web servers should be stored in the inventory.

### LAMP Configuration

LAMP stack servers can be configured:

  `ansible-playbook -i inventory/testing playbooks/lamp_infra/main.yaml`

This installs required packages (mysql, php, apache), copies the code from
app repo and deploys it. To check it on the webserver execute
`curl http://localhost/index.php`
