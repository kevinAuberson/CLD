# Lab 02: App scaling on IaaS

_Duration of this lab: 6 periods_

#### Pedagogical objectives

- Deploy a web application in a scalable way using a two-tier architecture that deploys presentation layer and business logic layer into one tier and the data source layer into another tier
    
- Use a cloud-managed database
    
- Use virtual machine images to clone a web application onto additional virtual machine instances
    
- Use a load balancer that is provided as cloud service
    
- Perform a load test on a load-balanced web application
    

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report. Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

You should have from the previous lab a micro instance running Ubuntu Server 22.04 LTS with Wordpress installed. In the following we will refer to it as the **Wordpress master instance**.

You will improve the Wordpress site to make it scalable. Your site will be able to absorb traffic increases from new users by adding virtual machines that process requests in parallel. Following a two-tier architecture the business logic and presentation layer will be separated from the database layer so that the former can be replicated in multiple virtual machines. The database moves into Amazon’s Relational Database Service (RDS) which provides automatic backup, data replication and failover.

Note 1: Not all deliverables get you the same number of points. Deliverables that only verify that you performed some instructions get fewer points, deliverables that ask questions that test your understanding and require thinking get more points.

Note 2: In the previous lab, you had to set up different userids and passwords / credentials for accessing the database. During this lab, you will need to set up even more of them. Be careful not to mix them up. Make sure to use different usernames and passwords. You need to put these credentials into your report at each task where you set one. At the end of this lab, collect all credentials from lab 1 and lab 2 and summarize them in a table.

### Task 1: Create a database using the Relational Database Service (RDS)

In this task you will create a new RDS database that will replace the MySQL database currently used by Wordpress.

Please read the document [What Is Amazon Relational Database Service (Amazon RDS)?](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) for reference. Once you have read the document, please perform the following steps:

1. In the EC2 console create a _Security Group_ for the database using the lab naming convention `GrF_Nicollier_Wordpress-DB` and open the TCP port on 3306 (MySQL default port).
    
2. Open the RDS console. Make sure the console is set to the same region as the Wordpress master instance.
    
3. Launch a DB instance: Click **Create database** and provide the following answers (leave any field not mentioned at its default value):
    
    - Choose a database creation method
        
        - Select **Standard Create**
    - Engine options
        
        - Engine type: **MySQL**
        - Edition: **MySQL Community**
        - Version: **MySQL 8.0.36**
    - Templates
        
        - Select **Dev/Test**
    - Availability and durability
        
        - Select **Single DB instance**
    - Settings
        
        - DB instance identifier: Use a modified lab naming convention. The identifier cannot contain underscores (`_`), therefore replace them by hyphens (`-`): `GrF-Nicollier-Wordpress-DB`
        - Master username and master password: Create a username / password credential for the master (root) user or the database and write it down.
    - Instance configuration
        
        - DB instance class
            
            - Select **Burstable classes**
            - Select **db.t3.micro**
    - Storage
        
        - Storage type: **General Purpose (SSD)**
            
        - Allocated Storage: **20 GB**
            
        - Storage Autoscaling:
            
            - Enable storage autoscaling: **unchecked**
    - Connectivity
        
        - Virtual Private Cloud (VPC): Select **Default VPC**
            
        - Subnet group: Select **default**
            
        - Public access: **Yes**
            
        - VPC security group: Select the database security group you created earlier (`xxx_Wordpress-DB`).
            
        - Leave the rest as it is
            
    - Database authentication: Select **Password authentication**
        
    - Monitoring
        
        - Enable enhanced monitoring: **unchecked**
    - Additional configuration
        
        - Backup: uncheck **Enable automated backups**
            
        - Encryption: uncheck **Enable encryption**
            
    - Write down the estimated monthly costs that are displayed. **Addendum 2024-03-12:** In certain cases the cost calculator is not displayed. In that case calculate the cost for one month manually with the following prices: in the N. Virginia region a db.t3.micro instance costs $0.017 per hour. Storage type gp2 costs $0.115 per GB-month.
        
    
    After launching the DB instance return to the instances view and wait for the DB instance to be created.
    
4. In the RDS console select the newly created DB instance and write down the **Endpoint** address.
    
