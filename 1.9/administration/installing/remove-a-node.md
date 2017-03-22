---
post_title: Removing a Node
menu_order: 801
---

You can remove agent nodes from an active DC/OS cluster. You can remove agent nodes by using maintenance windows or terminate signal. You need to remove an agent node if you are:

- Downsizing a cluster
- Draining a node before reconfiguring the Mesos agent
- Moving a node to a new IP (remove + add)

### Maintenance windows
With maintenance windows you can drain multiple nodes at the same time from outside the cluster. SSH access is not required.

### Terminate signal
Draining nodes by using terminate signal, SIGUSR1, is easy to integrate with automation tools that can execute tasks on nodes in parallel, for example Ansible, Chef, and Puppet. 

# Removing a node by using maintenance windows

### Prerequisites:

*   DC/OS is installed.
*   Access to the [Admin Router permissions](/docs/1.9/overview/architecture/components/#admin-router).

You can define a maintenance schedule to evacuate your tasks prior to changing agent attributes or resources. ⁠⁠⁠All tasks that are running on the agent will be killed when you change agent attributes or resources. Mesos treats re-registered agents as new agents.

When you change Mesos attributes (`⁠⁠⁠⁠/var/lib/dcos/mesos-slave-common`⁠⁠⁠⁠) or resources (⁠⁠⁠⁠`/var/lib/dcos/mesos-resources`⁠⁠⁠⁠), you must remove the agent node and re-register it with the master node under a new UUID. The master will then recognize the new attributes and resources specification.

1.  Define a maintenance schedule. For example, here is a basic maintenance schedule JSON file:
    
    ```json
    {
      "windows" : [
        {
          "machine_ids" : [
            { "hostname" : "10.0.2.107", "ip" : "10.0.2.107" },
            { "hostname" : "10.0.2.5", "ip" : "10.0.2.5" }
          ],
          "unavailability" : {
            "start" : { "nanoseconds" : 1 },
            "duration" : { "nanoseconds" : 3600000000000 }
          }
        }
      ]
    }
    ```
    
    For a more complex example, see the [maintain-agents.sh](https://github.com/vishnu2kmohan/dcos-toolbox/blob/master/mesos/maintain-agents.sh) script.
 
1.  Invoke the `⁠⁠⁠⁠machine/down` endpoint with a JSON definition that includes the agents. For example, [here](https://github.com/vishnu2kmohan/dcos-toolbox/blob/master/mesos/down-agents.sh) is how you might call `/machine/down/`. 

    **Important:** Invoking `machine/down` sends a `⁠⁠⁠⁠TASK_LOST`⁠⁠⁠⁠ message for any tasks that were running on the agent. Some DC/OS services, for example Marathon, will relocate tasks. However some DC/OS services will not relocate tasks, for example Kafka and Cassandra.  For more information, see the DC/OS [service guides](https://docs.mesosphere.com/service-docs/) and the Mesos maintenance primitives [documentation](https://mesos.apache.org/documentation/latest/maintenance/).

# Removing nodes by using signal

### Prerequisites:

*   DC/OS is installed.
*   SSH installed and configured. This is required for accessing nodes in the DC/OS cluster. <!-- not required for maint window -->
*   Access to the [Admin Router permissions](/docs/1.9/overview/architecture/components/#admin-router).

**Warning:** ⁠⁠⁠All tasks that are running on the agent will be killed since you are re-registering a UUID. Mesos treats a re-registered agent as a new agent.

1.  [SSH to the agent node](/docs/1.9/administration/access-node/sshcluster/).
1.  Stop the agent.

    -  **Private agent**
    
       ```bash
       $ sudo sh -c 'systemctl kill -s SIGUSR1 dcos-mesos-slave && systemctl stop dcos-mesos-slave
       ```
    -  **Public agent**
    
       ```bash
       $ ⁠⁠⁠⁠sudo sh -c 'systemctl kill -s SIGUSR1 dcos-mesos-slave-public && systemctl stop dcos-mesos-slave-public
       ```

1.  Reload the systemd configuration.

    ```bash
    $﻿⁠⁠⁠⁠sudo systemctl daemon-reload
    ```
    
1.  Remove the `latest` metadata pointer on the agent node:

    ```bash
    ⁠⁠⁠⁠sudo rm /var/lib/mesos/slave/meta/slaves/latest
    ```
    
1.  Start your agents with the newly configured attributes and resource specification⁠⁠.

    -  **Private agent**
    
       ```bash
       $ sudo systemctl start dcos-mesos-slave﻿⁠⁠⁠⁠
       ```﻿⁠⁠⁠⁠
       
    -  **Public agent**
    
       ```bash
       ⁠⁠⁠⁠$ sudo systemctl stop dcos-mesos-slave-public﻿⁠⁠⁠⁠
       ```
       
1.  After the agent service is up and running, invoke the `/machine/up` endpoint. This instructs the master to allow the agent to register.