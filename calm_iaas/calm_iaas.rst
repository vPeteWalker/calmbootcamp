.. _calm_linux:

---------------------
Calm: Linux Workloads
---------------------

*The estimated time to complete this lab is ?? minutes.*

Overview
++++++++

Nutanix Calm allows you to seamlessly select, provision, and manage your business applications across your infrastructure for both the private and public clouds. Nutanix Calm provides App lifecycle, monitoring and remediation to manage your heterogeneous infrastructure, for example, VMs or bare-metal servers. Nutanix Calm supports multiple platforms so that you can use the single self-service and automation interface to manage all your infrastructure.

Infrastructure-as-a-Service (IaaS) is defined as the ability to quickly provide compute resources, on-demand through a self service portal.  While many customers utilize Nutanix Calm to orchestrate complex, multi-tiered applications, a significant portion of customers also utilize Calm to provide basic IaaS for their end users.

**In this lab you will create a "Single VM Blueprint" based on either a Linux or Microsoft OS, ??and publish the blueprint into the marketplace to provide IaaS for end users.??**

Creating a Single VM Blueprint
++++++++++++++++++++++++++++++

A blueprint is the framework for every application or piece of infrastructure that you model by using Nutanix Calm.  While complex, multi-tiered applications utilize the "Multi VM/Pod Blueprint", the streamlined interface of the "Single VM Blueprint" is conducive for IaaS use cases.  You can model each type of infrastructure your company utilizes (for instance Windows, CentOS, and/or Ubuntu) in a Single VM blueprint, and end users can repeatedly launch the blueprint to create infrastructure on demand.  The resulting infrastructure (which is still referred to as an "application" within Calm), can then be managed throughout its entire lifecycle within Calm, including managing Nutanix Guest Tools (NGT), modifying resources, snapshotting, and cloning.

In this lab, you have the option of either creating a CentOS 7 Linux server, or Windows 2016 server.  It is recommended to start with the OS that you're most familiar with.  If desired, repeat the lab with the other OS after you've completed it first.

#. In **Prism Central**, select :fa:`bars` **> Services > Calm**.

   .. figure:: images/1_access_calm.png

#. Select |blueprints| **Blueprints** in the left hand toolbar to view and manage Calm bleuprints.

   .. note::

     Mousing over an icon will display its title.

#. Click **+ Create Blueprint > Single VM Blueprint**.

#. Fill out the following fields:

   - **Name** - *Initials*-CentOS-IaaS or *Initials*-Windows-IaaS, depending on your OS choice
   - **Description** - Something descriptive of your choice
   - **Project** - *Initials*-Calm

   .. figure:: images/2_centos_1.png
       :width: 400px
       :align: center
       :alt: CentOS 7 Blueprint Settings

       CentOS 7 Blueprint Settings

   .. figure:: images/3_windows_1.png
       :width: 400px
       :align: center
       :alt: Windows 2016 Blueprint Settings

       Windows 2016 Blueprint Settings

#. Click **VM Details** to proceed to the next step.

#. Note the following fields on the **VM Details** page:

   - **Name** - The internal-to-Calm name of the VM.  Can be left as default.
   - **Cloud** - The cloud we're deploying the infrastructure on.  Should be left as **Nutanix**.
   - **Operating System** - The type of OS we're deploying.  It should be left as Linux if you're using CentOS 7, or changed to Windows if using Windows 2016. 

   .. figure:: images/4_centos_2.png
       :width: 400px
       :align: center
       :alt: CentOS 7 VM Details

       CentOS 7 VM Details

   .. figure:: images/5_windows_2.png
       :width: 400px
       :align: center
       :alt: Windows 2016 VM Details

       Windows 2016 VM Details

#. Click **VM Configuration** to proceed to the next step.

