---
nav_title: Using Containerizers
menu_order: 40
---

A containerizer is a Mesos agent component responsible for launching containers, within which you can run a service. Running services in containers offers a number of benefits, including the ability to isolate tasks from one another and control task resources programmatically.

DC/OS supports the Mesos containerizer types:

- The [original Mesos containerizer](/docs/1.9/usage/containerizers/mesos-containerizer/).

- The [DC/OS Universal Container Runtime](/docs/1.9/usage/containerizers/ucr/).

- The [Docker containerizer](/docs/1.9/usage/containerizers/docker-containerizer/).

The tables below provide a feature comparison of your containerizer choices on DC/OS.

## DC/OS Features

| 																				| Docker			| Original Mesos				| UCR 			| Comments |
| --------------------------------------- | ----------- | --------------------- | --------- | -------- |
| **Command**  what is this??              | Yes         | ???                   | Yes
| **Container Images** 										| Yes					| No										| Yes				|					 |
| **Docker Containers** 									| Yes					| No										| Yes				|					 |
| **Pods**																| No					| No										| Yes				|					 |
| **GPUs**																| No					| No										| Yes				|					 |
| **DC/OS CLI**														| No					| Yes										| Yes				|					 |
| **URIs**																| Yes					| Yes										| Yes				|					 |
| **Runtime Privileges** ??is this the same as "privileged mode"?								| Yes					| No										| No				|					 |
| **Docker Options**											| Yes					| No										| No				|					 |
| **Force Pull**													| Yes					| No										| No				|					 |
| **Secrets**															| Yes					| Yes										| Yes				| Enterprise only |
| **Image GC** what is this??             | Yes         | ??                    | No        |          |
| **Debugging with exec**         				| No					| ??										| Yes				|	CLI only |
| **All Security Modes**									| Yes					| Yes										| Yes				| Enterprise only |

## Container Backend

|																					|	Docker			|	Original Mesos				|	UCR				|Comments |
| --------------------------------------- | ----------- | --------------------- | --------- | ------- |
| **Overlayfs**                           | Yes         | ??                    | Yes       |         |
| **Aufs**                                | Yes         | ??                    | Yes       |         |
| **Bind**                                | N/A         | ??                    | Yes       |         |

## Storage

|																					|	Docker			|	Original Mesos				|	UCR				|Comments |
| --------------------------------------- | ----------- | --------------------- | --------- | --------- |
| **Local Persistent Volumes**						| Yes					| Yes										| Yes				|						|
| **Host Volumes**												| Yes					| Yes									  | Yes				| CLI only  |
| **External Volumes**                    | Yes         | Yes                   | Yes       |           |

## Service Endpoints

|																					|	Docker			|	Original Mesos				|	UCR				|Comments   |
| --------------------------------------- | ----------- | --------------------- | --------- | --------- |
| **Named Ports**													| Yes					| ??										| Yes				|						|
| **Numbered Ports**											| Yes					| ??										| Yes				|						|

## Networking

|																					|	Docker			|	Original Mesos				|	UCR				|Comments   |
| --------------------------------------- | ----------- | --------------------- | --------- | --------- |
| **Host Networking**       							| Yes					| Yes										| Yes				|						|
| **Bridge Networking**       						| Yes					| No										| No				|						|
| **CNI**         												| N/A					| Yes										| Yes				|						|
| **CNM**??                 							| Yes					| ??										| N/A				| Docker 1.11+ |
| **L4lB**        												| Yes					| ??										| Yes				|	Requires defined service endpoints, TCP health checks do not work with L4LB |

## Private Registry

|																	|	Docker			|	Original Mesos				|	UCR				|
| ------------------------------- | ----------- | --------------------- | --------- |
| **Token-based Container Auth**	| Yes					| No										| No				|
| **Token-based Cluster Auth**		| Yes					| ??										| Yes				|
| **Basic Container Auth**        | Yes         | ??                    | No        |
| **Basic Cluster Auth**          | Yes         | ??                    | Yes       |

## Health Checks

|																					|	Docker			|	Original Mesos				|	UCR				|Comments   |
| --------------------------------------- | ----------- | --------------------- | --------- | --------- |
| **TCP**													        | Yes					| ??										| Yes				|	CLI only	|
| **HTTP/HTTPS**                          | Yes         | ??                    | Yes       | CLI only  |
| **Command**                             | Yes         | ??                    | Yes       |           |
| **Local TCP**                           | Yes         | ??                    | Yes       | CLI only  |
| **Local HTTP/HTTPS**                    | Yes         | ??                    | Yes       |           |
