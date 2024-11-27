# Lab 01: Deploy a web application on Infrastructure as a Service

_Duration of this lab: 6 periods_

#### Pedagogical objectives

- Gain experience with an Infrastructure-as-a-Service offering (Amazon Web Services)
    
- Create a service from scratch using virtual machines
    
- Assess the performance of a virtual machine
    
- Become aware of costs
    

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### Allocating and freeing cloud resources

HEIG-VD is paying for your use of Amazon cloud. Please use the service responsibly:

- Launch virtual machines and other resources only when you are ready to work with them.
- Shut down virtual machines when not in use, for example when you finish working for the day.
- Once you are done with your work free all resources, except when indicated otherwise.

#### Lab convention for naming cloud resources

You will work together with the other students in the same AWS account, and you will see the cloud resources created by other students. To avoid chaos it is therefore important that everybody follows the same convention for naming the resources. This includes public/private key pairs, security groups, EC2 Instances, load balancers, etc.

We will use the following **lab naming convention:** the name of every cloud resource starts with the group, followed by your last name. You can append more information if you want. For example when student Nicollier in Group F creates an EC2 Instance “master”, he names it

```
GrF_Nicollier_master
```

Sometimes you are not allowed to use underscores (`_`) in the name, in that case use dashes (`-`):

```
GrF-Nicollier-master
```

#### A note on regions

By selecting a region an AWS user is able to control where in the world his virtual machines or other cloud resources are located, for example in a particular country or close to the majority of users.

To simplify the administration of your account we impose a limitation in that you can only create resources in one region, _Northern Virginia (us-east-1)_.

You can see the regions when you click on the drop-down menu in the upper right corner. Make sure that you always have **N. Virginia** selected.

If you select another region and create a resource, you will get the error “You are not authorized to perform this operation”.

### Task 1: Set up

In this task you will create a public/private key pair that you will use in task 2 when you login to a newly created virtual machine. You will also create a default firewall configuration for the firewall that protects every virtual machine from attacks from the Internet.

At this point you should already have an AWS account. Sign into your account and access the **AWS Management Console**. You should see a field titled **AWS services** where you can access the 100+ AWS services.

- On the AWS console search for **EC2** and click on **EC2 - Virtual Servers in the Cloud**. You should be able to see the EC2 console.

Follow the instructions given by the guide [Setting Up with Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html) to

- Create a public/private key pair that you will later use to login to a newly created virtual machine. **Name your key** according to the lab naming convention like this:
    
    ```
      GrF_Nicollier
    ```
    
- Create a firewall configuration for the future virtual machine that will allow you to connect to it via SSH. AWS calls firewall configurations _Security Groups_. Allow traffic from anywhere (0.0.0.0/0) to port 22/SSH (this is not secure for a production environment, but OK for the lab). **Name your security group** according to the lab naming convention like this:
    
    ```
      GrF_Nicollier_default
    ```
    
    Give it a description of “default security group for web server”.
    

Skip the following sections of the guide:

- Sign up for AWS
- Create an IAM User
- Create a Virtual Private Cloud

### Task 2: Create an Amazon EC2 instance

In this task you will create a new virtual machine with a preinstalled operating system (Ubuntu) and you will login to the machine with an administrator account. AWS calls virtual machines _EC2 Instances_.