#. On this page, we'll be specifying a variety of settings for our infrastructure.

   - **General Configuration**

     - **VM Name** - this is the name of the virtual machine according to the hypervisor/cloud.  It can be left as default.
     - **vCPUs** - the number of vCPUs to assign to the VM.  For **CentOS**, enter **2**.  For **Windows**, enter **4**.  For both, mark the field as **runtime** by clicking the running man icon so it turns blue.  This allows the end user to modify this field at launch.
     - **Cores per vCPU** - the number of cores per vCPU to assign to the VM.  For both OSes, enter **1**.
     - **Memory (GiB)** - the amount of RAM to assign to the VM.  For **CentOS**, enter **4**.  For **Windows**, enter **6**.  Mark this field as **runtime**.

     .. figure:: images/6_centos_3.png
         :width: 400px
         :align: center
         :alt: CentOS 7 VM Configuration - General Configuration

         CentOS 7 VM Configuration - General Configuration

     .. figure:: images/7_windows_3.png
         :width: 400px
         :align: center
         :alt: Windows 2016 VM Configuration - General Configuration

         Windows 2016 VM Configuration - General Configuration


   - **Guest Customization** - Guest customization allows for the modification of certain settings at boot.  Linux OSes use "Cloud Init", while Windows OSes use "Sysprep".  Select the **Guest Customization**, and then paste in one of the two following scripts, depending on your OS.

     - CentOS 7

       .. literalinclude:: cloud-init.sh
          :language: bash

       .. figure:: images/8_centos_4.png
           :width: 400px
           :align: center
           :alt: CentOS 7 Cloud Init

           CentOS 7 Cloud Init

     - Windows 2016

       .. literalinclude:: sysprep.xml
          :language: xml

       .. figure:: images/9_windows_4.png
           :width: 400px
           :align: center
           :alt: Windows 2016 Sysprep

           Windows 2016 Sysprep

     .. note::
        Take note of the "@@{vm_password}@@" text.  In Calm the "@@{" and "}@@" characters represent a macro.  At runtime, Calm will automatically "patch" or substitute in the proper value(s) when it encounters a macro.  A macro could represent a system defined value, a VM property, or (as it does in this case) a runtime variable.  Later in this lab we'll create a runtime variable with the name "vm_password".

   - **Disks** - A disk is the storage of the VM / Infrastructure that we're deploying.  It could be based on a pre-existing image (as it will in our case), and/or it could be based on a blank disk to enable the VM to consume additional storage.  For instance, a Microsoft SQL server may need its base OS disk, a separate SQL Server binary disk, separate database data file disks, separate TempDB disks, and a separate logging disk.  In our case we're going to have a single disk, based on a pre-existing image.

     - **Type** - The type of disk, this can be left as default (**DISK**).
     - **Bus Type** - The bus type of the disk, this can be left as default (**SCSI**).
     - **Operation** - How the disk will be sourced.  "Allocate on Storage Container" is used for blank disks.  We're going to keep the default, **Clone from Image Service**, as we're using a pre-defined image.
     - **Image** - The image the VM will be based off of.  Select either **CentOS7.qcow2** or **Windows2016.qcow2**, depending on your OS choice.
     - **Bootable** - Whether or not this particular disk is bootable.  A minimum of one disk *must* be bootable.  In our case, leave it **enabled**.

     .. figure:: images/10_centos_5.png
         :width: 400px
         :align: center
         :alt: CentOS 7 VM Configuration - Disks

         CentOS 7 VM Configuration - Disks

     .. figure:: images/10_windows_5.png
         :width: 400px
         :align: center
         :alt: Windows 2016 VM Configuration - Disks

         Windows 2016 VM Configuration - Disks

   - **Boot Configuration** - The boot method of the VM.  We'll leave the default of **Legacy BIOS**.

   - **vGPUs** - Whether or not the VM needs a virtual graphical processing unit.  We'll leave the default of none.

   - **Categories** - Categories span several different products and solutions within the Nutanix portfolio.  They enable you to set security policies, protection policies, alert policies, and playbooks.  Simply choose the categories corresponding to the workload, and all of these policies will automatically be applied.



Defining Variables
++++++++++++++++++

Variables allow extensibility of Blueprints, meaning a single Blueprint can be used for multiple purposes and environments depending on the configuration of its variables.
Variables can either be static values saved as part of the Blueprint or they can be specified at **Runtime** (when the Blueprint is launched).  Variables are specific to a given **Application Profile**, which is the platform on which the blueprint will be deployed. For example, a blueprint capable of being deployed to both AHV and AWS would have 2 Application Profiles. Each profile could have individual variables and VM configurations.