5. Test whether the database can be reached from the Wordpress master instance.
    
    - Log into the Wordpress master instance.
        
    - Using the database’s endpoint address (**without** the port number) and the master password you wrote down run the command:
        
        ```
          mysql --host=<endpoint_address> --user=<rds_master_username> --password=<rds_master_password>
        ```
        
        You should see a welcome message and the MySQL command line prompt `mysql>`. Type `quit` to exit.
        
6. **Optional**: On your local machine download and install the **MySQLWorkbench** administration tool from [http://www.mysql.com/products/workbench/](http://www.mysql.com/products/workbench/) and use it to connect to the database.
    

#### Deliverable 1:

- Copy the estimated monthly cost for the database and add it to your report.
    
- Compare the costs of your RDS instance to a continuously running EC2 instance of the same instance type to see how much AWS charges for the extra functionality.
    
- In a two-tier architecture the web application and the database are kept separate and run on different hosts. Imagine that for the second tier instead of using RDS to store the data you would create a virtual machine in EC2 and install and run yourself a database on it. If you were the Head of IT of a medium-size business, how would you argue in favor of using a database as a service instead of running your own database on an EC2 instance? How would you argue against it?
    
- Copy the endpoint address of the database into the report.
    

### Task 2: Configure the Wordpress master instance to use the RDS database

In this task you will edit the Wordpress configuration running on the master instance to use the RDS database instead of the local MySQL database. This will trigger a new Wordpress installation as the RDS database is empty.

As a reminder, the purpose of this part is to extract the data source tier from the presentation and business logic tiers.

#### Change Wordpress’ database configuration

1. Log into the Wordpress master instance.
    
2. Stop the web server by typing:
    
    ```
     sudo systemctl stop apache2
    ```
    
3. To change Wordpress’ configuration parameters to point to the RDS database you will need to change the current configuration. In **/var/www/html/wp-config.php** you will find the current lines :
    
    ```
    // ** Database settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', '<ec2_db_name>' );
    
    /** Database username */
    define( 'DB_USER', '<ec2_db_user>' );
    
    /** Database password */
    define( 'DB_PASSWORD', '<ec2_db_password>' );
    
    /** Database hostname */
    define( 'DB_HOST', 'localhost' );
    ```
    
    Change it to :
    
    ```
    // ** Database settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', '<rds_db_name>' );
    
    /** Database username */
    define( 'DB_USER', '<rds_db_user>' );
    
    /** Database password */
    define( 'DB_PASSWORD', '<rds_db_password>' );
    
    /** Database hostname */
    define( 'DB_HOST', '<endpoint_address_rds_database>' );
    ```
    
4. Create the database `<rds_db_name>` in the RDS database. To do this, connect to the RDS database using the same command as in the verification step after creating the RDS database.
    
    ```
    mysql --host=<endpoint_address> --user=<rds_master_username> --password=<rds_master_password>
    ```
    
    You should see a welcome message and the MySQL command line prompt `mysql>`. Type the following command to create the database:
    
    ```
     CREATE DATABASE <rds_db_name>;
    ```
    
    Type `quit` to exit.
    
5. Start the web server again:
    
    ```
     sudo systemctl start apache2
    ```
    
6. In your browser, go to `http://<public IP address>:80/wp-admin/install.php` and, as you did in the previous lab, install Wordpress.
    
7. Create a quick post containing an image and some text and verify that it is displayed correctly.
    

#### Deliverable 2:

- Copy the part of **/var/www/html/wp-config.php** that configures the database into the report.

### Task 3: Create a custom virtual machine image

Now that you have properly configured the Wordpress master instance, you will save it into a virtual machine image. This image will be used later to create new instances with the exact same configuration.

1. In the EC2 console bring up the **Instances** panel and select the Wordpress master instance.
    
2. Bring up the context menu and select **Image > Create Image**. Provide the following answers (leave any field not mentioned at its default value):
    
    - Image Name: Use the lab naming convention **GrF_Nicollier_Wordpress**.
    - Image Description: **Wordpress connected to RDS database GrF-Nicollier-Wordpress-DB**.
    
    Click **Create Image**. The instance will shut down temporarily and the image will be created.
    
3. In the console bring up the **AMIs** panel. Wait until the status of the AMI goes from **pending** to **available**.
    

#### Deliverable 3:

- Copy a screenshot of the AWS console showing the AMI parameters into the report.

### Task 4: Create a load balancer

In this task you will create a load balancer in AWS that will receive the HTTP requests from clients and forward them to the Wordpress instances.

1. Verifiy and note in which availability zone your instance is running.
    
2. In the EC2 console bring up the **Load Balancers** panel. Click on **Create Load Balancer**. Provide the following answers (leave any field not mentioned at its default value):
    
    - Choose : **Application Load Balancer**
    - Basic configuration
        - Name: Use the lab naming convention with hyphens **GrF-Nicollier-LoadBalancer**.
        - Scheme: **internet-facing**
        - IP address type: **IPv4**
        - Mappings
            - Select 2 availability zones. Make sure that one of them corresponds to where your instance is running.
    - Security Groups
        - Use the same security group as the Wordpress master instance.
    - Listeners and routing
        - Make sure the protocol **HTTP** on port **80** is set, else set it.
        - Target Group : Create a new target group
            - Basic configuration
                - Target type: **Instances**
                - Target group name: Use the lab naming convention with hyphens **GrF-Nicollier-TargetGroup**
            - Health checks
                - Advanced health check settings
                    - Healthy threshold : **2**
                    - Interval : **10** seconds
            - Register targets / Review targets
                - Select your own Wordpress master instance
    - **Create load balancer**
3. In the EC2 console select the newly created load balancer. Write down its **DNS Name** (A Record).
    
4. In the EC2 console select the Target Group. In the lower half of the panel, click on the **Targets** tab. Watch the status of the instance go from **unused** to **initial**.
    
5. Log into the Wordpress master instance. Examine the Apache access log **/var/log/apache2/access.log** to see who is connecting to the web server.
    

#### Deliverable 4:

- On your local machine resolve the DNS name of the load balancer into an IP address using the `nslookup` command (works on Linux, macOS and Windows). Write the DNS name and the resolved IP Address(es) into the report.
    
- In the Apache access log identify the health check accesses from the load balancer and copy some samples into the report.
    

### Task 5: Launch a second instance from the custom image

In this task you will launch a second Wordpress instance and connect it to the load balancer.

1. Using the custom virtual machine image you created earlier launch a second instance.
    
2. Make sure that the instance works correctly by navigating with your browser to the Wordpress post of the new instance at `http://<hostname>/wp-admin`. If some links are broken because of Wordpress storing the IP address in the database, you can either fix them with the same tool as the previous lab or just ignore them for now.
    
3. Using the AWS console connect the instance to the load balancer. Watch the status of the instance go from **Out of Service** to **In Service**.
    
4. Using any of the instances, run the wp search and replace tool to replace the old IP address with the load balancer’s DNS name in the database.
    
5. Make sur that you can access the Wordpress post using the load balancer’s DNS name.
    

#### Deliverable 5:

- Draw a diagram of the setup you have created showing the components (instances, database, load balancer, client) and how they are connected. Include the security groups as well. Make sure to show every time a packet is filtered.
    
- Calculate the monthly cost of this setup. You can ignore traffic costs.
    

### Task 5b: Delete and re-create the load balancer using the command line interface

The cost of the load balancer is around $16 per month, more than a t2.micro instance. To save costs, you will find out in this task how to quickly delete and re-create the load balancer using the command line interface. You can keep the target group with the instances attached to it.

1. Consult the document “The AWS command line interface (CLI)” in the course space in Cyberlearn. To become familiar with the command line, create an EC2 instance, verify that you see it in the management console, then delete it.
    
2. We assume you have the load balancer running. Before deleting it, we have to save some important information so that we can re-create it later. Run the command `describe-load-balancers` to list all load balancers and filter with the JSON tool `jq` for your load balancer name:
    
    ```
     aws elbv2 describe-load-balancers \
       | jq '.LoadBalancers[] | select(.LoadBalancerName | contains("<load-balancer-partial-name>"))'
    ```
    
    It is a good idea to save the output into a file by appending `> my-load-balancer.json`.
    
    We will need later the following information: The load balancer ARN, the IDs of the various subnets and the security group ID.
    
3. Inside the load balancer is the listener. We have to retrieve the information about the listener separately with the `describe-listeners` command. Use the load balancer ARN you retrieved in the previous step.
    
    ```
     aws elbv2 describe-listeners --load-balancer-arn <load-balancer-arn> | jq '.Listeners'
    ```
    
    It is again a good idea to save the output into a file. From the listener information we will need later
    
    - the protocol
    - the port number
    - the default actions with the target group ARN.
4. Delete the load balancer using the `delete-load-balancer` command.
    
5. Re-create the load balancer using the `create-load-balancer` command. Using the previously saved information specify:
    
    - the name
    - the subnets in the different Availability Zones
    - the security group
    - that it is internet-facing: `--scheme internet-facing`
    - that it is an application load balancer: `--type application`
    - that it should use ipv4 addresses: `--ip-address-type ipv4`
    
    Verify on the management console that it has been correctly created.
    
6. Re-create the listener in the load balancer using the `create-listener` command.
    
    Using the previously saved information specify:
    
    - The ARN of the load balancer
    - The protocol HTTP
    - The port 80
    - The default action: forward to the target group (ARN saved earlier)
    
    Verify on the management console that it has been correctly created.
    
7. Reconfigure Wordpress with the new domain name of the load balancer.
    

From this point on we ask you to delete the load balancer (and stop the instances) when you are interrupting your work for a longer time.

#### Deliverable 5b:

- Put the commands to delete the load balancer, re-create the load balancer and re-create the listener into the report.

### Task 6: Test the distributed application

In this task you will test the distributed application with a load generator and use the monitoring tools of the AWS console.

1. Download and install on your local machine the Vegeta load testing tool. You will find in a separate document in this Cyberlearn space a document describing how to install and use Vegeta.
    
2. Open two terminal windows side-by-side and, using SSH, log into each instance. Bring up a continuous display of the Apache access log by running the command **sudo tail -F /var/log/apache2/access.log**.
    
3. Using the AWS console, enable detailed (1-minute interval) monitoring of the two instances: Select an instance and click on the **Monitoring** tab. Click on **Enable Detailed Monitoring**.
    
4. Run a 1-minute load test using default parameters. For the URL of the target of the HTTP requests, use the IP address of the load balancer and the path of the Wordpress app. Observe the output of the load testing tool to verify the requests are properly answered. Create a plot of the latencies and observe how the latencies are distributed.
    
5. Observe which of the instances gets the load. Increase the load and re-run the test. Observe the response times. Do you see errors and/or time-outs (requests that don’t get an answer)? Repeat and increase the load and the duration of the test until you see inacceptable response times, errors and/or time-outs.
    
6. Immediately after having created a high load for the site, re-run the `nslookup` command to resolve the DNS name of the load balancer into IP addresses to see if AWS has scaled out the load balancer itself (it’s normal if you don’t see any changes).
    

Note: Your ability to overload the instances depends on your local network connection. If wifi reception is bad or your network is loaded with other traffic, the load testing traffic may not get enough bandwidth.

#### Deliverable 6:

- Document your observations. Include reports and graphs of the load testing tool and the AWS console monitoring output.
    
- When you resolve the DNS name of the load balancer into IP addresses what do you see? Explain.
    
- Did this test really test the load balancing mechanism? What are the limitations of this simple test? What would be necessary to do realistic testing?
    

Note: In this task it is not important that you reproduce exactly an expected behavior of the load balancer. Your load generator (your local machine and your local network) may not behave always the same, the load balancer may not always behave the same and you may get results different from your colleagues. **What is important however is that you show that you understand the distributed system that you are testing and that you know how to observe the performance of its components with the monitoring functions provided by AWS. You should show that you are able to correlate the performance with different loads generated by the load testing tool and draw conclusions.**

### Task 7: Release Cloud resources

Once you have completed this lab release the cloud resources to avoid unnecessary charges:

- Terminate the EC2 instances.
    - Delete the attached EBS volumes.
- Delete any leftover Elastic IP Addresses from the previous lab.
- Delete the load balancer, but not the target group. You will re-create the load balancer for the next lab.

For the next lab on auto scaling you can reuse these resources:

- AMI of Wordpress master instance
- RDS instance (be sure to stop it)