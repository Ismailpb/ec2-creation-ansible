# Ec2-creation-ansible

This is a simple project to create an ec2 instance with sample website.

For working on Ansible we need to first set up a few things,

    AWS user account
    Ansible
    Python
    Boto

For creating the AWS account just go to the Amazon AWS server and follow the signup process.
Once the AWS account gets created you need to create the IAM user with programmatic access (As we will need a secret key and secret ID).

Open the AWS Console, search for IAM (Identity and Access Management) and follow these steps to create a user and take note of the Access Key and Secret Key that will be used by Ansible to set up the instances. (For account access just give Programmatic access as of now.)

Once you are done with the AWS account and the User creation, you can move forward and install the required things.

    Ansible:
        Install Ansible on a amazon Linux based system
        ```
        $ amazon-linux-extras install ansible2 -y
        
        ```
 Once the ansible is installed check the python version enabled for the ansible by using
        
```
        ansible --version
        
        ansible 2.9.23
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.18 (default, Jun 10 2021, 00:11:02) [GCC 7.3.1 20180712 (Red Hat 7.3.1-13)]
```
 Here in my case the python version is 2.7.18, so I need to install python module called "pip" in my amaozn linux machine.
 ```
 # yum install python-pip
 ```
 Once the pip is installed we need to install python package "boto" for creating ec2-instance. The command used to install is 
     
 ```
 pip2 install boto boto3 botocore
 ```

Now, we are done with the package installation, we can move ahead and start writing our Ansible playbook.

## Installation.

In order to create an Ec2-instance we need some informations mentioned below.
- Key pair
- Security Group
- Userdata

Lets start with creating some variables in the yml file.

```
  vars:
    keypair_name: "ansible"      
    access_key: "XXXXXXXXXXX"
    secret_key: "YYYYYYYYYYYYYYYYYYYYYYYYYYY"
    region: "ap-south-1"
    sg1: "ansible-remote"
    sg2: "ansible-webserver"
```
 So at first we need to create a key pair using ansible for accessing the ec2- instance. You can refer the below codes on how to create a key pair
## 1.Keypair Creation

```

    - name: "Creating key pair"
      ec2_key:
        aws_access_key: "{{ access_key }}"
        aws_secret_key: "{{ secret_key }}"
        region: "{{ region }}"
        name: "{{ keypair_name }}"
        state: present
      register: keypair_content
```
This will create a keypair named "ansible.pem in local to connect the ec2.

## 2. Security Group

For this project I have created two security group named ansible-remote and ansible-webserver.

```
- name: "Creating Security Group {{ sg1 }}"
      ec2_group:
        aws_access_key: "{{ access_key }}"
        aws_secret_key: "{{ secret_key }}"
        region: "{{ region }}"
        name: "{{ sg1 }}"
        description: "Allows only 22 connection"
        rules:
          - proto: tcp
            from_port: 22
            to_port: 22
            cidr_ip:
              - 0.0.0.0/0
            cidr_ipv6:
              - ::/0

        tags:
          name: "{{ sg1 }}"
      register: sg1_status


    - name: "Creation of Security Group {{ sg2 }}"
      ec2_group:
        aws_access_key: "{{ access_key }}"
        aws_secret_key: "{{ secret_key }}"
        region: "{{ region }}"
        name: "{{ sg2 }}"
        description: "Allows 80 443 connection"
        rules:
          - proto: tcp
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
            cidr_ipv6: ::/0

          - proto: tcp
            from_port: 443
            to_port: 443
            cidr_ip: 0.0.0.0/0
            cidr_ipv6: ::/0

        tags:
          name: "{{ sg2 }}"
      register: sg2_status
```

Once this completed we can proceed with the ec2 creation.

```
    - name: "Creation of ec2-Instance"
      ec2:
        aws_access_key: "{{ access_key }}"
        aws_secret_key: "{{ secret_key }}"
        region: "{{ region }}"
        key_name: "{{ keypair_name }}"
        instance_type: "t2.micro"
        image: "ami-052cef05d01020f1d"
        user_data: "{{ lookup('file', 'user-data.sh') }}"
        group_id:
          - "{{ sg1_status.group_id }}"
          - "{{ sg2_status.group_id }}"
        instance_tags:
          Name: "ansible-test"
        count_tag:
          Name: "ansible-test"
        exact_count: 1
```
In this section, we can define the "user_data" field so that we can provide the post installation like (httpd,php..etc) without accessing the ec2.





## Results

Sample website is loading on the ec2

![image](https://github.com/Ismailpb/ec2-creation-ansible/blob/1c4358ebb69164c533d3aad011aa1673bfc41d92/Screenshot%20from%202022-01-08%2003-28-04.png)
