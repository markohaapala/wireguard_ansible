# wireguard_ansible_alpine

This is the ansible automation of the Wireguard VPN set up for an Alpine image. A part of it is related to Linode's services.

This will create ten VPN client profiles when done.

This project has also been restructured as an ansible role for inclusion in other ansible playbooks.

# Goals for this project

Exposing your home lab to the public through an inexpensive Alpine host having a public IP.

# Requirements

This assumes an Alpine host (server). One step, how to disable automatic networking on your server so /etc/resolv.conf isn't overwritten during boot, is explained for Linode-only.

# Server set up

This assumes you have an Alpine server with ssh access on port 22. Tested with Alpine 3.15.
Ensure that you've already added the server key to your known hosts file by sshing into it at least once.

# Quick setup

Install ansible on your localhost. Clone these files.

```bash
git clone https://github.com/markohaapala/wireguard_ansible_alpine
cd wireguard_ansible_alpine
```

Edit the hosts file in that folder and fill in the IP field with the VPN server IP

Begin the remote installation process by running

```bash
ansible-playbook wireguard.yml -u root -k -i hosts
```

If you're using an SSH key for authentication run this instead

```bash
ansible-playbook wireguard.yml -u root -i hosts --key-file /path/to/keyfile
```

Your server will probably have to be rebooted once during the installation. If your VPS provider doesn't automatically restart your instance, you'll have to restart it manually.

Ten client configs will be created on the VPN server in the folder /root/wg_clients. They will also be downloaded to the **wireguard_role/profiles** folder on your local host.

Assuming you're using the first client config, copy it to **/etc/wireguard/** and you can start using the VPN tunnel on your client.

To bring up the VPN interface
```bash
sudo wg-quick up wg0-client
```

To bring down the VPN interface
```bash
sudo wg-quick down wg0-client
```

To view connection details
```bash
sudo wg show
```

# Advanced use

You have the option of determining the vpn network subnet you prefer your clients to use by editing the file **wireguard_role/defaults/main.yml**, and setting the vpn_network variable as desired. You can also change the vpn server port and the number of client profiles you want generated in the same file:

```bash
vpn_network: '10.200.200'

vpn_port: '51820'

clients: 10
```

## Adding a client

If you want to generate an additional client profile in future, edit the following two variables in **wireguard_role/tasks/main.yml** to your specific needs:

```bash
    new_client: newclient
    new_client_ip: 10.200.200.12
```

Then run the setup process again but now with the tag **add_client** specified:

```bash
ansible-playbook wireguard.yml -u root -k -i hosts -t add_client
```

The new client config will then be downloaded to the **wireguard_role/profiles** folder on your local host.

Note: This needs to be run from the directory the initial setup was done from and not from a newly cloned one.

## Use as an ansible role

This project has been structured as an ansible role. You can therefore include it in other ansible playbooks

```bash
- name: Setup Wireguard VPN
  hosts: all
  gather_facts: true
  roles:
    - {role: 'wireguard_role', tags: 'wireguard'}
```

# DNS

If there is another service listening on port 53, you will have issues with getting DNS resolution working.
It is therefore advisable to either disable or change the port of any service already using port 53.

