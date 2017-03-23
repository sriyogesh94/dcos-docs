---
post_title: Datadog Metrics for DC/OS
feature_maturity: preview
menu_order: 3.5
---

The Datadog metrics plugin for DC/OS supports sending metrics from the DC/OS metrics service on master and agent hosts to a Datadog agent for shipping to DatadogHQ.

# Installation

## Build the plugin

**Prerequisite:** 
- DC/OS is installed
- [Go programming environment](https://golang.org/doc/install) <!-- dcos-metrics must be run from within the go directory -->
- GitHub

1. Clone the DC/OS metrics repository:

   ```bash
   $ git clone git@github.com:dcos/dcos-metrics
   ```
   
1. Navigate to the `dcos-metrics` repository and run the build command:

   ```bash
   $ cd dcos-metrics && make
   ```

   The plugin is available in the build directory:

   ```
   $ tree build
   build
   ├── collector
   │   └── dcos-metrics-collector-1.0.0-rc7
   ├── plugins
   │   └── dcos-metrics-datadog_plugin-1.0.0-rc7
   └── statsd-emitter
       └── dcos-metrics-statsd-emitter-1.0.0-rc7
   ```

## Install the Datadog agent in your DC/OS cluster

Install the `datadog` package in DC/OS:

1.  Go to the **Universe** tab of the DC/OS GUI and find the **Datadog** package. 
    ![datadog package](/docs/1.9/administration/monitoring/metrics/img/datadog-package.png)
1.  Click **INSTALL PACKAGE** -> **ADVANCED INSTALLATION** and enter [your Datadog API_KEY](https://app.datadoghq.com/account/settings#api).
    ![API key](/docs/1.9/administration/monitoring/metrics/img/datadog-api-key.png)
1.  Click **REVIEW AND INSTALL** to complete your installation.

After a minute or two a Datadog agent will be running in the cluster at `datadog-agent.marathon.mesos:8125`. This is the default location used by the Datadog plugin.

## Test the DC/OS Datadog metrics plugin (agents only)
As a stopgap during testing, you may be able to manually run the Datadog plugin on your agents by running it as a Marathon task. You must first upload the binary you built to a web server that's visible to your cluster, then create a Marathon application like the following (with customized `cmd`, `instances`, and `uris` to meet your needs):

```json
{
  "cmd": "chmod +x ./dcos-metrics-* && ./dcos-metrics-* -dcos-role agent -auth-token <CONTENT OF 'dcos config show core.dcos_acs_token'>",
  "instances": NUMBER_OF_AGENTS,
  "uris": [ "https://YOURFILEHOST.COM/dcos-metrics-datadog_plugin-YOURBUILDVERSION" ],
  "id": "test-datadog-plugin",
  "cpus": 0.1,
  "mem": 128,
  "disk": 0,
  "acceptedResourceRoles": [ "slave_public", "*" ]
}
```

## Install the DC/OS Datadog metrics plugin
Once you're happy with the result, you'll need to install the plugin into your cluster. For each host in your cluster, you'll need to transfer the binary you build for the plugin and then add a systemd unit to manage the service. This unit differs slightly between agent and master hosts.

### Create a Valid Auth Token for DC/OS
The DC/OS docs have good info on making this auth token for [OSS](https://dcos.io/docs/1.7/administration/id-and-access-mgt/managing-authentication/) and [enterprise](https://docs.mesosphere.com/1.8/administration/id-and-access-mgt/service-auth/custom-service-auth/) DC/OS.

### Deploy the Metrics Plugin to Every Cluster Host
1. `scp dcos-metrics-datadog-plugin-1.0.0-rc7 my.host:/usr/bin`
1. `ssh my.master "chmod 0755 /usr/bin/dcos-metrics-datadog-plugin-1.0.0-rc7"`

### Master Systemd Unit
Add a master systemd unit file: `cat /etc/systemd/system/dcos-metrics-datadog-plugin.service`

```
[Unit]
Description=DC/OS Datadog Metrics Plugin (master)

[Service]
ExecStart=/usr/bin/dcos-metrics-datadog-plugin-1.0.0rc7 -dcos-role master -metrics-port 80 -auth-token <MY_AUTH_TOKEN>
```

### Agent Systemd Unit
Add a agent systemd unit file: `cat /etc/systemd/system/dcos-metrics-datadog-plugin.service`

```
[Unit]
Description=DC/OS Datadog Metrics Plugin (agent)

[Service]
ExecStart=/usr/bin/dcos-metrics-datadog-plugin-1.0.0rc7 -dcos-role agent -auth-token <MY_AUTH_TOKEN>
```

*Note:* This plugin runs on port :61001 (agent adminrouter) by default, so we don't pass the port as we did in the master version of the service.

### Enable, Start & Verify

1. `systemctl enable dcos-metrics-datadog-plugin && systemctl start dcos-metrics-datadog-plugin`
1. `journalctl -u dcos-metrics-datadog-plugin` -> Make sure everything is OK

## That's it!