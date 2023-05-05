# Ansible playbooks to setup a foundation pot cluster

Ansible playbook to spin up a viable foundation host/cluster for pot images, monitoring and alerting, nomad jobs

It will configure a single server with pot, consul, nomad, traefik-consul, mariadb, openldap, beast-of-argh, and wireguard server with roadwarrior clients. 

Or, it will setup a cluster of 3 or 5 servers, with pot, consul, nomad, traefik-consul, mariadb, openldap, beast-of-argh, all connected via wireguard mesh. 

This is a platform to build on. It doesn't do very much by itself. You need to add your applications.

## Preparation

### Package sources

You must have decided on the `quarterly` or `latest` package stream beforehand. This script proceed to install from the already configured stream. 

### Default network interface must be labelled untrusted

Your default network interface must be called `untrusted`. Make sure it's named correctly and you've rebooted before running the playbook.

You can do this in `/etc/rc.conf` as follows
```
ifconfig_vtnet0_name="untrusted"
ifconfig_untrusted="inet 192.168.1.1 netmask 255.255.255.0"
```

Or it might already be configured that way if FreeBSD was installed with the [https://depenguin.me](depenguin.me installer).

## Installation

### Clone the git repo

Clone the git repo:

```
git clone https://github.com/hnygd/ansible-pot-foundation.git
cd ansible-pot-foundation
```

### Create hosts file from hosts.sample

Copy the `hosts.sample` file to `hosts` and edit to your needs. For example, a 5 server cluster will use the file in `inventory.five`:

```
cd inventory.five
cp hosts.sample hosts
vi hosts
```

### Configure variables in hosts

Replace the following values in your inventory `hosts` file with correct data:

* REPLACE-WITH-USERNAME (username for your server)
* REPLACE-WITH-IP-HOSTNAME (IP or hostname of your server)
* REPLACE-WITH-EMAIL-ADDRESS (your email address)
* REPLACE-WITH-DOMAIN-NAME (your domain name)
* REPLACE-FRONTEND-IP (the IP address of your frontend host)
* REPLACE-GRAFANA-USER (username for grafana, e.g. admin)
* REPLACE-GRAFANA-PASSWORD (password for grafana user)
* REPLACE-WITH-LDAP-PASSWORD (ldap master password)
* REPLACE-DEFAULT-LDAP-USER (default openldap user)
* REPLACE-DEFAULT-LDAP-USER-PASSWORD (openldap user password)
* REPLACE-WITH-MYSQL-ROOT-PASSWORD (mariadb root password)
* REPLACE-WITH-PROM-SCRAPE-PASSWORD (password for prometheus stat scraping)

### Run script

You can install a full system by running:

```
ansible-playbook -i inventory.five/hosts site.yml
```

Make sure to specify the correct inventory. Single-server setups will use `inventory.single` while cluster setups will use `inventory.three` or `inventory.five`.

## Cleanup

### Cleanup script

You can remove the clone images, persistent data, and wireguard setup with:

```
ansible-playbook -i inventory.five/hosts clean.yml
```

# What is configured and installed?

## Networks

### Untrusted Interface

The default network interface must be labelled `untrusted` before running the script. 

Two vlan interfaces are setup on the `untrusted` interface.

#### jailnet

The `jailnet` interface will be used by pot images with IP addresses in this range.

#### compute

The `compute` interface will be used for nomad jobs.

### Wireguard

#### Single Server

A wireguard server will be setup, with two roadwarrior clients.

#### Cluster

A wireguard mesh network will be setup. Pot jails and nomad jobs can communicate with any other host or container in the cluster.

On the primary server, two default roadwarrior clients will be configured.

## PF firewall

Firewalling is handled by PF. A simple pf ruleset is applied. 

## Container Management

### Pot

```
Pot is an open source jail framework/manager for FreeBSD. It's designed to support multiple jail usage, from VM emulation to small container model.
```

Pot has a [https://pot.pizzamig.dev/](website) and [https://github.com/bsdpot/pot](github).

### Potluck

```
Potluck is the central repository for reusable Pot flavours & images 
```

The [https://potluck.honeyguide.net/](potluck website) has a [https://github.com/bsdpot/potluck](github) too.

The images in use in this environment are from the Potluck repository.

## Service Networking and Orchestration

### Consul

```
HashiCorp Consul is a service networking solution that enables teams to manage secure network connectivity between services and across on-prem and multi-cloud environments and runtimes. Consul offers service discovery, service mesh, traffic management, and automated updates to network infrastructure device.
```

A single consul instance, or cluster of 3 or 5 members will be setup. Everything uses consul for service networking.

### Nomad

```
Nomad is a flexible workload orchestrator that enables an organization to easily deploy and manage any containerized or legacy application using a single, unified workflow. Nomad can run a diverse workload of Docker, non-containerized, microservice, and batch applications.
```

For single server instances, one nomad server and one nomad client will be setup.

For clusters, between 3 and 5 servers and clients will be configured.

The two nomad jobs are currently restricted to the primary host.

### Traefik-Consul

```
Traefik Proxy integrates with Consul and uses it as a provider to enable easy north-south traffic configuration. This complements the east-west service mesh that Consul service mesh provides while also sharing some architectural goals: being easy to configure, dynamic, and platform-agnostic.
```

A single traefik-consul host will be setup on the primary server, and is connected to consul and nomad usage.

## Monitoring and Alerting

### Beast of Argh Pot Image

The Beast of Argh pot image is a collection of applications to help with monitoring and alerting.

Specifically it offers:
* Prometheus
* Alertmanager
* Grafana
* Loki
* Promtail syslog-ng log ingestion

The applications are available as individual pot images, however they were developed for a more complicated environment. 

The Beast of Argh pot image is intended for smaller setups seeking simplicity.

## Frontend

### acme.sh

`acme.sh` is used to register a certificate for the public website nomad job.

### Nomad jobs

#### Stadard website

The standard website is a nginx pot image and loads an HTML file with links to internal services, such as consol and nomad dashboards, or monitoring and alerting applications.

#### Nomad job: Public website

The public website is a nginx pot image that serves files from persistent storage. You can add your own website files to this persistent storage.

### Haproxy

```
HAProxy is a free and open source software that provides a high availability load balancer and reverse proxy for TCP and HTTP-based applications that spreads requests across multiple servers.
```

We're using haproxy to send requests from the public IP to the internal nomad job, with SSL certificate management. 

It can be configured to do a lot more.

## Backend

### Openldap

```
OpenLDAP Software is an open source implementation of the Lightweight Directory Access Protocol.
```

A very functional openldap pot image is loaded, complete with web frontend automatically configured. You must access this connected to the wireguard network, or SSH tunnel to server.

### Mariadb

```
MariaDB is a community-developed, commercially supported fork of the MySQL relational database management system, intended to remain free and open-source software under the GNU General Public License.
```

A Mariadb pot image is loaded, but doesn't do anything currently. Databases can be added here. There is no GUI frontend facility yet.


# Room for improvement

## Missing componets

The following improvements could add value:

* Vault server/cluster
* Postgres cluster
* Mail server
* Minio

## Contribute

Pull requests will be evaluated and tested. Please make a contribution!