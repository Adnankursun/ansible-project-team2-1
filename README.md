PROVISIONING AN AUTOSCALING INFRASTRUCTURE WITH WORDPRESS INSTALED USING ANSIBLE

We use Ansible to manage WordPress deployments to EC2 with Auto Scaling. One crucial feature is that it is able to hand-hold a rolling deploy (that is, 1 min downtime) by terminating and replacing instances.
In this project we'll walk through each stage of the build and deployment process, and use Ansible to perform all the work. The goal is to build our entire environment from scratch, save for a few manually created resources at the outset.

Prerequisites:

1. Terraform 0.11.14 and >
2. Ansible 2.9
3. Python >=2.6
4. boto3
5. boto
6. docs.ansible.com 

Setting up Terraform

To prepare the environment we need to install necessary packages  
1) yum install unzip wget git -y  

2) wget https://releases.hashicorp.com/terraform/0.11.14/terraform_0.11.14_linux_amd64.zip

3) unzip terraform_0.11.14_linux_amd64.zip 

4) mv terraform /bin  

source setenv.sh configurations/awx.tfvars

terraform apply -var-file configurations/awx.tfvars 

Setting up Ansible

Ansible uses Boto for AWS interactions, so you'll need that installed on your control host. Your platform may differ, but the following will work for most platforms:

sudo amazon-linux-extras install ansible2 (amazon linux)

sudo yum install python-boto3 -y

sudo yum install python-pip -y

sudo pip install boto

sudo pip install boto3

Sometimes we have errors because of boto version so we need to run following command in order to upgrade it:

pip install boto3 --upgrade

Step 1: Launch a new EC2 instance and Create AMI

A prerequisite to setting up an application for auto scaling involves building an AMI containing your WordPress application, which will be used to launch new instances to meet demand. We'll start by launching a new instance onto which we can deploy our application by using Playbook.  We have used CentOS and Amazon Linux AMI each separately in order to install WordPress. After Instance and WordPress running, we can copy AMI to localhost or different region.

Step 2: Create Security Group

Next Step you need to create Security Group for EC2 and ELB in order that they can communicate with each other and taking care to open the necessary ports for your application in addition to TCP/22 for SSH, we need to add HTTP ( use module ec2_group).

Step 3: Create a Launch Configuration

Our AMI is built, so now we'll want to create a new Launch Configuration to describe the new instances that should be launched from this AMI. We need to choose size, type and other requirements for instances to be used by ASG, use module - ec_2_lc.

Step 4: Create a Target Group

Since each target group is used to route requests to one or more registered targets. When you create each listener rule, you specify a target group and conditions. When a rule condition is met, traffic is forwarded to the corresponding target group. Next our step we need to create Target group for Applicational Load Balancer, use module -elb_application_lb

Step 5: Create an Application Load Balancer

We will connect to an Application Load Balancer which will distribute incoming requests among the instances we have launched into our upcoming Auto Scaling Group. Again we'll create another role to handle the management of the ELB, and apply it from our playbook.

Step 6: Create and configure an Auto Scaling Group

We'll create an Auto Scaling Group to make architecture Highly Available and in order that our WordPress application will not down even one of our instances will be stopped or terminated (use module-ec2_as). We need to configure it to use the Launch Configuration we previously created. Within the boundaries that we define, AWS will launch instances into the ASG dynamically based on the current load across all instances. Equally when the load drops, some instances will be terminated accordingly. Exactly how many instances are launched or terminated is defined in one or more scaling policies, which are also created and linked to the ASG. The minimum and maximum sizes for the auto scaling group are set to 3 and 48 respectively. It's important to get these values right for your application workload. You do not want to be under resourced for early peaks in traffic, and for redundancy reasons it's a good idea to always have at least 3 instances in service. Equally you probably want your application to scale for peak periods, but perhaps not beyond a safety limit in the event you receive massive amounts of traffic which could result in escalating costs.
