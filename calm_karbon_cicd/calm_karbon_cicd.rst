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


Accessing the Developer Workstation
+++++++++++++++++++++++++++++++++++

Throughout this entire lab, we'll be running a large number of commands from our developer workstation, as it has already been configured with all the necessary software packages, the correct kubeconfig file, and is pointed at our git repository that's stored in Gitea.

To access the developer workstation, you have two options: 1, use the web SSH client as we did for the Jenkins Service, or 2, use your laptop's terminal or PuTTY to SSH into the workstation.  Either option is perfectly valid, however we recommend you stick with whatever you're most comfortable with.  Since we already covered how to use the web SSH client in the previous step, we'll cover SSH'ing in from your laptop here.

#. Back in our Calm application page, navigate to the **Services** tab, and select the **Workstation** Service.  In the right column that appears, **copy** the IP address of the service by clicking the button just to the right of the IP.

   .. figure:: images/12_copy_workstation_ip.png
       :align: center
       :alt: CI/CD Infrastructure App Copy Workstation IP

#. In your laptop's terminal, run the following commands (TODO: figure out how we're handling keys) to SSH into your workstation.

    .. literalinclude:: ssh-workstation.sh
       :language: bash

#. Validate that our kubeconfig and git repo are set up properly by running the following commands.  Your output should be similar to the image below, but with different node names and IPs.

    .. literalinclude:: validate-workstation.sh
       :language: bash

   .. figure:: images/13_validate_workstation.png
       :align: center
       :alt: Validate Workstation Configuration


Gitea Webhook Setup
+++++++++++++++++++

Our next configuration step is to create a webhook in Gitea, which tells Gitea to inform some server (in our case Jenkins) each time there is a new commit.  Many popular git servers have this functionality, including GitHub, GitLab, and Gitea.

#. We'll access our Gitea Service in the same manner as Jenkins, by navigating to the **Overview** tab of our Calm application, right clicking on the **Gitea** link, and opening it in a new tab.

#. It is expected to receive a warning from your browser about the site's security certificate not being trusted by your computer.  This is due to the use of self signed SSL certificates during setup (which is not recommended for production workloads).  Select the **Proceed Anyway** option (exact wording may depend on your browser).

#. On the Gitea homepage, click the **Sign In** button in the upper right.

   .. figure:: images/14_gitea_home.png
       :align: center
       :alt: Gitea Homepage

#. Sign in with the following credentials.

   - **Username** - gitadmin
   - **Password** - your password specified when launching the Calm blueprint

   .. figure:: images/15_gitea_signin.png
       :align: center
       :alt: Gitea Sign In

#. On the page that appears, click the **gitadmin/hello-kubernetes** repository link, then **Settings** along the right-hand side, and finally the **Webhooks** tab.

   .. figure:: images/16_repo_settings.png
       :align: center
       :alt: Gitea Repository Settings

#. Click the blue **Add Webhook** button, in the list that appears click **Gitea**, and then fill in the following fields.

   - **Target URL** - The output of the **echo $JENKINS_HOOK_URL** command from the previous "Developer Workstation" section, should be of the format **http://<jenkins-ip>:8080/gitea-webhook/post**
   - **HTTP Method** - Leave the default of **POST**
   - **POST Content Type** - Leave the default of **application/json**
   - **Secret** - Leave it blank (Jenkins does not require a secret by default)
   - **Trigger On** - Leave the default of **Push Events** (any time a user runs "git push" Gitea will send the webhook)
   - **Branch filter** - Leave the default of * (this means the webhook will be triggered for *any* branch)
   - **Active** - Leave the Active checkbox **enabled**.

   .. figure:: images/17_gitea_add_webhook.png
       :align: center
       :alt: Gitea Add Webhook

#. Click the green **Add Webhook** button.  You should receive a notification that the webhook has been added.

   .. figure:: images/18_gitea_webhook_added.png
       :align: center
       :alt: Gitea Webhook Added

#. To validate the webhook is operating as expected, click the **pencil** to the right of the webhook.  Scroll all the way to the bottom of the page, and click the teal **Test Delivery** button.  After a moment, the page should refresh, and there should be a successful test event created.  If the **Response** has a green **200** code, then everything is configured properly.

   .. figure:: images/19_gitea_test_webhook.png
       :align: center
       :alt: Gitea Successful Test Webhook