By default, variables are stored as a **String** and are visible in the Configuration Pane. Setting a variable as **Secret** will mask the value and is ideal for variables such as passwords. In addition to the String and Secret options, there are Integer, Multi-line String, Date, Time, and Date Time **Data Types**, and more advanced **Input Types**, however these are outside the scope of this lab.

Variables can be used in scripts executed against objects using the **@@{variable_name}@@** construct. Calm will expand and replace the variable with the appropriate value before sending to the VM.

#. In the **Configuration Pane** on the right side of the Blueprint Editor, under **Variables**, add the following variables (**Runtime** is specified by toggling the **Running Man** icon to Blue):

   +------------------------+-------------------------------+------------+-------------+
   | **Variable Name**      | **Data Type** | **Value**     | **Secret** | **Runtime** |
   +------------------------+-------------------------------+------------+-------------+
   | User_initials          | String        | xyz           |            |      X      |
   +------------------------+-------------------------------+------------+-------------+
   | Mysql\_user            | String        | root          |            |             |
   +------------------------+-------------------------------+------------+-------------+
   | Mysql\_password        | String        | nutanix/4u    |     X      |             |
   +------------------------+-------------------------------+------------+-------------+
   | Database\_name         | String        | homestead     |            |             |
   +------------------------+-------------------------------+------------+-------------+

   .. figure:: images/5.png

#. Click **Save**.

Adding a Downloadable Image
+++++++++++++++++++++++++++

VMs in AHV can be deployed based on a disk image. With Calm, you can select a Downloadable Image via a URI. During the application deployment, Prism Central will automatically download and create the image specified. If an image with the same URI already exists on the cluster, it will skip the download and use the local image instead.

#. From the top toolbar, click **Configuration > Downloadable Image Configuration** :fa:`plus-circle` and fill out the following fields:

   - **Package Name** - CentOS_7_Cloud
   - **Description** - CentOS 7 Cloud Image
   - **Image Name** - CentOS_7_Cloud
   - **Image Type** - Disk Image
   - **Architecture** - X86_64
   - **Source URI** - http://download.nutanix.com/calm/CentOS-7-x86_64-GenericCloud.qcow2
   - **Product Name** - CentOS
   - **Product Version** - 7

   .. note::
      This Generic Cloud image is the same that's used for the majority of the Nutanix Pre-Seeded Application Blueprints.

   .. figure:: images/6.png

#. Click **Save**, and then **Back**.

Creating Services
+++++++++++++++++

Services are the virtual machine instances, existing machines or bare-metal machines, that you can provision and configure by using Nutanix Calm.

In this exercise you will create the database, webserver, and load balancer services that comprise your application.

Creating the Database Service
.............................

#. In **Application Overview > Services**, click :fa:`plus-circle` to add a new Service.

   By default, the Application Overview is located in the lower right-hand corner of the Blueprint Editor and is used to create and manage Blueprint layers such as Services, Application Profiles, and Actions.

   .. figure:: images/7.png

   Note **Service1** appears in the **Workspace** and the **Configuration Pane** reflects the configuration of the selected Service.

#. Fill out the following fields:

   - **Service Name** - MySQL
   - **Name** - MySQLAHV

   .. note::
      This defines the name of the substrate within Calm. Names can only contain alphanumeric characters, spaces, and underscores.

   - **Cloud** - Nutanix
   - **OS** - Linux
   - **VM Name** - @@{User_initials}@@-MYSQL-@@{calm_array_index}@@-@@{calm_time}@@

   .. note::

     This will use the Runtime **User_initials** variable you previously provided to prepend the VM name with your initials. It will also use built-in macros to provide the array index (for scale out services) and a time stamp.

   - **Image** - CentOS_7_Cloud
   - **Device Type** - Disk
   - **Device Bus** - SCSI
   - Select **Bootable**
   - **vCPUs** - 2
   - **Cores per vCPU** - 1
   - **Memory (GiB)** - 4
   - Select **Guest Customization**

     - **Type** - Cloud-init
     - **Script** -

       .. code-block:: bash

         #cloud-config
         users:
           - name: centos
             ssh-authorized-keys:
               - @@{CENTOS.public_key}@@
             sudo: ['ALL=(ALL) NOPASSWD:ALL']

       .. note::

         When using an SSH Private Key Credential, Calm is able to decode that private key into the matching public key, and makes the decoded value accessable via the @@{Credential_Name.public_key}@@ macro. Cloud-Init is then leveraged to populate the SSH public key value as an authorized key, allowing for the corresponding private key to be used to authenticate to the host.

   - Select :fa:`plus-circle` under **Network Adapters (NICs)**
   - **NIC 1** - Primary
   - **Credential** - CENTOS

