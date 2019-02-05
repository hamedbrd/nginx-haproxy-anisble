# Nginx + haproxy
In this ansible playbook we aim to have loadbalancer which is haproxy and some php applications which are running by nginx web server, in this case we choosed lumen framework of php.

## how to run it
For running this ansible you just need to install `vagrant` on your mashine and just run this command.
```
vagrant up
```

 Then by running above command you will bring up 3 nodes as application nodes and 1 node as a loadbalancer
 
## What we have in this ansible
In this ansible we are using two different roles
- php-application
- haproxy


### application

This role is for php application provisioning,  for this job we call some different roles nginx, php, composer and some other configurations which you should have in order to run a php application.

This handles pretty basic stuff for new PHP app nodes:
* nginx configuration
* php configuration
* Directory creation
* Setting up project directory
* composer update

These applications block all incoming request except requests from load balancer which is haproxy in this case.we block all incoming request by `ufw`.

If you want to change the configuration, you have to change the `group_vars/nginx.yml` and also there is another configuration file beside the php-application role located in this path.

``roles/php-application/vars/external_vars.yml``


### haproxy
We use this role `https://github.com/krzyzakp/ansible-haproxy.git` in order to have a load balancer which we put 3 applications nodes behind its backend and it will balance the load by roundrobin and also there is health check to check if a node is healthy then send traffic on that node.



## How to have an organized and mutli stage playbook
The following section shows one of many possible ways to organize playbook content.
One crucial way to organize your playbook content is Ansible’s `roles` organization feature, which is documented as part of the main playbooks page.

I have been writing this document to go through various stages, and used different ways to layout Ansible files as expected from the beginning of this page.


What I mention here would be as a best practice for writing ansible playbook for different stages and different offices.
Here is the example :


```
nginx-haproxy-ansible

└───group_vars
│   │   tehran.yml
│   │   esfahan.yml
│   │   office1.yml
│   │   office2.yml
│   │   nginx.yml
│   │   haproxy.yml
│   │
└───host_vars
│   │   haproxy.yml
└───invertories
│   │   production.yml
│   │   staging.yml
│   │   test.yml
└───roles
│   └───php-applications
│       └───defaults
│           │   main.yml
│       └───tasks
│           │   main.yml
│       └───templates
│           │   hosts.j2
│           │   web.php.j2
│       └───vars
│           │   external_vars.yml
│   README.md
│   requirements.yml   
│   ansible.cfg  
│   Vagrantfile  
│   main.yml  
|
|---------------------------------------
```

For each environment we can have different inventories . Let’s show a static inventory example. Below, the production file contains the inventory of all of your production hosts.

It is suggested that you define groups based on purpose of the host (roles) and also geography or datacenter location (if applicable):

```

# everything in the Tehran geo
[tehran]
web01 ansible_host=172.20.0.21
web02 ansible_host=172.20.0.22


# everything in Esfahan geo
[esfahan]
web03 ansible_host=172.20.0.23
haproxy ansible_host=172.20.0.24


# everything in the office 1
[office1]
web01 ansible_host=172.20.0.21


# everything in the office 2
[office2]
web02 ansible_host=172.20.0.22

# everything in the group nginx
[nginx]
web01 ansible_host=172.20.0.21
web02 ansible_host=172.20.0.22
web03 ansible_host=172.20.0.23


# everything in the group haproxy
[haproxy]
haproxy ansible_host=172.20.0.24


```

Groups are nice for organization, but that’s not all groups are good for. You can also assign variables to them! For instance, Tehran has its own NTP servers, so when setting up ntp.conf, we should use them.

So by above structure in inventory your `main.yml` must be look like this

```
---
- hosts: tehran
  become: true
  roles:
    - { role: php-application }
    - { role: krzyzakp.haproxy}

- hosts: esfahan
  become: true
  roles:
    - { role: php-application }
    - { role: krzyzakp.haproxy}

- hosts: office1
  become: true
  roles:
    - { role: php-application }
    - { role: krzyzakp.haproxy}

- hosts: office2
  become: true
  roles:
     - { role: php-application }
     - { role: krzyzakp.haproxy}

- hosts: nginx
  become: true
  roles:
    - { role: php-application }

- hosts: haproxy
  become: true
  roles:
    - { role: krzyzakp.haproxy}


```

In `main.yml` we could have different groups which each of them could have its own variables in `group_vars` folder, like this 

```
nginx-haproxy-ansible

└───group_vars
│   │   tehran.yml
│   │   esfahan.yml
│   │   office1.yml
│   │   office2.yml
│   │   nginx.yml
│   │   haproxy.yml
│   │
```



## How to test it
When you run vagrant then you have to be able to see the result by typing this ip on your brower
```
172.20.0.24
```
you have to see simple message showing your request is responding on each machine. If something goes wrong with one of them then you are not able to send request on this node anymore, so you no longer see this machine up on loadbalancer.



