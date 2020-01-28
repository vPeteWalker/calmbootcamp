.. _karbon_cicd:

------------------------------
Nutanix Calm and Karbon: CI/CD
------------------------------

*The estimated time to complete this lab is ?? minutes.*

Overview
++++++++

Countless studies have shown that reducing the amount of time for developers to receive feedback on code changes improves software quality.  Automating the build, test, and deployment of software with tools such as Jenkins is one of the best ways to accelerate software development.  If you’re unfamiliar with Jenkins you can learn more on their website here_.

.. _here: https://www.cloudbees.com/jenkins/about

In this lab, we'll utilize Nutanix Calm to build the infrastructure required to create a Continuous Integration / Continuous Delivery (CI/CD) Pipeline, which includes deploying Jenkins, a Gitea_ git server, a developer workstation, and configuring that workstation to manage a Karbon Kubernetes cluster.  Once the infrastructure is deployed via Nutanix Calm, we'll go through configuring Jenkins and Gitea to create a fully function CI/CD pipeline.  The end result will be a developer running a "git push", which triggers Jenkins to build a docker container based on the new code, publishes that container to DockerHub, and then deploys that container onto the Nutanix Karbon Kubernetes cluster.

.. _Gitea: https://gitea.io/en-us/

.. figure:: images/01_cicd_pipeline.png
    :align: center
    :alt: CI/CD with Gitea, Jenkins, and Nutanix Karbon

Pre-requisites
++++++++++++++

This lab requires:

- a **running Nutanix Karbon Kubernetes cluster**.  When the Calm blueprint is launched in the next step, a runtime variable will prompt you for the Karbon Kubernetes cluster name.  If you do not have a cluster already deployed, please go do that and then return to this lab.
- a **DockerHub account**.  If you do not already have one, go ahead and sign up for an account_ (it's free).

.. _account: https://hub.docker.com/

All other required components are deployed via a Calm blueprint.

Creating our CI/CD Infrastructure with Nutanix Calm
+++++++++++++++++++++++++++++++++++++++++++++++++++

Building a CI/CD pipeline generally involves connecting a large number of disparate tools.  To streamline this lab, we'll be deploying many of these tools via a Nutanix Calm blueprint.  We'll first launch the blueprint, and then in the ~20 minutes it takes to deploy, we'll cover the blueprint architecture to familiarze ourselves with the tools involved.

#. In **Prism Central**, select :fa:`bars` **> Services > Calm**.

#. Select |blueprints| **Blueprints** in the left hand toolbar to view and manage Calm blueprints.

   .. note::

     Mousing over an icon will display its title.

#. Find the blueprint titled **CICD_Infra**, select its checkbox, and from the **Action** dropdown, click **Launch**.

   .. figure:: images/02_bp_launch_1.png
       :align: center
       :alt: Nutanix Calm CI/CD Infrastructure Blueprint Launch 1

#. On the launch page, fill in the following fields.

   - **Name of the Application** - *initials*-cicd-infra
   - **gitea_password** - Any password desired, which will be set as the Gitea admin user password
   - **karbon_cluster_name** - The name of the Karbon cluster to use for this lab (it **must** already be depoyed)

   .. figure:: images/03_bp_launch_2.png
       :align: center
       :alt: Nutanix Calm CI/CD Infrastructure Blueprint Launch 2

#. Click the blue **Create** button, and ensure you're redirected to the application page.

Now that we're waiting for our CI/CD Infrastructure to deploy, let's review the architecture of the blueprint.  If desired, open the blueprint in Calm and view the Services and their underlying scripts as they're covered.  Alternatively, here's an image of the blueprint canvas.

.. figure:: images/04_calm_cicd_infra_bp.png
    :align: center
    :alt: Nutanix Calm CI/CD Infrastructure Blueprint Architecture

In approximate order (approximate as Calm deploys Services in parallel, unless there is a dependency), the blueprint deploys the following Services:

- **Kubernetes** (Existing machine) - this Service utilizes VM Pre-create eScript tasks to make API calls into Prism Central, to find a Karbon Kubernetes cluster matching the **karbon_cluster_name** variable defined at launch.  It also sets the content of the cluster's kubeconfig_ as a variable, which will be later applied to the Workstation VM.
- **Gitea** (AHV) - this Service installs Gitea, which is a community managed lightweight code hosting solution.  It first installs MySQL, as Gitea requires a backend DB to operate.  It then creates self signed certificates, installs the Gitea service, and configures the repo which stores our application code.
- **Jenkins_Master** (AHV) - this Service installs Jenkins, a popular Continuous Integration server.  It also trusts Certificate Authority (CA) generated during the Package Install of the **Gitea** Service.
- **Jenkins_Slave** (AHV) - this Service installs a Jenkins Slave, which is used for builds in our Jenkins pipeline.  Meaning this is the node that is responsible for building a docker container based on the new application code, publishing tha container to DockerHub, and then deploying the new container to the Kubernetes cluster.
- **Workstation** (AHV) - this Service represents a "developer workstation," and is where we'll be making changes to our application code later in this lab.  It first installs necessary software (like *git* and  *kubectl*), and then configures the kubeconfig based on the variable set in the **Kubernetes** Service.  Finally, it clones the git repo configured in the **Gitea** Service.

.. _kubeconfig: https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/

Once your Jenkins_Master and Jenkins_Slave Services have been fully deployed, move on to the next section.


Accessing Jenkins
+++++++++++++++++

Now that our CI/CD Infrastructure has been deployed, we're ready to start configuration of the various components that make up our Pipeline.  First up, is Jenkins.

#. On the **Overview** tab of your Application, **right click** on the **Jenkins** link, and open the page in a new tab.

   .. figure:: images/05_app_overview.png
       :align: center
       :alt: Nutanix Calm CI/CD Infrastructure App Overview

#. While still within the Calm Application page, navigate to the **Services** tab, select the **Jenkins_Master** Service, and in the right column, click **Open Terminal**.

   .. figure:: images/06_open_terminal.png
       :align: center
       :alt: CI/CD Infrastructure App Open Terminal

#. In the Web SSH Terminal that just opened, run the following command to print out Jenkins' temporary administrator password.

    .. literalinclude:: cat-password.sh
       :language: bash

#. Double click the result from the previous step's command to copy it to your clipboard.

   .. figure:: images/07_temp_admin_pass.png
       :align: center
       :alt: Jenkins Master Temporary Admin Password

#. Change to the **Sign in [Jenkins]** tab that was previously opened.  In the **Administrator password** field, paste in the contents of the previous step, and click **Continue**.

   .. figure:: images/08_unlock_jenkins.png
       :align: center
       :alt: Unlock Jenkins

#. On the next page, click the large **Install suggested plugins** button.

   .. figure:: images/09_suggested_plugins.png
       :align: center
       :alt: Install Jenkins Suggested Plugins

#. Wait for the suggested plugins to install, after which you'll be re-directed to create the first admin user.  Fill in the following fields, and click **Save and Continue**.

   - **Username** - admin
   - **Password** - any password of your choice
   - **Confirm password** - matching password
   - **Full name** - admin
   - **Email address** - noreply@nutanix.com

   .. figure:: images/10_create_user.png
       :align: center
       :alt: Create Jenkins Admin User

#. On the Instance Configuration page that appears, **leave** the Jenkins URL as **default**, and click **Save and Finish**.

#. Jenkins setup is now complete, but first our Jenkins instance needs to be restarted.  Click **Restart**, and then move on to the next section.

   .. figure:: images/11_restart_jenkins.png
       :align: center
       :alt: Restart Jenkins


Gitea Webhook Setup
+++++++++++++++++++


Jenkins Credentials Creation
++++++++++++++++++++++++++++


Jenkins Pipeline Creation
+++++++++++++++++++++++++


Jenkins Pipeline Snippet Generator
++++++++++++++++++++++++++++++++++


Jenkinsfile and Yaml Creation
+++++++++++++++++++++++++++++


Manual Build and Application Deployment
+++++++++++++++++++++++++++++++++++++++


Automated Application Deployment Through a Git Push
+++++++++++++++++++++++++++++++++++++++++++++++++++



Takeaways
+++++++++



.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: images/blueprints.png
.. |applications| image:: images/blueprints.png
.. |projects| image:: images/projects.png
