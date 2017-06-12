---
post_title: Enabling GPU Support
feature_maturity: preview
menu_order: 110
---

DC/OS supports allocating GPUs (Graphics Processing Units) to your long-running DC/OS services. Adding GPUs to your service can dramatically accelerate big data workloads. 

# Enabling GPUs
GPUs must be enabled during DC/OS installation. Follow the instructions to enable GPUs based on your DC/OS deployment method.

## Enabling GPUs for a Custom DC/OS Installation

1.  Install DC/OS using the [custom installation instructions](/docs/1.10/installing/custom/advanced/) with the `enable-gpu-isolation: 'true'` configuration parameter specified.
1.  Install the [NVIDIA Management Library (NVML)](https://developer.nvidia.com/nvidia-management-library-nvml) on each node of your cluster that has GPUs, unless it is already installed. Find detailed installation instructions [here](https://github.com/apache/mesos/blob/master/docs/gpu-support.md#external-dependencies).

## Enabling GPUs for an AWS EC2 DC/OS Installation

###  Prerequisites
- The AWS DC/OS advanced template [system requirements](/docs/1.10/installing/cloud/aws/advanced/).
- The `zen.sh` script copied to your local machine. The script and instructions are [here](/docs/1.10/installing/cloud/aws/advanced/).

### Create Dependencies

1. Run the `zen.sh` script to create the Zen template dependencies. These dependencies will be used as input to create your stack in CloudFormation.
  ```
  bash ./zen.sh <stack-name>
  ```
  **Important:** You must run the `zen.sh` script before performing the next steps.

1. Follow the instructions [here](/docs/1.10/installing/cloud/aws/advanced/) to create a cluster with advanced AWS templates, using the following GPU-specific configuration.

1. On the **Create Stack** > **Specify Details** page, specify your stack information and click **Next**. Here are the GPU-specific settings.
  - **CustomAMI** - Specify the custom AMI for your region:

      - us-west-2: `ami-9b5d97fb`
      - us-east-1: `ami-e10e50f6`
      - ap-southeast-2: `ami-37b28f54`
  - **MasterInstanceType** - Accept the default master instance type (e.g. `m3.xlarge`).
  - **PrivateAgentInstanceType** - Specify a machine type of `g2.2xlarge`. This is the GPU-supported machine type.
  - **PublicAgentInstanceType** - Specify a machine type of `g2.2xlarge`. This is the GPU-supported machine type.

1. On the **Options** page, accept the defaults and click Next.

    **Tip**: You can choose whether to rollback on failure. By default this option is set to **Yes**.

1. On the **Review** page, check the acknowledgement box, then click **Create**.

    **Tip**: If the **Create New Stack** page is shown, either AWS is still processing your request or you’re looking at a different region. Navigate to the correct region and refresh the page to see your stack.

# Usage Examples

You can specify GPUs in your app definition with the `gpus` parameter.

## Simple GPU Application Definition
In this example, a simple sleep app is defined which uses GPUs.

1.  Create an an app definition named `simple-gpu-test.json`.

    ```json
    {
         "id": "simple-gpu-test",
         "acceptedResourceRoles":["slave_public", "*"],
         "cmd": "while [ true ] ; do nvidia-smi; sleep 5; done",
         "cpus": 1,
         "mem": 128,
         "disk": 0,
         "gpus": 1,
         "instances": 1
    }
    ```
  
1.  Launch your app using the DC/OS CLI:

    ```bash
    dcos marathon app add simple-gpu-test.json
    ```

    After your service has deployed, check the contents of `stdout` to verify that the service is producing the proper output from the `nvidia-smi` command. You should see something like the following, repeated once every 5 seconds. Access the log [via the DC/OS CLI](/docs/1.10/monitoring/logging/quickstart/) or from the **Health** page for your service on the DC/OS dashboard.
    
    ```bash
    +------------------------------------------------------+
    | NVIDIA-SMI 352.79     Driver Version: 352.79         |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla M60           Off  | 0000:04:00.0     Off |                    0 |
    | N/A   34C    P0    39W / 150W |     34MiB /  7679MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    ```

    You will also see an entry for **GPU** in the DC/OS GUI on the **Configuration** tab for your service.

## Docker-Based Application Definition
In this example, an app is deployed with GPUs that specifies a Docker container and [DC/OS Universal Container Runtime (UCR)](/docs/1.10/deploying-services/containerizers/).  You must use UCR to run a containerized application that uses GPUs. To use the Universal Container Runtime, set the container type to `MESOS`.

1.  Create an app definition named `docker-gpu-test.json`.

    ```json
    {
        "id": "docker-gpu-test",
        "acceptedResourceRoles":["slave_public", "*"],
        "cmd": "while [ true ] ; do nvidia-smi; sleep 5; done",
        "cpus": 1,
        "mem": 128,
        "disk": 0,
        "gpus": 1,
        "instances": 1,
        "container": {
          "type": "MESOS",
          "docker": {
            "image": "nvidia/cuda"
          }
        }
    }
    ```
    
1.  Launch your app using the DC/OS CLI:

    ```bash
    dcos marathon app add docker-gpu-test.json
    ```  

    After your service has deployed, check the contents of `stdout` to verify that the service is producing the proper output from the `nvidia-smi` command. You should see something like the following, repeated once every 5 seconds. Access the log [via the DC/OS CLI](/docs/1.10/monitoring/logging/quickstart/) or from the **Health** page for your service on the DC/OS dashboard.

    ```
    +------------------------------------------------------+
    | NVIDIA-SMI 352.79     Driver Version: 352.79         |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla M60           Off  | 0000:04:00.0     Off |                    0 |
    | N/A   34C    P0    39W / 150W |     34MiB /  7679MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    ```
    
    You will also see an entry for **GPU** on the **Configuration** tab of the page for your service.

# Limitations

- Unlike other resources, like  CPUs, memory, and disk, you can only specify whole numbers of GPUs in your application definition. If a fractional amount is selected, launching the task will result in a `TASK_ERROR`.
- NVIDIA GPU support is only available for tasks launched through the [DC/OS Universal container runtime](/docs/1.10/deploying-services/containerizers/). No support exists for launching GPU-capable tasks through the Docker containerizer. However, the DC/OS Universal container runtime supports running Docker images natively.
-  While GPU resources are advertised to the Mesos master alongside other resources like CPUs, memory, and disk, the master will only forward offers that contain GPUs to DC/OS services that have explicitly enabled the GPU_RESOURCES capability. Below is an example of setting this capability in a C++-based service.
    
   ```
   FrameworkInfo framework;
   framework.add_capabilities()->set_type(
       FrameworkInfo::Capability::GPU_RESOURCES);
    
   GpuScheduler scheduler;
    
   driver = new MesosSchedulerDriver(
     &scheduler,
     framework,
     127.0.0.1:5050);
    
    driver->run();
   ```

## Learn More about GPUs

- [What is GPU Computing?](http://www.nvidia.com/object/what-is-gpu-computing.html)
- [Mesos NVIDIA GPU Support](https://github.com/apache/mesos/blob/master/docs/gpu-support.md).
- [Tutorial: Deep learning with TensorFlow, Nvidia and Apache Mesos (DC/OS)](https://dcos.io/blog/2017/tutorial-deep-learning-with-tensorflow-nvidia-and-apache-mesos-dc-os-part-1/index.html)
- Presentation: [Supporting GPUs in Docker Containers on Apache Mesos](https://docs.google.com/presentation/d/1FnuEW2ic5d-cpSyVOUMfUSM7WxJlZtTAAWt2dZXJ52A/edit#slide=id.p).
- Presentation: [GPU Support in Apache Mesos](https://www.youtube.com/watch?v=giJ4GXFoeuA).
- Presentation: [Adding GPU Support to Mesos](https://docs.google.com/presentation/d/1Y1IUlWV6g1HzD1wYIYXy6AmbfnczWfjvvmqqpeDFBic/edit#slide=id.p).