Please read the following guide for reference: [Getting Started with Amazon EC2 Linux Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

Once you have read the guide, please perform the following tasks:

- Launch an instance running **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type**, architecture **x86**, instance type **t2.micro**.
    
    - Leave the instance details at their default settings.
        
    - Leave storage at its default settings.
        
    - In **Step5: Add Tags** click on **name tag** and name the instance, volumes and network interface according to the lab naming convention like this:
        
        ```
          GrF_Nicollier
        ```
        
    - Use the security group you have created earlier (ignore the warning about source 0.0.0.0/0).
        
    - Use the key pair you have created earlier.
        
- Observe how the instance is being created. How long does it take until its state is **running**?
    
- Login to your instance via SSH once it has been launched. AWS has created an account named **ubuntu** for you. To authenticate use the private key you have downloaded before. To get help on the available options for connecting, select the instance and choose **Connect** from the Actions menu.
    
    Tip: By default all ports on your Instance have been locked down. Ensure you have correctly setup the firewall on your instance to allow SSH (Port 22) in case you are having trouble in connecting.
    
- Poke around - Try out the following commands to explore the machine.
    
    - `id` - get info about the account you are logged in, including which groups you are a member of
    - `date` - show the timezone and the current time (notice something?)
    - `lsb_release -a` - get info about the Linux distribution
    - `sudo lshw -short` - display hardware (remember this is all virtual)
    - `lscpu` - display info about the CPU, the hypervisor and the virtualization type
    - `free -h` - display the total amount of memory and how much is used
    - `lsblk` - display disks and other block storage devices (ignore the loop devices)
    - `df -h` - display file systems, their size and how much space is used and available (ignore everything not starting with `/dev/xvd`)
    
    You can always install software packages that you need using the command `sudo apt update` followed by `sudo apt install package_name`. For example to install `tree` type `sudo apt install tree`.
    

**Deliverables:**

- What is the smallest and the biggest instance type (in terms of virtual CPUs and memory) that you can choose from when creating an instance?
    
- How long did it take for the new instance to get into the _running_ state?
    
- From the **EC2 Management Console** copy the **public DNS** name of the instance into the report.
    
- Using the commands to explore the machine listed earlier, respond to the following questions and explain how you came to the answer:
    
    - What’s the difference between time here in Switzerland and the time set on the machine?
    - What’s the name of the hypervisor?
    - How much free space does the disk have?
- Try to ping the instance from your local machine. What do you see? Explain. Change the configuration to make it work. Ping the instance, record 5 round-trip times.
    
- Determine the IP address seen by the operating system in the EC2 instance by running the `ifconfig` command. What type of address is it? Compare it to the address displayed by the ping command earlier. How do you explain that you can successfully communicate with the machine?
    

Note: Once you have completed this part, should you not want to continue immediatley working on the next part, please stop your instance - Do not leave your instance running idle for more than a few hours.

### Task 3: Install a web application

In this part you will install a web-based content management system (CMS), Wordpress. Wordpress can be used to create corporate web sites and among its users are The New York Times Company, Ubisoft Québec, etc. Wordpress is written in PHP and requires a web server in which to run. It makes use of a relational database to store its data. In this case the web server is Apache and the database is MySQL.

**Install LAMP stack**

We will run Wordpress on a classical LAMP stack (Linux, Apache, MySQL, PHP). To install this base software :

```
sudo apt update
sudo apt install lamp-server^
```

It is possible that a dialog box appears asking to restart some services. You can select all the services by checking the box with a `*` using the space bar and then press `Enter` to continue.

Change the configuration of the firewall to allow incoming HTTP connections. In the **EC2 Management Console** navigate to the instance. Scroll to the right until you see the **Security Group** associated with the instance. Click on the security group. Add a rule for HTTP connections coming from anywhere. The security group should now have the following rules:

```
TCP Port (Service) Source
22 (SSH)           0.0.0.0/0
80 (HTTP)          0.0.0.0/0
```

Verify that you can reach the Apache server on port 80 from the browser on your local machine. Enter the public DNS name of the instance into your web browser. You should see the following page:

> **It works!**  
> This is the default welcome page…

Note: Modern browsers choose by default to connect via HTTPS (encrypted, port 443), not HTTP (not encrypted, port 80). To force the browser to connect using HTTP, type the full URL in the address bar

```
http://<public IP address>:80
```

**Download Wordpress**

Note: The following instructions are adapted from this AWS article: [Host a WordPress blog on Amazon Linux 2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hosting-wordpress.html)

We need to download the latest version of Wordpress and extract it.

```
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```

**Set up the database**

We need to set up the MySQL database for Wordpress. First, ensure that the MySQL server is running:

```
sudo systemctl start mysql
```

Then, we create a user and a database for Wordpress.

```
sudo mysql
mysql> CREATE USER 'wordpress-user'@'localhost' IDENTIFIED BY 'wordpress-password';
mysql> CREATE DATABASE `wordpress-db`;
mysql> GRANT ALL PRIVILEGES ON `wordpress-db`.* TO "wordpress-user"@"localhost";
mysql> FLUSH PRIVILEGES;
mysql> exit
```

**Configure Wordpress**

Wordpress uses a configuration file to set a bunch of parameters, such as the database connection. We need to copy the sample configuration file and edit it (we use the `nano` editor).

```
cp wordpress/wp-config-sample.php wordpress/wp-config.php
nano wordpress/wp-config.php
```

In the editor, find the lines below and replace them with as follows:

```
define('DB_NAME', 'wordpress-db');
define('DB_USER', 'wordpress-user');
define('DB_PASSWORD', 'wordpress-password');
```

Then, copy the files to the Apache directory for site content and set the correct permissions.

```
sudo cp -r wordpress/* /var/www/html/
sudo chown -R www-data:www-data /var/www
```

Finally, enable Wordpress permalinks.

```
sudo nano /etc/apache2/apache2.conf
```

You need to find the block `<Directory /var/www/>` and change `AllowOverride None` to `AllowOverride All`.

Then, restart the Apache server.

```
sudo systemctl restart apache2
```

**Install Wordpress**

Now, in your web browser, navigate to `http://<public IP address>:80/wp-admin/setup-config.php`. It will ask you to fill some information about the website. Feel free to choose whatever you want, as long as you remember the username and password. You can check the box to discourage search engines from indexing the site, as we don’t want to be indexed by Google or other search engines (which is pretty unlikely to happen anyway).

When installation is finished you should see a functional Wordpress site.

- Start to use the newly created Wordpress site by creating a new page.
    
- Save your instance’s IP somewhere and then, in the AWS console create an Elastic IP Address and attach it to the instance.
    
    - Click on **Elastic IPs** in the left navigation bar then click **Allocate New Address**.
        
    - To be able to find it later tag it with the key `Name` and as value put the lab naming convention like this:
        
        ```
          GrF_Nicollier
        ```
        
    - Select the newly created address and click on **Actions > Associate**. Select the Wordpress instance.
        
    - Unfortunatly, you will have to do a small modification in Wordpress because it is not aware of the new IP address. There is a tool called `wp-cli` that can help you with that. Install it and replace the IP with the following commands:
        
        ```
          curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
          php wp-cli.phar search-replace '<OLD_IP>' '<NEW_IP>' --path=/var/www/html/ --skip-columns=guid
        ```
        
- Ping the Elastic IP Address from your local machine. You may have to wait a few seconds until the address is associated. Using the browser navigate to the Wordpress start page by using the Elastic IP Address.
    

**Deliverables:**

- Add a screenshot of the page you created in Wordpress to the report.
    
- Add the Elastic IP Address you created to the report.
    
- Why is it a good idea to create an Elastic IP Address for a web site (our web application)? Why is it not sufficient to hand out as URL for the web site the public DNS name of the instance?
    

Note: Once you have completed this part, should you not want to continue immediatley working on the next part, please stop your instance - Do not leave your instance running idle for more than a few hours. Also detach and release the Elastic IP Address.

### Task 4: Performance analysis of your instance

In this part you will run a benchmarking application on your provisioned EC2 instance and your local machine to test compute performance and memory throughput and compare the performance of the instance with your local machine.

On your EC2 instance perform the following steps.

1. Download the Linux version of the Geekbench benchmark:
    
    ```
     curl -O http://cdn.primatelabs.com/Geekbench-3.3.0-Linux.tar.gz
    ```
    
2. Extract the files from the archive:
    
    ```
     tar -xvzf Geekbench-3.3.0-Linux.tar.gz
    ```
    
3. Install 32-bit compatibility libraries needed by Geekbench:
    
    ```
     sudo dpkg --add-architecture i386
     sudo apt update
     sudo apt install libc6:i386 libstdc++6:i386
    ```
    
4. Run the benchmarks:
    
    ```
     cd dist/Geekbench-3.3.0-Linux
     ./geekbench
    ```
    
    You should see output similar to this:
    
    ```
     Geekbench 3.3.0 Tryout : http://www.primatelabs.com/geekbench/
    
     Geekbench 3 is in tryout mode.
     [...]
     Running Gathering system information
     [...]
     Uploading results to the Geekbench Browser. This could take a minute or two 
     depending on the speed of your internet connection.
     [...]
    ```
    

**Deliverables:**

- Provide the URLs of the Geekbench results for the EC2 instance and your local machine.
    
- Provide system information about the EC2 instance.
    
- Provide the single-core and multi-core performance scores for overall, integer, floating-point and memory performance of the EC2 instance.
    
- Provide system information about your local machine.
    
- Provide the single-core and multi-core performance scores for overall, integer, floating-point and memory performance of your local machine.
    
- Compare the overall scores of the two machines.
    

Note: Once you have completed this part, should you not want to continue immediatley working on the next part, please stop your instance - Do not leave your instance running idle for more than a few hours.

### Task 5: Resource consumption and pricing

In this part you will determine how much Amazon charges customers for using cloud resources.

**Deliverables:**

- How much does your instance cost per hour? How much does your disk cost per hour? What was the cost for this lab (make an approximate guess)?
    - You will find hourly/monthly prices at the links below. Look up the prices for the region in which you created the instance.
    - Do not take into account the AWS free tier, only regular prices.
    - Show the details of your calculation.
    - [Prices for EC2 Instances](https://aws.amazon.com/ec2/pricing/on-demand/#On-Demand_Pricing)
    - [Prices for EBS Volumes](https://aws.amazon.com/ebs/pricing/): Choose the cheapest SSD
    - The EBS Volume you created uses an SSD drive to store its data. When you buy an SSD drive at Digitec ([https://www.digitec.ch/en/producttype/ssd-545](https://www.digitec.ch/en/producttype/ssd-545)) how much do you pay per TB? (Change the sort order to_top-selling_ and look at the best selling model. Prices per TB are shown in gray.)  
        
- Calculate the total cost of the configuration used in the lab if everything was running continuously in production during a whole month. Take additionally into consideration the resources below.
    - Show the details of your calculation.
    - [Prices for data transfer](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer): Consider the data transfer from the EC2 Instance to the users. Assume 100’000 visitors downloading 85 MB each.
    - [Prices for public IPv4 addresses](https://aws.amazon.com/vpc/pricing/)

Note: Please stop your instance once you have completed this part - Do not leave your instance running idle for more than a few hours.

### Task 6: Cleanup

Stop your instance. Don’t terminate it, as you will need it for the next lab.

Release the Elastic IP Address.