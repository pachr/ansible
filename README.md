# ansible

This folder contains playbooks for :
- AWS EC2 provisioning (Security group and launch instances)
- Install MySQL server
- Install Memcached

There are always three scripts (dev / stage / prod) for each roles

##AWS EC2 provisioning

###Usage :


- Run ec2_provisioning role (Create a security group)

```shell
ansible-playbook -i inventory_name aws_ec2_[dev/stage/prod].yml --extra-vars="hosts=local connection=local keypair=keypair_name" --private-key=/path/to/keypair.pem --tags "ec2_provisionning"

```

- Run ec2_launch_instances 

```shell
ansible-playbook -i inventory_name aws_ec2_[dev/stage/prod].yml --extra-vars="hosts=local connection=local keypair=keypair_name" --private-key=/path/to/keypair.pem --tags "ec2_launch_instances"

```

*You can run both roles, without --tags options in the cmd line*

###Extra vars :

For ec2_provisioning (Create a security group)

- hosts : hosts groupname
 * Type : string
 * No default, it must be specifiy in cmd => local or 127.0.0.1 should be a good choice 

- connection : 
 * Type : string
 * No default, it must be specifiy in cmd => local or 127.0.0.1 should be a good choice 

- security_group : Security group name (will be created with this name)
*It's also the groupname in my inventories*
 * Type : string
 * Default : my_security_group_[dev/prod/stage]

- region : Region name on AWS
 * Type : string
 * Default : eu-west-1

- server_env : Environment name (development, production, staging...)
 * Type : string
 * Default : dev, prod or consistent

CMD :

```shell
ansible-playbook -i inventory_name aws_ec2_[dev/stage/prod].yml --extra-vars="hosts=local connection=local security_group=security_groupname region=region_name server_env=environment_name" --private-key=/path/to/keypair.pem --tags "ec2_provisionning"

```
For ec2_launch_instances 

- hosts : hosts groupname
 * Type : string
 * No default, it must be specifiy in cmd => local or 127.0.0.1 should be a good choice 

- connection : 
 * Type : string
 * No default, it must be specifiy in cmd => local or 127.0.0.1 should be a good choice 

- security_group : Security group name (will be created with this name)
 * Type : string
 * Default : my_security_group_[dev/prod/stage]

- region : Region name on AWS
 * Type : string
 * Default : eu-west-1

- server_env : Environment name (development, production, staging...)
 * Type : string
 * Default : dev, prod or consistent

- keypair: keypair name (not the path and without .pem)
 * Type : String
 * Must be specify in extra vars

- group: 
 * Type : String
 * Default : my-security-group-[dev/stage/prod]
    
- instance_type: 
 * Type : String
 * Default : t2.micro (Free on AWS)
    
- image: (It could be a custom image)
 * Type : String
 * Default : ami-234ecc54 (Free on AWS)
    
- count: Number of instances to launch
 * Type : Integer
 * Default : 1

- tag_name
 * Type : String
 * Default : Dev / Consistent / Prod


# ansible-memcached

The playbook install memcached with a custom /etc/memcached.conf

## Vars :

- hosts : hosts groupname
 * Type : string
 * No default, it must be specifiy in cmd

- remote_user : the remote user on AWS
 * Type : string
 * No default, it must be specify in cmd 

- listen : the listening ip address
 * Type : string
 * Default : 127.0.0.1

- user : the user who launch the memcached deamon
 * Type : string
 * Default : root

- memory : the maximum memory used (Mb)
 * Type : integer
 * Default : 64

- max_connections : the maximum simultaneous connections
 * Type : integer
 * Default : 1024

## group_vars :

Variables which can be configure (group_vars/all) => for the when condition (default : true)

## Usage :

###With default vars :
```
ansible-playbook -i hosts  memcached.yml --extra-vars "hosts=launched remote_user=ubuntu" --private-key=/path/to/keypair 

```

###With extra_vars :
```
ansible-playbook -i hosts  memcached.yml --extra-vars "hosts=launched remote_user=ubuntu listen=127.0.0.1 user=root memory=66 max_connections=1026" --private-key=/path/to/keypair

```

#MySQL

##Vars

- hosts : hosts groupname
 * Type : string
 * No default, it must be specifiy in cmd

- remote_user : the remote user on AWS
 * Type : string
 * No default, it must be specify in cmd 

- update_apt_cache :
 * Type : String 
 * Default : yes

- mysql_root_password : Password for the MySQL root user
 * String
 * Must be specify in extra vars

Others vars :

kernel_shmmax: 5368709120  
shared_buffers: 3840MB  
effective_cache_size: 7680MB  
work_mem: 32MB  
maintenance_work_mem: 1024MB  
checkpoint_segments: 32  
checkpoint_completion_target: 0.9  
innodb_buffer_pool_size: 10G  
innodb_buffer_pool_instances: 5  

##Usage

CMD

```shell
 ansible-playbook -i hosts_[dev/stage/consistent] mysql_[dev/stage/prod].yml --extra-vars="hosts=hosts_groupname remote_user=remote_username mysql_root_password=root_user_password db_name=db_name_to_create db_user=db_username db_password=custom_pass" --private-key=/path/to/keypair.pem

```

##Bug fixed

There was a bug with innodb_buffer, MAX 8G for a t2.micro instance_type on AWS EC2