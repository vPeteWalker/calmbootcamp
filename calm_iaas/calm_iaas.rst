.. _calm_iaas:

---------------------------------
Calm: Infrastructure as a Service
---------------------------------

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

   - **Disks** - A disk is the storage of the VM or infrastructure that we're deploying.  It could be based on a pre-existing image (as it will in our case), and/or it could be based on a blank disk to enable the VM to consume additional storage.  For instance, a Microsoft SQL server may need its base OS disk, a separate SQL Server binary disk, separate database data file disks, separate TempDB disks, and a separate logging disk.  In our case we're going to have a single disk, based on a pre-existing image.

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

     .. figure:: images/11_windows_5.png
         :width: 400px
         :align: center
         :alt: Windows 2016 VM Configuration - Disks

         Windows 2016 VM Configuration - Disks

   - **Boot Configuration** - The boot method of the VM.  We'll leave the default of **Legacy BIOS**.

   - **vGPUs** - Whether or not the VM needs a virtual graphical processing unit.  We'll leave the default of none.

   - **Categories** - Categories span several different products and solutions within the Nutanix portfolio.  They enable you to set security policies, protection policies, alert policies, and playbooks.  Simply choose the categories corresponding to the workload, and all of these policies will automatically be applied.  In this lab however, we're going to leave this field **blank**.

   .. figure:: images/12_boot_gpu_cat.png
       :width: 400px
       :align: center
       :alt: VM Configuration - Boot Configuration, vGPUs, and Categories

       VM Configuration - Boot Configuration, vGPUs, and Categories

   - **NICs** - Network adapters allow communication to and from your virtual machine.  We'll be adding a single NIC by clicking the **blue plus**, and then selecting **Primary** in the dropdown.

   .. figure:: images/13_vm_nic.png
       :width: 400px
       :align: center
       :alt: VM Configuration - NICs

       VM Configuration - NICs

   - **Serial Ports** - Whether or not the VM needs a virtual serial port.  We'll leave the default of none.

   .. figure:: images/14_serial.png
       :width: 400px
       :align: center
       :alt: VM Configuration - Serial Ports

       VM Configuration - Serial Ports

#. At the bottom of the page, click the blue **Save** button.  It is expected to have a single error about an incorrect macro due to our Guest Customization containing "vm_password".  If you have additional errors, please be sure to resolve them before continuing to the next section.

   .. figure:: images/15_error.png
       :width: 400px
       :align: center
       :alt: Blueprint Save - Error

       Blueprint Save - Error


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