DockerHub Setup
+++++++++++++++

After a GitHub commit triggers a Jenkins build, and Jenkins successfully builds our new docker image, it needs some place to store the image.  In this lab, we'll be using DockerHub, however there are many free container registries available.

#. First, login to DockerHub_ (or create a free account) and click the **Create Repository** button.

  .. _DockerHub: https://hub.docker.com/

   .. figure:: images/20_dockerhub_create_1.png
       :align: center
       :alt: DockerHub Create Repository Button

#. Name the repository **hello-kubernetes**, give it a description of your choice, leave all other fields as default, and click **Create**.

   .. figure:: images/21_dockerhub_create_2.png
       :align: center
       :alt: DockerHub Create Repository


Jenkins Credentials Creation
++++++++++++++++++++++++++++

The first step of our Jenkins Setup is to add our various credentials to Jenkins’ credential store, which gives Jenkins the ability to authenticate to other pieces of our pipeline.  We’ll first add our DockerHub credentials, which allows Jenkins to push images.  **TODO: Validate this statement. In many environments, you would also need to add git credentials for Jenkins to be able to read the repository, however in this particular environment, our Gitea server has our git repository marked as public, so no authentication is necessary to read the repo.**  Lastly, we’ll add our Karbon kubeconfig file to allow Jenkins to deploy our application directly onto our Kubernetes cluster.

#. Log in to your Jenkins server with the credentials you created earlier (you may need to refresh your browser page due to the Jenkins reboot in a previous section).

#. In the Jenkins UI, select **Credentials** along the left, and then in the **Stores scoped to Jenkins** section, select the **global** domain.

   .. figure:: images/22_jenkins_global_creds.png
       :align: center
       :alt: Jenkins Global Credentials

#. Click **Add Credentials** along the left column.

   .. figure:: images/23_jenkins_add_creds.png
       :align: center
       :alt: Jenkins Add Global Credentials

#. Fill in the following fields to add your DockerHube credentials, and click **OK**.

   - **Kind** - leave as default (**Username with password**)
   - **Scope** - leave as default (**Global**)
   - **Username** - your DockerHub username (**not** your email)
   - **Password** - your DockerHub password
   - **ID** - leave blank
   - **Description** - **DockerHub Credentials**

   .. figure:: images/24_jenkins_dockerhub_creds.png
       :align: center
       :alt: Jenkins Add DockerHub Credentials

#. Lastly, we’ll need to add our kubeconfig file as a credential to allow Jenkins to deploy our updated application onto our Kubernetes cluster.  In our Workstation CLI, run the following commands  to create a Kubernetes Service Account **jenkins**, and then create a Role Binding which maps our Service Account the the built-in **admin** role. 

    .. literalinclude:: create-sa.sh
       :language: bash

   .. note::

     We're limiting our jenkins Service Account to a single Kubernetes namespace (default).

#. We'll now replace the token in our existing kubeconfig with the token of our newly generated Service Account, which we can do in one line with the following command.

    .. literalinclude:: create-kubeconfig.sh
       :language: bash

#. Copy the long output of that command into your buffer, and head back into the Jenkins UI.  Select **Add Credentials** again, fill in the following fields, and click **OK**.

   - **Kind** - **Kubernetes configuration (kubeconfig)**
   - **Scope** - leave as default (**Global**)
   - **ID** - leave blank
   - **Description** - **Karbon Kubernetes Kubeconfig**
   - **Kubeconfig** - select the **Enter directly** radio button
   - **Content** - paste in the output from the previous step

   .. figure:: images/25_jenkins_kubconfig.png
       :align: center
       :alt: Jenkins Add Kubeconfig Credential


Jenkins Pipeline Creation
+++++++++++++++++++++++++

It's now time to create our Jenkins Pipeline.  The pipeline is the crux of this entire CI/CD workload: our Gitea webhook calls this pipeline, which is then responsible for building our docker container, uploading the container to DockerHub, and deploying the new container to our Karbon Kubernetes cluster.