#. Click **Save**.

   .. note::

     If errors or warnings are presented after saving the blueprint, hover over the icon in the top toolbar to see a list of issues. Resolve any issues and **Save** the blueprint again.

     .. figure:: images/8.png

   Now that you have completed the deployment details for the VM associated with the service, the next step is to tell Calm how the application will be installed on the VM.

#. With the **MySQL** service icon selected in the Workspace pane, scroll to the top of the **Configuration Panel**, and select the **Package** tab.

   The Package is the configuration and application(s) installed on the Service, and is typically accomplished by executing a script on the Service VM.

#. Specify **MySQL_PACKAGE** as the **Package Name** and click **Configure install**.

   - **Package Name** - MYSQL_PACKAGE

   .. figure:: images/9.png

   Note the **Package install** field that appears on the MySQL service in the Workspace pane.

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel** to define the script that Calm will remotely execute on the MySQL Service VM:

   - **Task Name** - Install_sql
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       set -ex

       sudo yum install -y "http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm"
       sudo yum update -y
       sudo setenforce 0
       sudo sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config
       sudo systemctl stop firewalld || true
       sudo systemctl disable firewalld || true
       sudo yum install -y mysql-community-server.x86_64

       sudo /bin/systemctl start mysqld
       sudo /bin/systemctl enable mysqld

       #Mysql secure installation
       mysql -u root<<-EOF

       UPDATE mysql.user SET Password=PASSWORD('@@{Mysql_password}@@') WHERE User='@@{Mysql_user}@@';
       DELETE FROM mysql.user WHERE User='@@{Mysql_user}@@' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
       DELETE FROM mysql.user WHERE User='';
       DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';

       FLUSH PRIVILEGES;
       EOF

       mysql -u @@{Mysql_user}@@ -p@@{Mysql_password}@@ <<-EOF
       CREATE DATABASE @@{Database_name}@@;
       GRANT ALL PRIVILEGES ON homestead.* TO '@@{Database_name}@@'@'%' identified by 'secret';

       FLUSH PRIVILEGES;
       EOF

   .. figure:: images/10.png

   .. note::
      You can click the **Pop Out** icon on the script field for a larger window to view/edit scripts.

   Reviewing the script you can see the package will install MySQL, configure the credentials and create a database based on the variables specified earlier in the exercise.

#. Select the **MySQL** service icon in the Workspace pane again, select the **Package** tab in the **Configuration Panel**.

#. Click **Configure uninstall**.

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel**:

   - **Task Name** - Uninstall_sql
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       echo "Goodbye!"

   .. figure:: images/11.png

   .. note::
      The uninstall script can be used for removing packages, updating network services like DHCP and DNS, removing entries from Active Directory, etc. It is not being used for this simple example.

#. Click **Save**. You will be prompted with specific errors if there are validation issues such as missing fields or unacceptable characters.

Creating the Web Server Service
................................

You will now follow similar steps to define a web server service.

#. In **Application Overview > Services**, add an additional service.

#. Select the new service and fill out the following **VM** fields in the **Configuration Panel**:

   - **Service Name** - WebServer
   - **Name** - WebServerAHV
   - **Cloud** - Nutanix
   - **OS** - Linux
   - **VM Name** - @@{User_initials}@@-WebServer-@@{calm_array_index}@@
   - **Image** - CentOS_7_Cloud
   - **Device Type** - Disk
   - **Device Bus** - SCSI
   - Select **Bootable**
   - **vCPUs** - 2
   - **Cores per vCPU** - 1
   - **Memory (GiB)** - 4
   - Select **Guest Customization**

     - **Type** - Cloud-init
     - **Script** -

       .. code-block:: bash

         #cloud-config
         users:
           - name: centos
             ssh-authorized-keys:
               - @@{CENTOS.public_key}@@
             sudo: ['ALL=(ALL) NOPASSWD:ALL']

   - Select :fa:`plus-circle` under **Network Adapters (NICs)**
   - **NIC 1** - Primary
   - **Credential** - CENTOS

