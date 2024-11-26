# Ansible Openstack to install guacamole

This repository will use Ansible to create an instance in Openstack, install guacamole and copy configuration files. It does not use any external tool (like Terraform or OpenStack Heat), only Ansible.

## Requirements

In order to run this code you need to first install some tools into your computer.

1. First, you need ansible to be installed. There are several methods to install ansible, one of them being `pip`:

	```sh
	pip install ansible
	```

1. You need the [openstack.cloud collection](https://docs.ansible.com/ansible/latest/collections/openstack/cloud/index.html). Type this command:

	```sh
	ansible-galaxy install -r requirements.yaml
	```

1. The latest version of [openstacksdk for Python](https://pypi.org/project/openstacksdk/) is also required:

	```sh
	pip install openstacksdk
	```

1. Finaly, you need to source the OpenRC file corresponding to the project the infrastructure will be deployed to. The file can be downloaded from the [API access](https://pouta.csc.fi/dashboard/project/api_access/) page of the Pouta interface. See [Pouta access through OpenStack APIs](https://docs.csc.fi/cloud/pouta/api-access/) for more reference.

## Explore the repository variables

In `group_vars/all.yml` you will see the following variables:

* `number_instances`, number of servers to deploy.
* `instance_name`, prefix for the name of the servers
* `os_image`, image to install in the servers
* `flavor`, size/flavor of the servers to be created
* `internal_ips`, IP ranges that will be able to connect to port 8080/HTTP and 22/SSH.
* `guacamole_version`, As this of writing, latest guacamole version is 1.5.5.
* `guacamole_username`, Set the username to connect to guacamole web.
* `guacamole_password`, Set a MD5 password. You can run `echo -n <your_password> | openssl md5` to generate one

_NOTE_:

In `templates` you can find `user-mapping.xml.j2`. It needs to be edited according your configuration. You can add more users if you wish, replace `MyFirstMachine` and `MySecondMachine` with more explicit names and replace `0.0.0.0` by the IP of the machines where you want to connect.

The default protocol in this file is `vnc`. You can use different protocols like `rdp`, `ssh`, ...

More information in the official documentation: https://guacamole.apache.org/doc/gug/configuring-guacamole.html#user-mapping-xml

## Launch the playbook

In order to launch the playbook:

```sh
ansible-playbook main.yaml
```

It will prompt for:

* The key that you will use to SSH to your instance
* The network that you will use (Use this command to list the different networks: `openstack network list`)

* Other option is to specify the values in the command line:

```sh
ansible-playbook \
	-e key_name=xxxxxx-key \
	-e network=project_200xxxx \
	main.yaml
```

After ansible has run you should have 1 server running guacamole listening in port 8080/HTTP (`http://`) and 22/SSH.

If you change the variables and rerun `ansible-playbook`, ansible will automatically apply the changes. For example, if your current ip (`curl ifconfig.me` will give you your IP) does not belong to the default range of IPs, you will need to change the range of IPs to include yours. As a side note, it is very recommended to always have a narrow range of IPs that are allowed to connect to the port 22/SSH, it adds a good extra layer of security.

## Destroy the cluster

In order to destroy the servers, one must simply run:

```sh
ansible-playbook \
	-e key_name=xxxxxx-key \
	-e network=project_200xxxx \
	-e state=absent
	main.yaml
```
