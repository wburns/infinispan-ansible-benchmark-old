Infinispan Ansible Hyperfoil Benchmark
=========

Manages starting a cluster of Infinispan Server container nodes along with uploading a cache configuration to be performance tested with Hyperfoil with automatic provisioning of a controller and agent nodes.

Requirements
------------

Ansible controller node requires ansible and jmespath to be installed
Ansible managed nodes require sshd and public key
- Infinispan Server nodes will install podman if a sudoer, otherwise must be installed manually
- Hyperfoil nodes will install latest open jdk 17 if a sudoer, otherwise must be installed manually

Roles
------------
server: Manages starting and stopping Infinispan servers. Utilizes the `server` inventory group to determine nodes
hyperfoil_controller: Manages starting and stopping the hyperfoil controller node. Requires a singleton in the `hyperfoil_controller` inventory group. It is recommended for this to be the localhost `ansible_connnection=local`.
hyperfoil_agent: Manages deployment and running of Hyperfoil agent nodes. Utilizes the `hyperfoil_agent` inventory group to determine nodes.

Example Inventory
------------
Here is an example inventory.ini file defining 3 Infinispan server nodes, a local hyperfoil contoller and 2 hyperfoil agent nodes
```ini
[all:vars]
user=benchmark
pass=benchmark

[server]
host[1-3]

[hyperfoil_controller]
controller ansible_connection=local

[hyperfoil_agent]
host[4-5]
```

Playbooks
--------------
* benchmark.yml - Overarching playbook that encompasses all of the ones below. Will start the Infinispan server nodes, Hyperfoil controller, start a Hyperfoil run on the agents, shutdown the Hyperfoil controller and shutdown and remove the Infinispan container
* server.yml - Starts the Infinispan server container nodes with host networking enabled. Also uploads the `cache_name` named cache with the configuration in the `cache_file` file.
  * Default `cache_name` variable is `benchmark` and `cache_file` is `files/cache.xml` if you wish to change the name or xml file.
  * This playbook can also be ran with `-e operation=shutdown` to shutdown the inventoried servers
* hyperfoil.yml - Playbook that encompasses starting the hyperfoil controler, agents and starts a run and then shuts them all down. Note the server must already be running and configured in the server hosts file.
* hyperfoil_controller.yml - Sets up the hyperfoil controller.
  * This playbook can also be ran with `-e operation=shutdown` to shutdown the controller and agents if running
* hyperfoil_agent.yml - Runs a hyperfoil jobs, requires the controller and Infinispan servers to already be running.
  * Utilizes the `test_name` variable to read a templated file with extension `yaml.j2` in the `benchmarks` directory. Default is `hotrod-benchmark`

License
------------

Apache License, Version 2.0
