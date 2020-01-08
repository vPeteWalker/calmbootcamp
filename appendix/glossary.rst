.. glossary:

------------
Glossary
------------

There are a multitude of familiar industry concepts and overloaded terms which have specific meanings when used by Calm.
Refer to this glossary of Calm terms to help orient your discovery and mastery.
Please see the :ref:`Additional Resources <additional-resources>` for deeper exploration!

Calm Terms
++++++++++++++++++++

.. glossary::

    Action
        A set of seqentially executed :term:`tasks <task>` in a :term:`blueprint`.
        Basic actions included in every blueprint consist of: Create, Delete, Stop, Start, Restart.
        A developer role can create additional custom actions (such as a runbook), scale-in and scale-out actions, pre- and post- create actions.

    Application
        A deployment instance of a :term:`blueprint`.
        A top level Calm menu icon.

    Application Profile
        A developer role can bundle and name a set of :term:`blueprint` variables and service creation definitions into a application profile.
        An operator role can choose an application profile during blueprint launch for deployment choice.
        Examples: service resource sizes (small, medium, large VM), deployment scenarios for development, staging, production credentials.

    Blueprint
        An application model of :term:`service` topology with orchestrational :term:`dependency` of :term:`action` and :term:`task`,
        a blueprint must contain at least one :term:`service` and default :term:`application profile`.
        A top level Calm menu icon.

    Brownfield
        A blueprint consisting of imported :term:`existing machine` from a :term:`provider`.

    Calm
        Calm is an application lifecycle manager, which provides an automation platform to model your business governance, applications, and infrastructure together as a single artifact: the blueprint.
        Currently, Calm is delivered in Nutanix :term:`Prism Central` and enabled with one-click.

    Dependency
        A logical sequential connection between services, orchestrating their basic actions together, indicated by a white arrow.
        A generated or developer specified sequential connection between :term:`task` (across services in the same action), indicated by an orange arc.

    Epsilon
        The workflow engine of Calm, it is an internal component controlled by Life Cycle Manager, and used by other Prism Central facilites.

    eScript (Epsilon Script)
        A Calm :term:`task` type to execute a local, restricted Python environment.

    Existing Machine
        An existing service (not provisioned by Calm) which can be managed by :term:`task`.

    Macro
        Either a Calm built-in or blueprint specified variable that can be used in :term:`task` and variables.

    Marketplace
        An end-user, self-service portal of published Calm blueprints, controlled by projects and roles, in Prism Central.
        A top level Calm menu icon.

    Prism Central
        The Nutanix scale-out control plane to manage multiple joined Nutanix clusters and provide
        advanced management capabilites (Karbon, Calm, Flow, etc.) from a single pane of glass web console.

    Project
        A combined bundle of users or groups associated with roles who can access providers
        with provider resource quotas.
        A top level Calm menu icon.

    Provider
        An infrastructure host that provides services with specified show back resource costs.
        Specified in Calm > Settings.

    Role
        A set of permissions that define a user's abilities. e.g.: start a VM, create a Calm blueprint.

    Runtime
        A runtime property allows the specified property to be overriden at blueprint or action launch.

    Secret
        A credential password or key, an application profile or service variable specified with a secret property.
        Secrets are specified by the developer role, they are hidden when a blueprint is saved to prevent stealing.
        Secrets are blanked when a blueprint is exported and secrets are blanked out in :term:`task` output.

    Service
        An instance of a substrate, typically a virtual machine, existing machine, or a Kubernetes pod.

    Substrate
        An infrastructure provider, a blueprint can use all the providers enabled in a project.

    Task
        An individual stage of operational execution in an :term:`action`.

    Variable
        A blueprint property: they can be statically set by the developer role with a default value,
        used as a macro in :term:`task`, and specified with a :term:`runtime` property to delegate setting by an operator role during blueprint launch.

.. _additional-resources:

Additional Resources
++++++++++++++++++++

#. `Nutanix Calm Admin Operations Guide: Major Components <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Calm-Admin-Operations-Guide-v297:nuc-nucalm-major-components-c.html>`_
#. `Calm Terminology <https://next.nutanix.com/blog-40/calm-terminology-actions-and-dependencies-33852>`_ `[Source] <https://github.com/MichaelHaigh/calm-blueprints/blob/master/DependencyTaskExample/README.md>`_