#. Select the **Package** tab.

#. Specify a **Package Name** and click **Configure install**.

   - **Package Name** - WebServer_PACKAGE

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel**:

   - **Name Task** - Install_WebServer
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       set -ex

       sudo yum update -y
       sudo yum -y install epel-release
       sudo setenforce 0
       sudo sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config
       sudo systemctl stop firewalld || true
       sudo systemctl disable firewalld || true
       sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
       sudo yum update -y
       sudo yum install -y nginx php56w-fpm php56w-cli php56w-mcrypt php56w-mysql php56w-mbstring php56w-dom git unzip

       sudo mkdir -p /var/www/laravel
       echo "server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;
       root /var/www/laravel/public/;
        index index.php index.html index.htm;
       location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
        }
        # pass the PHP scripts to FastCGI server listening on /var/run/php5-fpm.sock
        location ~ \.php$ {
        try_files \$uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)\$;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
        }
       }" | sudo tee /etc/nginx/conf.d/laravel.conf
       sudo sed -i 's/80 default_server/80/g' /etc/nginx/nginx.conf
       if `grep "cgi.fix_pathinfo" /etc/php.ini` ; then
        sudo sed -i 's/cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/' /etc/php.ini
       else
        sudo sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/' /etc/php.ini
       fi

       sudo systemctl enable php-fpm
       sudo systemctl enable nginx
       sudo systemctl restart php-fpm
       sudo systemctl restart nginx

       if [ ! -e /usr/local/bin/composer ]
       then
        curl -sS https://getcomposer.org/installer | php
        sudo mv composer.phar /usr/local/bin/composer
        sudo chmod +x /usr/local/bin/composer
       fi

       sudo git clone https://github.com/ideadevice/quickstart-basic.git /var/www/laravel
       sudo sed -i 's/DB_HOST=.*/DB_HOST=@@{MySQL.address}@@/' /var/www/laravel/.env

       sudo su - -c "cd /var/www/laravel; composer install"
       if [ "@@{calm_array_index}@@" == "0" ]; then
        sudo su - -c "cd /var/www/laravel; php artisan migrate"
       fi

       sudo chown -R nginx:nginx /var/www/laravel
       sudo chmod -R 777 /var/www/laravel/
       sudo systemctl restart nginx

   This script installs PHP and Nginx to create a web server, and then a Laravel based web application.
   It then configures the web application settings, including updating the **DB_HOST** with the MySQL IP address, accessed via the **@@{MySQL.address}@@** macro.

#. Select the **Package** tab and click **Configure uninstall**.

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel**:

   - **Name Task** - Uninstall_WebServer
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       set -ex

       sudo rm -rf /var/www/laravel
       sudo yum erase -y nginx

   For many applications it is common to need to scale out a given service, such as the web tier in order to handle more concurrent users. Calm makes it simple to turn deploy an array containing multiple copies of a given service.

#. With the **WebServer** service icon selected in the Workspace pane, scroll to the top of the **Configuration Panel**, and select the **Service** tab.

#. Under **Deployment Config > Number of Replicas**, increase the **Min** value from 1 to 2 and the **Max** value from 1 to 4.

   .. figure:: images/12.png

   This change will provision a minimum of 2 WebServer VMs for each deployment of the application, and allow the array to grow up to a total of 4 WebServer VMs.

   .. note::

     Scaling an application in and out will require additional scripting so that the application understands how to leverage the additional VMs.

#. Click **Save**.

.. _haproxyinstall:

Creating the Load Balancer Service
..................................

To take advantage of a scale out web tier, your application needs to be able to load balance connections across multiple web server VMs. HAProxy is a free, open source TCP/HTTP load balancer used to distribute workloads across multiple servers. It can be used anywhere from small, simple deployments to large web-scale environments such as GitHub, Instagram, and Twitter.

#. In **Application Overview > Services**, add an additional service.

