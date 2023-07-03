Satellite Capsule Deployment
=========

This role configures and deploys a Satellite Capsule. The role will check of suitability of the host designated to become a Capsule by checking required system facts. This includes checking host hardware requirements, ensuring the correct packages and repos are present, configuring storage, configuring firewall, registering the host to satellite, deploying custom SSL certificates.
This role is designed to be reusable, so many variables are stored in the appropriate files, instead of being hard-coded directly into the `/tasks/main.yml` file. Run this role on the host to become a capsule, ensuring this host can be used soley for this purpose. This role can also be edited to create multiple capsule hosts.

The role runs on the hosts designated to become capsule servers, but it delegates tasks to localhost and the Satellite server.

Requirements
------------

Satellite should be set up, ready for a capsule to be joined and configured. This includes having SSL certificates for the capsule(s). 
Sudo access on the remote host (to become the capsule) is required.
This role uses `inventory_hostname` to get the fully qualified domain name (fqdn) to use as a variable in the role, so please ensure that the inventory allows this.

Role Variables
--------------

Variables related to the satellite capsule are located in `defaults/main.yml`. This includes: credentials and details related to the capsule and the central satellite server, storage device configuration details, firewall ports to open, repository names, Hashicorp vault details etc.
The variable `satellite_curl_cmd` must be generated on the Satellite server and copied into the variable file prior to running the role. 

Check your current disk storage prior to role deployment and edit the disks that are used in the `tasks/main.yml` file, so that the correct disks with the required space available are used.


Dependencies
------------

Other roles called in this role: `[custom config role]`, `redhat.rhel_system_roles.storage`
This role needs to use at least three hosts to run commands: the localhost with Ansible installed to run the role from, a central Satellite server to run some commands to set up the capsule, and the host(s) that are designated to become Satellite Capsule(s).

Example Playbook
----------------


    - hosts: capsule
      roles:
         - install_satellite_capsule

License
-------

BSD

Author Information
------------------
Elise Elkerton elise@redhat.com