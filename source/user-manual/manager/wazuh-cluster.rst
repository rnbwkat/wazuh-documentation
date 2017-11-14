.. _wazuh-cluster:

Deploying a Wazuh cluster
=========================

The Wazuh cluster has been developed in order to get a stronger communication between agents and managers. This means that the cluster is able to synchronize the necessary files between
its manager to allow agents for reporting events to any manager of the cluster.

- `Why do we need a Wazuh cluster?`_
- `How it works`_
- `Use case: Deploying a Wazuh cluster`_

Why do we need a Wazuh cluster?
-------------------------------

The Wazuh cluster provides horizontal scalability to our Wazuh environment, allowing agents to report to any manager belonging to the cluster. In this way, we have the ability
to ingest more events than before distributing the load of information between several managers simultaneously.

In addition, a cluster of Wazuh managers is prepared to handle the fall of any manager without affecting its operation, unless it is the master manager.
Agents that were reporting to this fallen manager will start to report to another manager of the cluster automatically, without the loss of event.

Finally, the Wazuh cluster is in continuous development and we hope it to include many new features very soon. For example, we refer to the possibility of
switching the role of master between the different managers providing high availability.


How it works
------------

The Wazuh cluster is managed by a cluster daemon which communicates managers, following a master-client architecture.

Master
^^^^^^^^

The master node is the manager who has the control of the cluster. This means that every configuration done in the master node is pushed to the others managers, and it allows
to centralize the following configurations:

- Agents registration and deletion.
- Local configuration for managers, sychronizing its ``ossec.conf`` and ``internal_options.conf`` files.
- Centralized configuration for agents, synchronizing its ``agent.conf`` file.
- More customization options, such as rootcheck files, custom rules and decoders, etc.

To sum up, the master node sends to its clients the whole ``etc`` folder contained in the Wazuh installation directory, with
its ``client.keys`` file allowing to connect each agent to each manager of the cluster. For all the communications in the cluster,
it has been developed an own protocol which synchronize those files with a specific interval time
defined in the ``<cluster>`` section of :doc:`Local configuration <../reference/ossec-conf/cluster>`.
These communications are encrypted with AES providing confidentiality.

Client
^^^^^^^^

Managers which have the client role in the cluster receive the data from the master node, and they update their files with the received ones. In this way, we make sure that they will always
contain the same configuration.

On the other hand, client nodes send to the master the ``agent-info`` file. This file contains the most important information of each agent, and allows to the master node to know in real-time
the connection state of agents, as well as which manager is connected to each agent.

Cluster daemons
^^^^^^^^^^^^^^^^^

- **wazuh-clusterd** is the module that manage the synchronization between managers in the cluster. This daemon has to be running in all the managers of the cluster in order to ensure the correct communication between them. There exits a file located at  ``logs/cluster.log`` where can be found all the logging messages of the cluster.

- **wazuh-clusterd-internal** is the daemon in charge of manage the cluster database and the files located in each manager.

In the section :doc:`Daemons <../reference/daemons/index>` can be found more information about the usage of these daemons.

Cluster database
^^^^^^^^^^^^^^^^^

It has been incorporated a database for each manager in the cluster called `cluster.db`. In this database is stored information about the state of each synchronized
file for each manager in the format ``<node> <file> <state>``.


Use case: Deploying a Wazuh cluster
-----------------------------------

Any use case that shows how the cluster works in a practical way. Setting a specific configuration and showing the result of use it.