#. Select the new service and fill out the following **VM** fields in the **Configuration Panel**:

   - **Service Name** - HAProxy
   - **Name** - HAProxyAHV
   - **Cloud** - Nutanix
   - **OS** - Linux
   - **VM Name** - @@{User_initials}@@-HAProxy-@@{calm_array_index}@@
   - **Image** - CentOS\_7\_Cloud
   - **Device Type** - Disk
   - **Device Bus** - SCSI
   - Select **Bootable**
   - **vCPUs** - 2
   - **Cores per vCPU** - 1
   - **Memory (GiB)** - 4
   - Select **Guest Customization**

     - **Type** - Cloud-init
     - **Script** -

       .. code-block:: bash

         #cloud-config
         users:
           - name: centos
             ssh-authorized-keys:
               - @@{CENTOS.public_key}@@
             sudo: ['ALL=(ALL) NOPASSWD:ALL']

   - Select :fa:`plus-circle` under **Network Adapters (NICs)**
   - **NIC 1** - Primary
   - **Credential** - CENTOS

#. Select the **Package** tab.

#. Specify a **Package Name** and click **Configure install**.

   - **Package Name** - HAProxy_PACKAGE

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel**:

   - **Name Task** - Install_HAProxy
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       set -ex

       sudo yum update -y
       sudo yum install -y haproxy
       sudo setenforce 0
       sudo sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config
       sudo systemctl stop firewalld || true
       sudo systemctl disable firewalld || true

       echo "global
        log 127.0.0.1 local0
        log 127.0.0.1 local1 notice
        maxconn 4096
        quiet
        user haproxy
        group haproxy
       defaults
        log global
        mode http
        retries 3
        timeout client 50s
        timeout connect 5s
        timeout server 50s
        option dontlognull
        option httplog
        option redispatch
        balance roundrobin
       # Set up application listeners here.
       listen admin
        bind 127.0.0.1:22002
        mode http
        stats uri /
       frontend http
        maxconn 2000
        bind 0.0.0.0:80
        default_backend servers-http
       backend servers-http" | sudo tee /etc/haproxy/haproxy.cfg

       hosts=$(echo "@@{WebServer.address}@@" | tr "," "\n")
       port=80

       for host in $hosts
         do echo " server host-${host} ${host}:${port} weight 1 maxconn 100 check" | sudo tee -a /etc/haproxy/haproxy.cfg
       done

       sudo systemctl daemon-reload
       sudo systemctl enable haproxy
       sudo systemctl restart haproxy

   Note the use of the @@{WebServer.address}@@ macro in the script above. The macro returns a comma delimited list of all IPs of the VMs within that service. The script then uses the `tr <https://www.geeksforgeeks.org/tr-command-unixlinux-examples/>`_ command to replace commas with carriage returns. The result is an array, **$hosts**, containing strings of all WebServer IP addresses. Those addresses are then each added to the **HAProxy** configuration file.

#. Select the **Package** tab and click **Configure uninstall**.

#. Select **+ Task**, and fill out the following fields in the **Configuration Panel**:

   - **Name Task** - Uninstall_HAProxy
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS
   - **Script** -

     .. code-block:: bash

       #!/bin/bash
       set -ex

       sudo
       yum -y erase haproxy

#. Click **Save**.

Adding Dependencies
+++++++++++++++++++

As our application will require the database to be running before the web server starts, our Blueprint requires a dependency to enforce this ordering.  There are a couple of ways to do this, one of which you've already done without likely realizing it.

#. In the **Application Overview > Application Profile** section, expand the **Default** Application Profile and click the **Create** Action.

   .. figure:: images/13.png

   Take note of the **Orange Orchestration Edge** going from the **MySQL Start** task to the **WebServer Package Install** task. This edge was automatically created by Calm due to the **@@{MySQL.address}@@** macro reference in the **WebServer Package Install** task. Since the system needs to know the IP Address of the MySQL service prior to being able to proceed with the WebServer Install task, Calm intelligently creates the orchestration edge for you. This requires the MySQL service to be started prior to moving on to the WebServer Install task.

#. Return to the **HAProxy Package Install** task, why are orchestration edges automatically created between the WebServer and HAProxy services?

