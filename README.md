Squid Proxy Configuration Role ReadMe
=========
### Description
This role can be used to configure a Squid proxy, a caching and forwarding http and ftp proxy. The inventory file must identify the host to become the Squid proxy. The tasks/main.yml will join Satellite (if applicable), check that Squid is installed, (and install it if not), start and enable Squid, allow Squid through the firewall, create the Squid configuration file on the host (/etc/squid/squid.conf) using the templates/squid_template.j2 file, join IPA using redhat.rhel_idm.ipaclient role, create IPA services and service account, create keytab files and password file, check Squid configuration and restart Squid. The proxy includes Kerberos and LDAP authentication. Handlers are stored in the handlers/main.yml file, to reload firewalld and restart the squid service after configuration.
### Testing The Squid Proxy and Authentication

The success of the configuration can then be verified with squid -k check followed by systemctl status squid, then using the curl command to access a web page.

If testing authentication, after following the above steps, set the environment vairable with export https_proxy=http://"{{ inventory_hostname }}:3128" and then test Kerberos with curl -l --proxy-negotiate -u : https://wwww.google.com or test LDAP with curl -l --negotiate -u : https://wwww.google.com. To obtain and cache a Kerberos granting ticket, use kinit [your_username] for the user, or kinit -k 'HTTP/{{ hostname }}@{{ domain | upper }} -t /etc/squid/krb5-proxy.keytab for the principal. To check keytab files, use klist -kt /etc/squid/krb5-proxy.keytab. To check all tickets, use klist.
Requirements

Ansible and packages required by Ansible must be installed, otherwise the "Join Satellite" option in the config file must be included.

Role Variables
------------------
Variables specific to the host are configured in the group_vars/all/host_config.yml file. Variables specific to Squid are configured in the defaults/main.yml file. An ansible vault is used to store passwords, at group_vars/all/vault.yml. Read below for more information on the variables used and how to change them.
group_vars/all/host_config.yml
Domain to authenticate

Update the name of your domain to be authenticated here, it will populate other variables requiring domain.
IPA Authentication

lab_admin_principal and the ipa_server are defined here. These are used in the tasks/main.yml file. lab_admin_password is stored in an ansible vault.
Kerberos Authentication

Realm must be in uppercase. children is set to 10 anq keep_alive is on.
LDAP Authentication

children is set to 10, list the ldap authentication basic realm proxy here, as well as the search base, bind account and ipa server details. Check search base and bind_account details are correct.
Cache Peer

Add other Squid proxies for the configured Squid proxy to communicate with. Additional proxies can be added here, by starting a new list in the same format is the current one. Add IP address, port number, name and weight. Default weight is 1, and higher weights are favoured more. Order of entries affacts selection of default and first available peer.

Example:

next_proxy:

- ip_address: xxx.xxx.xxx.xx
  port: xxxx
  name: [proxy1]
  weight: 2
- ip_address: xxx.xxx.xxx.xx
  port: xxxx
  name: [proxy2]
  weight: 1

### Other environments proxy connects to

The above each require a domain(s) and IP address. The always_direct allow rule allows squid to always connect directly to internal servers.

### Port Access

Destination ports that are allowed connection are listed as SSl port and safe ports. Rules are in place to to deny access to all other ports than those explicitly listed in above lists.
User Agent Filters

Rules are in place to stop VSCode extensions user agent.

### Source ACL

Access control lists (ACLs) are written as acl [aclname] [acltype] [argument]. Use this format to allow or deny access to ports, IP addresses or domains.

### No Authentication Lists

Devices that don't need any authentication as they authenticate locally are listed in noauth_lists, with their source IP and name. Be cautious editing this list.

### Unauthentication Allow Lists

These are restricted destinations that don't need to authenticate with IPA. Sites that unauthentication allowed devices are allowed to visit are grouped together under unauth_lists, and can include source IP, destination domain and destination IP.

Example: unauth_lists:

- name: name
  source_ips:
    - ip.address
  destination_domains:
    - domain.name
  destination_ips:
    - ip.address

### Sites Blocked For All

blocked_sites states sites to be blocked for everyone.

### Source To Destination Mapping

http_access allow lists to define which http clients have access to the http port, this is the primary access control list. always_direct controls which requests should always be directed to origin servers.
defaults/main.yml

The below parameters are specific to squid.

### Parameters

These items set initial configuration parameters for the Squid Proxy.

    shutdown_lifetime: set how long to wait until all client requests are complete and Squid can exit.
    http_port: http port to use.
    max_filedescriptors: set the maximum number of file descriptors.
    cache_mem: set ideal amount of memory to use for in transit objects, hot objects and negative cached objects.
    maximum_object_size_in_memory: objects over set size will not be kept in memory cache. This helps optimise performance by preventing too large objects taking up a lot of space, but can also allow frequently accessed objects to be kept in memory.
    memory_cache_mode: controls which objects are kept in cache, set to always to keep most recently fetched objects in memory.
    pipeline_prefetch: set to 1 for on, so squid will prefetch pipeline requests and can processes up to two requests simultaneously.

### Coredump Directory

Location for coredumps, logs of the state of the working memory at a point in time, like when a crash occured. Squid will chdir() to that directory at startup and coredump files will be left there.
Logging

Locations for access logs and cache logs to be stored. cache.log contains debug and error messages from Squid. access.log form the basis for most log file analyses. logformat defines an access log format (logformat squid %ts.%03tu %6tr %>a %Ss/%03>Hs %<st %rm %ru %[credentials %[un %Sh/%<a %mt).

### Refresh Patterns

Determines what is saved to cache and for how long. They are checked in the order listed, the first pattern that matches is used. Options include override-expire, override-lastmod, reload-into-ims, ignore-reload, ignore-no-store, ignore-private, max-stale=NN, refresh-ims or store-stale. Usage: refresh_pattern regex [mode] min [time] percent [max-age value for response] max [time] [options]

### group_vars/all/vault.yml

The Squid bind account password and the IPA admin password are stored in an ansible vault, such as group_vars/all/vault.yml. Please create a vault with a passphrase in this location, otherwise change the path in the role.

## Dependencies

This role uses the redhat.rhel_idm.ipaclient role and has variables set for this role in the tasks/main.yml file.

Example Playbook
------------------
Example playbook of how to use the role, as also seen in tests/test.yml.

- hosts: host_to_become_squid_proxy
  
  become: true
  
  roles:
     - squid

License
------------------
BSD

Author Information
------------------
Elise Elkerton elise@redhat.com


