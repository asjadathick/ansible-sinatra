# Sinatra as Code

Ansible playbooks to deploy a ruby Sinatra application on a newly provisioned AWS EC2 instance.

The instructions are for a local macOS environment. Substitute brew with your package manager of choice if on a different platform.

### Requirements
  - awscli 
```brew install awscli```
  - AWS Credentials with EC2 Admin privileges (Access/Secret key)
Run:
```aws configure``` and enter in the keys/default region (ap-southeast-2) for your aws account.
  - ansible
```brew install ansible```
  - Create and download a keypair with the name 'ansible' on AWS EC2. Save the ansible.pem file in your ~/.ssh/ directory. Instructions on creating a keypair [here](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html). Alternatively, use aws-cli and ```create-key-pair```.
  - Update your vpc_subnet_id (line 14) in [provision.yml](/provision.yml). You can find your subnet_id in your EC2 VPC dashboard. This ID is required to create the EC2 instance.

### Playbook structure
The solution is organised into 2 playbooks:
- Provision.yml
This step provisions the appropriate security group and EC2 instance on AWS and adds the provisioned host into a local host file

- Configure.yml
This step configures the instance provisioned in the previous step by installing bundler, cloning the sinatra git repo and running bundle/rake exec on the sample Sinatra application. The web application is accessible on the public hostname at the end of this step

- Terminate_all.yml
This script terminates all EC2 instances on your AWS account with the tag Name: Webserver. This is used to cleanup any hosts provisioned using the Provision.yml playbook.

### Steps for deployment

Open up a shell and use the following commands to deploy the Sinatra app.

```sh
$ ansible-playbook provision.yml
$ ansible-playbook configure.yml
```

> Navigate to the public IP shown when the configure playbook runs to see the Sinatra app in action.

### Future improvements

Some improvements that come to mind include:
- Removing SSH access on the Security Group after configuring the instance to reduce risk posture
- Disabling publickey login after configuration is complete to reduce risk posture
- Deploy the application using a pre-built AMI image, rather than configuring the instance after provisioning a fresh EC2 (Use packer to build image and upload AMI to AWS prior to provisioning)
- Run best practice Linux hardening [steps](https://www.linuxjournal.com/content/security-hardening-ansible) to improve host security

### Issues
- terminate_all.yml does not remove old hosts from the host file. This will leave configure.yml trying to configure dead hosts when invoked in the future

