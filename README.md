# Setting up Debian/Ubuntu container pipeline
This project help create a jenkins server on a Debian/Ubuntu server.

## Creating the VM
The creation of the VM has been automated by [Vagrant](https://www.vagrantup.com/).

Use Vagrant to create the VM:

    vagrant up

Remember to chage the interface for the public ip on the Vagrant file:

    config.vm.network "public_network", type: "dhcp", bridge: "<network_interface>"

## Provvisioning the VM
The install process behind Jenkins has been automated via the [Ansible](https://www.redhat.com/en/technologies/management/ansible).

All the Ansible automation can be found in the file `playbook.yaml`.

It's possible to run the playbook again against the vm by using the inventory Asnible created by Vagrant:

    ansible-playbook playbook.yaml -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory

## Launching the docker-compose
Once the machine is ready, it's possible to launch the docker-compose on it

    ansible-playbook playbook-compose.yaml -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory

## Compiling the container in the remote machine
Once the machine is ready, it's possible to compile the container on it

First create a file with all the necessary variables and specify it's location via the variable containers_config.
The file must contain the following entries:

    login: <boolean: wheter to login into the container registry>
    logout: <boolean: wheter to logout from the container registry>

    container_registry:
      server: <string: container registry login entrypoint>
      username: <string: container registry username>
      password: <string: container registry password>

    containers:
      - image:
          name: <string: name of the image>
          push: <boolean: whter to push it onto the container registry or not>
        dockerfile: 
          name: <string: the dockerfile name>
          location: <string: the dockerfile location>
          context: <string: the location of the Dockerfile contex>

The variable `containers` is a list of all the containers that have to be compiled, it's possible to add as many container as desired.

When the variable `push`is set to `true` either make sure that the target machine is logged into the container registry.
Alternatively the playbook can login into the container registry by setting the variable `login` to `true`.

If `logout` is set to `true`, after it's exexution, the playbook will logout from the container registry. Even if other taks fail, the logout will still be carried out.

To launch the playbook:

    ansible-playbook playbook-compile.yaml -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory

## Hiding the container registry credentials
Create a file called `container_registry.enc` and populate it with the container credentials:

    container_registry:
      server: <container registry login entrypoint>
      username: <container registry username>
      password: <container registry password>

The default server for docker hub is: `https://index.docker.io/v1`

Encrypt the file:

    ansible-vault encrypt container_registry.enc

Use the encrypted file when launching the playbook:

    ansible-playbook playbook-compile.yaml -e @container_registry.enc --ask-vault-pass -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