#. In the Jenkins UI, click **New Item** in the upper left, enter **helo-kubernetes** as the name, select **Pipeline**, and click **OK**.

   .. figure:: images/26_jenkins_create_pipeline_1.png
       :align: center
       :alt: Jenkins Create Pipeline 1

#. Under the **General** section, give your pipeline a description, and leave all checkboxes as **unselected**.

   .. figure:: images/27_jenkins_create_pipeline_2.png
       :align: center
       :alt: Jenkins Create Pipeline 2

#. Under the **Build Triggers** section, select **Poll SCM**, and leave the Schedule **blank**.  Without a schedule, Jenkins will *only* run this pipeline from a Webhook, which is desired for this setup.  Leave all other checkboxes as **unselected**.

   .. figure:: images/28_jenkins_create_pipeline_3.png
       :align: center
       :alt: Jenkins Create Pipeline 3

#. Skip the **Advanced Project Options** section.

#. Under the **Pipeline** section, fill in the following fields.

   - **Definition** - Change the dropdown to **Pipeline script from SCM**, which allows us to store our Jenkinsfile in the same source code repository as our application
   - **SCM** - Change the dropdown to **git**
   - **Repositories**

     - **Repository URL** - Fill in your Gitea repository URL, which can be found by running **echo $GIT_REPO_URL** from your Workstation, and should be of the format **https://<gitea-ip>:3000/gitadmin/hello-kubernetes**
     - **Credentials** - Leave as default **none** (if your git repository is private, you would need to specify your git credentials here)

   - **Branches to build** - Leave all as default
   - **Repository browser** - Leave as default of **Auto**
   - **Additional Behaviours** - Leave default of none
   - **Script Path** - Leave as default of **Jenkinsfile**
   - **Lightweight checkout** - Leave as default **checked**

   .. figure:: images/29_jenkins_create_pipeline_4.png
       :align: center
       :alt: Jenkins Create Pipeline 4

#. Click **Save** to save the pipeline configuration.


Jenkins Pipeline Snippet Generator
++++++++++++++++++++++++++++++++++

We'll now use the Jenkins Pipeline Syntax Snippet Generator to assist us when we go to create our Jenkinsfile in the upcoming section.  Since the result of this section is a text string which will be included in our Jenkinsfile (and will be provided in the next section), it’s not required to perform the same steps on your system.  However, it is good practice as it’s something you’ll likely need to do if you expand upon this example.

#. Within your pipeline homepage, click the **Pipeline Syntax** button in the left column, and fill out the following fields.

   - **Sample Step** - change the dropdown to **kubernetesDeploy: Deploy to Kubernetes**
   - **Kubeconfig** - select the Kubeconfig that was added in a previous section
   - **Config Files** - enter **hello-kubernetes-dep.yaml** (we have not created this file yet, but will in an upcoming section)
   - Leave all other options as **defaults**

   .. figure:: images/30_jenkins_gen_pipeline.png
       :align: center
       :alt: Jenkins Generate Pipeline Script

#. Click **Generate Pipeline Script**.  In the text box that appears, you should see a string like this, however your **kubeconfigId** *will be different*.  When this string is placed in a Jenkinsfile, it instructs Jenkins to deploy a certain configuration (hello-kubernetes-dep.yaml) against a particular Kubernetes cluster (in our case the cluster config is stored in the kubeconfig credential we created in an earlier section).

    .. literalinclude:: gen-pipeline-script.sh
       :language: bash

   .. note::

     The serverUrl field does not need an actual URL as that information is stored in our Kubeconfig.

Jenkinsfile and Yaml Creation
+++++++++++++++++++++++++++++



   .. figure:: images/.png
       :align: center
       :alt:

Manual Build and Application Deployment
+++++++++++++++++++++++++++++++++++++++


Automated Application Deployment Through a Git Push
+++++++++++++++++++++++++++++++++++++++++++++++++++



Takeaways
+++++++++


   .. figure:: images/.png
       :align: center
       :alt:


.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: images/blueprints.png
.. |applications| image:: images/blueprints.png
.. |projects| image:: images/projects.png
