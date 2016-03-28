# AMP Mesos DayTrader Demo

These demonstration applications create an autoscaling cluster of DayTrader nodes running on a choice of both Web Application Servers ([IBM WebSphere Liberty](https://developer.ibm.com/wasdev/websphere-liberty/) or [Wildfly 10](http://wildfly.org/)) and Databases ([MariaDB](https://mariadb.com/) or [PostgreSQL](http://www.postgresql.org/)) within an AMP Pro deployed Apache Mesos Cluster running the Marathon Framework.

Load balancing is provided by [Nginx](http://nginx.org/) with the balancer group auto-updating as the AMP Pro autoscaling policy adds and removes DayTrader nodes in response to traffic hitting those nodes.

The DayTrader App and Web Application Servers are deployed via YAML blueprint catalogs containing simple shell scripts.

Both the Apache Mesos Cluster and the DayTrader Cluster(s) deployed to it can be provisioned and deployed to [IBM Blue Box](https://www.blueboxcloud.com/) in a matter of minutes.

## Prerequisites

Before proceeding you will need the following:

- Access to an IBM Blue Box environment - see [here](#ibm-blue-box).


## Quickstart

The Quickstart sections will guide you through the process of deploying AMP Pro, adding a cloud location to its catalog (where you'll be deploying the Apache Mesos Cluster), adding the demo application catalogs and subsequent deployment of a selection of DayTrader Cluster variants to Apache Mesos.

You can use the provided Vagrant or manual process to start your AMP Pro instance.

### Starting AMP Pro with Vagrant

You can use the following steps with Vagrant 1.8.1+ and Virtualbox 5.0.16+. You can examine the installation steps and Vagrant config in the `servers.yaml` file [here](vagrant_config/servers.yaml).

1. Clone and cd into this repository:
    ```
    git clone https://github.com/cloudsoft/amp-daytrader-mesos-demo.git
    cd amp-daytrader-mesos-demo
    ```

2. Copy public and private keys for your Cloud Provider to the `my_keys` directory, these keys will be copied to the `/home/vagrant/.ssh/` directory on startup so as to be available to AMP Pro. For example if you use the `openstack` keypair to connect to your IBM Blue box tenant you should copy them as follows:

    ```
    cp ~/.ssh/openstack.pem ~/.ssh/openstack.pub ./my_keys/
    ```
3. Start the Vagrant box
    ```
    vagrant up
    ```

4. Connect to the AMP Pro web console [http://localhost:8081](http://localhost:8081).

5. Add your Blue Box OpenStack location to the catalog - see [here](#ibm-blue-box). This is where we'll be deploying the Apache Mesos Cluster.

### Starting AMP Pro Manually

1. Download and extract the current AMP Pro release archive and change to the extracted directory:
    ```
    curl -O http://developers.cloudsoftcorp.com/amp-pro/amp-pro-dist-3.1.0-20160327.1622-dist.tar.gz
    tar zxf amp-pro-dist-3.1.0-20160327.1622-dist.tar.gz
    cd cloudsoft-amp-pro-3.1.0-20160327.1622
    ```

2. Launch AMP Pro passing the Mesos Cluster, and demo WebSphere Liberty, Wildfly, and Daytrader catalog definitions on startup (this is a very long command as we're pulling in multiple catalogs in a comma seperated list):

    ```
    ./bin/amp launch --persist=auto  --catalogAdd https://raw.githubusercontent.com/cloudsoft/amp-mesos/master/mesos.bom,https://raw.githubusercontent.com/cloudsoft/amp-daytrader-mesos-demo/master/app-servers/websphere-liberty/websphere-liberty.bom,https://raw.githubusercontent.com/cloudsoft/amp-daytrader-mesos-demo/master/app-servers/wildfly-10/wildfly10.bom,https://raw.githubusercontent.com/cloudsoft/amp-daytrader-mesos-demo/master/daytrader-application/daytrader-websphereliberty-cluster.bom,https://raw.githubusercontent.com/cloudsoft/amp-daytrader-mesos-demo/master/daytrader-application/daytrader-wildfly-cluster.bom
    ```

4. Connect to the AMP Pro web console [http://localhost:8081](http://localhost:8081).

5. Add your Blue Box OpenStack location to the catalog - see [here](#ibm-blue-box). This is where we'll be deploying the Apache Mesos Cluster.

## Start Apache Mesos Cluster

1. From the AMP Pro web console Home tab click '*add application*', select '*Apache Mesos*' and hit next.

2. Supply values for the following fields, leaving the rest with defaults:

   1. Set Locations to your added location, for example `ibm-bluebox-mesos`.

   2. Give the Apache Mesos Cluster a *Name:*, for example `Demo Apache Mesos Cluster`.

   3. Point `mesos.slave.privateKey` at the location of your locations private key, for example `~/.ssh/mesos.pem`.

3. Hit deploy, your Apache Mesos cluster will be ready in a few minutes.

   Switching to the Applications tab and expanding the tree view you can find the Mesos Url on the Apache Mesos Cluster Summary page, and Marathon Url under Mesos Frameworks on the Marathon Framework Summary page.

## Start a DayTrader Cluster with IBM WebSphere Liberty and MariaDB

1. From the AMP Pro web console Home tab click '*add application*', select '*DayTrader App Cluster (**WebSphere Liberty/MariaDb**)*' and hit next'.

2. Supply values for the following fields, leaving the rest with defaults:

   1. Set Locations to `my-mesos-cluster`.

   2. Give the DayTrader Cluster a *Name:*, for example `DayTrader WSL MariaDB`.

   3. Provide a WebSphere Liberty Archive URL for the `webProfile7` release archive (`wlp-webProfile7-8.5.5.x.zip`), either hosting it yourself or via an IBM provided source (see [here](https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/wasdev/downloads/wlp/index.yml) for download metadata for recent releases).

4. Hit deploy, your DayTrader Cluster will be ready in a few minutes.

   While the deployment to Apache Mesos is proceeding you can navigate to the Marathon Dashboard where you can observe AMP Pro adding application instances.

5. When the DayTrader Cluster finishes deploying (the green icon will stop flashing) you can connect to Daytrader via the mapped load balancer URI (main.uri) found on the applications Summary Page.


## Demonstrating Autoscaling

1. Follow the instructions on the IBM Blog post '[Measuring performance with the Daytrader 3 benchmark sample](https://developer.ibm.com/wasdev/docs/measuring-performance-daytrader-3-benchmark-sample/)' to download the `DayTrader3Install.zip` via the WebSphere Performance Page. This archive contains a sample Daytrader JMeter test suite file `daytrader3.jmx` which can be used to drive load through the DayTrader application.

2. Populate the DayTrader Database before running JMeter load

    1. Open the DayTrader webpage, from the `main.uri` on the '*DayTrader WSL MariaDB*' Summary page.
    2. Navigate to the Configuration tab and click the link **(Re)-populate DayTrader Database**

       This will open a new, initially blank, page which will periodically update as sample stock and user data is loaded. This can take some time to complete, and updates only infrequently. You will know it is complete when the plain update text is followed by what looks like a normal configuration page.

3. Start the JMeter GUI supplying the `HOST` and `PORT` values for your deployed DayTrader cluster (from the `main.uri` on the '*DayTrader WSL MariaDB*' Summary page).
    ```
    ./apache-jmeter-2.13/bin/jmeter.sh -t ./daytrader3.jmx -JHOST=<MAPPED BALANCER HOST> -JPORT=<MAPPED BALANCER PORT> -JDURATION=600
    ```

4. On opening JMeter you should be presented with the **Thread Group** page, you should alter the `Number of Threads (users)` setting as desired. For demonstration purposes it is recommended to lower this to `10` results in approximately 60 requests/second from an i7 mbp. The default autoscaling thresholds for both Wildfly and WebSphere Liberty have been set to upper `40` and lower `20`.

5. Right click and enable **Aggregate Report** in the left hand tree, and when ready to start traffic click the **green play** menubar item. You should immediately start to see results fill the report pane. The reqs/sec rate should approximately match that show in the Dynamic Cluster Groups `reqs.per_sec.per_node` sensor.

   Expanding the '*DynamicCluster of WebSphere Liberty Servers*' entry on the '*Applications*' tab you should observe AMP Pro provision new DayTrader nodes which are automatically added to the NGinx balancer group, examining the `reqs.per_sec` sensors for each node and for the Dynamic Cluster Group you can observe the traffic being distributed evenly.

6. To stop traffic click the **red stop** menubar item. You can also click the sweeping brush icons (two brushes to the left of the binoculars), to clear the results window.

   You should observe AMP Pro scale the DayTrader nodes back down to the original single node.

## Further Activities

The following additional activities are left to the reader to explore.

### Daytrader with WebSphere and PostgreSQL

You can deploy an additional DayTrader cluster using WebSphere and PostgreSQL by repeating the [Start a DayTrader Cluster](#start-a-daytrader-cluster-with-ibm-websphere-liberty-and-mariadb) section from step 2 onwards and choosing the  '*DayTrader App Cluster (**WebSphere Liberty/PostgreSQL**)*' application.

### Daytrader with Wildfly 10

Blueprint catalog files DayTrader running on Wildfly10 with both MariaDB and PostgreSQL have also been added to AMP, repeat the [Start a DayTrader Cluster](#start-a-daytrader-cluster-with-ibm-websphere-liberty-and-mariadb) section from step 1 onwards chosing the Wildfly apps when adding Appications to the Apache Mesos Cluster.


## Cloud Provider Configuration

### IBM Blue Box
This demo used the following very basic public configuration on IBM Blue Box to provision Ubuntu 14.04 VMs with floating IPs assigned, additional configurations (private, mixed etc) are possibly but not documented here.

The following sections give a very high level overview of the OpenStack configuration used and instructions on adding your own credentials and settings to the AMP node as a location using a provided location template.

#### OpenStack Configuration Overview

- **Image**

  The demo uses Ubuntu 14.04 LTS (Trusty Tahr).

- **Key Pair**

  A demo specific keypair - e.g. `mesos.pem`, and `mesos.pub` stored in the ~/.ssh directory of the user running AMP.

  You may use an existing keypair if desired.

  This keypair must be [accessible in your Blue Box cloud](http://docs.openstack.org/user-guide/cli_nova_configure_access_security_for_instances.html) - it is used to ssh to newly provisioned VMs. We will configure AMP to continue using this `mesos.pem` with the default `ubuntu` user on the Ubuntu 14.04 instances.

- **Security Group**

  An open Mesos security group `mesos` with the following settings was used for the demo:

| Direction | Ether Type | IP Protocol | Port Range    | Remote IP Prefix | Remote Security Group |
| --------- | ---------- | ----------- | ------------- | ---------------- |:---------------------:|
| Egress    | IPv4       | Any         | Any           | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | ICMP        | Any           | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 22            | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 80            | 0.0.0.0/0        | -                     |
| Egress    | IPv4       | TCP         | 2181          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 2181          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 5000          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 5050          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 5051          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 7077          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 8080          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 8081          | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | TCP         | 31000 - 32000 | 0.0.0.0/0        | -                     |
| Egress    | IPv4       | UDP         | 53            | 0.0.0.0/0        | -                     |
| Ingress   | IPv4       | UDP         | 53            | 0.0.0.0/0        | -                     |

- **Network**

  A subnet with DHCP and external routing was used for the demo, you will need to supply the Network ID when creating the location in AMP.

- **Floating IP**

  The demo assigned public IPs to the cluster nodes, you will need to supply the Floating IP pool name when creating the location in AMP.

#### Adding Location to AMP

1. Create a local copy of the templated location catalog yaml file `sample-bluebox.bom` [here](locations/sample-bluebox.bom).

2. Update the file replacing the `<REPLACE_THIS>` tokens with appropriate values from your Blue Box OpenStack environment.

3. From the [AMP Pro web console](http://localhost:8081) Catalog tab, add the updated `sample-bluebox.bom` to the catalog by pasting the contents into the blueprint composer and clicking `Add to Catalog`.

   The location `ibm-bluebox-mesos` will now be available in AMP and should be used as the target location for your Mesos Cluster.

## License

amp-daytrader-mesos-demo is built on [Cloudsoft AMP](http://www.cloudsoft.io) and [Apache Brooklyn](http://brooklyn.apache.org)
and is copyright &copy; 2016 by Cloudsoft Corporation.

This software is released under the terms of [the Cloudsoft EULA](LICENSE.txt).