#. Next, select the **Stop** Profile Action.

   Note that lack of orchestration edges between services when stopping an application. Why might issuing shutdown commands to all services within the application simultaneously create an issue?

#. Click on each Profile Action to take note of the current presence (or lack thereof) of the orchestration edges.

   .. figure:: images/14.png

   To resolve this, you'll manually define a dependencies between services.

#. Select the **WebServer** Service and click the **Create Dependency** icon that appears above the Service icon, and then click on the **MySQL** service.

   .. figure:: images/15.png

#. This represents that the **WebServer** service "depends" upon the **MySQL** service, meaning the **MySQL** service will start before, and stop after, the **WebServer** service.

#. Now create a dependency for the **HAProxy** service to depend on the **WebServer** service.

#. Click **Save**.

#. Re-visit the Profile Actions and confirm the edges now properly reflect the dependencies between the services, as shown below:

   .. figure:: images/16.png

   Drawing the white dependency arrows will cause Calm to create orchestration edges for all **System Defined Profile Actions** (Create, Start, Restart, Stop, Delete, and Soft Delete).

Launching and Managing the Application
++++++++++++++++++++++++++++++++++++++

#. From the upper toolbar in the Blueprint Editor, click **Launch**.

#. Specify a unique **Application Name** (e.g. *Initials*\ -CalmLinuxIntro1) and your **User_initials** Runtime variable value for VM naming.

#. Click **Create**.

   The **Audit** tab can be used to monitor the deployment of the application.

   Why don't all of the CentOS based services deploy at the same time following the download of the disk image?

#. Once the application reaches a **Running** status, navigate to the **Services** tab and select the **HAProxy** service to determine the IP address of your load balancer.

#. In a new browser tab or window, navigate to \http://<HAProxy-IP>, and verify your Task Manager application is functioning.

   .. note::

     You can also click the link in the Description of the Application.

   .. figure:: images/17.png

Takeaways
+++++++++

What are the key things you should know about **Nutanix Calm**?

- Nutanix Calm, as a native component of Prism, is built on and carries forward the benefits of the platform.  The simplicity provided by Acropolis lets Calm focus on applications, rather than trying to mask the complexity of the underlying infrastructure management.

- Calm blueprints are easy to use.  In 60 minutes you went from nothing to a full infrastructure stack deployment.  Because Calm uses standard tools for configuration - bash, PowerShell, Python, etc. - there's no new language to learn and you can immediately apply skills and code you already have.

- While not as visually impressive, even single VM blueprints can have a massive effect on customers.  One bank in India is using Calm for single-VM deployments, reducing the time to deploy these applications from 3 days to 2 hours.  Remember that many customers have little or no automation today (or the automation they have is complex/hard to understand thus limiting it's adoption).  This means that Calm can help them right now, today, instantly.

- "Multi-Cloud Application Automation and Lifecycle Management" sounds big and scary.  The 'future' sounds amazing, but many operators can't see the path to there.  Listen to what the customer is struggling with today (backups require specialized skills, VM deployment takes a long time, upgrades are hard) and speak to how Calm can help with that; jumping right to the multi-cloud automation story pushes Calm from a "I need this right now" to a "well let's evaluate this later on, once things have quieted down" (and things never truly 'quiet down'.

- The Blueprint Editor provides a simple UI for modeling potentially complex applications.

- Blueprints are tied to SSP Projects which can be used to enforce quotas and role based access control.

- Having a Blueprint install and configure binaries means no longer creating specific images for individual applications. Instead the application can be modified through changes to the Blueprint or installation script, both of which can be stored in source code repositories.

- Variables allow another dimension of customizing an application without having to edit the underlying Blueprint.

- There are multiple ways of authenticating to a VM (keys or passwords), which is dependent upon the source image.

- Application status can be monitored in real time.

- Applications typically span across multiple VMs, each responsible for different services. Calm is capable of automated and orchestrating full applications.

- Dependencies between services can be easily modeled in the Blueprint Editor.

- Users can quickly provision entire application stacks for production or testing for repeatable results without time lost to manual configuration.

- Interested in using Calm for more app lifecycle operations? Check out the :ref:`calm_day2`!


.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: images/blueprints.png
.. |applications| image:: images/blueprints.png
.. |projects| image:: images/projects.png
