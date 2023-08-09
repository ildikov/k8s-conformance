# Kubernetes Conformance Tests on OpenNESS

## OpenNESS Setup

The OpenNESS Experience Kit [OEK](https://github.com/open-ness/openness-experience-kits) repository contains a set of Ansible playbooks for the easy setup of OpenNESS.

To be able to reproduce these conformance results, user need to create a 1 Edge Controller (OpenNESS Kubernetes Control Plane node) and 2 Edge Node (OpenNESS Kubernetes node) setup with deployment flavor - [minimal](https://github.com/open-ness/specs/blob/master/doc/flavors.md#minimal-flavor). The detailed deployment instructions are available in [Quickstart Guide](https://github.com/open-ness/specs/blob/master/doc/getting-started/network-edge/controller-edge-node-setup.md#quickstart)

## Preconditions

To use the playbooks, ensure the following preconditions are fulfilled. Detailed steps are available in github [Preconditions](https://github.com/open-ness/specs/blob/master/doc/getting-started/network-edge/controller-edge-node-setup.md#preconditions) section

  - CentOS* 7.6.1810 must be installed on hosts where the product is deployed. It is highly recommended to install the operating system using a minimal ISO image on nodes that will take part in deployment.
  - Hosts for the Edge Controller (Kubernetes control plane) and Edge Nodes (Kubernetes nodes) must have proper and unique hostnames (i.e., not localhost). This hostname must be specified in /etc/hosts.
     ```shell
     127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 <host_name>
     ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 <host_name>
     ```
  - SSH keys must be exchanged between hosts
  
## Running Playbooks

Playbooks can be executed by running helper deployment scripts from the Ansible host or Edge Controller.

1. Pull the OpenNESS Experience Kit repository in the host meant to be configured as Edge Controller.

   ```shell
   mkdir -p ~/conformance/
   git clone https://github.com/open-ness/openness-experience-kits
   cd ~/conformance/openness-experience-kits
   ```

2. Deployment helper scripts require that the Edge Controller and Edge Nodes to be configured in inventory.ini

   ```shell
   vi inventory.ini
   ```

   ```yaml
   [all]
   controller ansible_ssh_user=root ansible_host=<host ip address>
   node01 ansible_ssh_user=root ansible_host=<host ip address>
   node02 ansible_ssh_user=root ansible_host=<host ip address>

   [controller_group]
   controller

   [edgenode_group]
   node01
   node02

   [edgenode_vca_group]

   [ptp_master]
   controller

   [ptp_slave_group]
   node01
   node02
   ```

3. If a proxy is required to connect to the internet, update ./group_vars/all/10-default.yml as follows:
   [Example](https://github.com/open-ness/specs/blob/master/doc/getting-started/network-edge/controller-edge-node-setup.md#setting-proxy)

   - Edit the proxy_ variables in the group_vars/all/10-default.yml file.
   - Set the proxy_enable variable in group_vars/all/10-default.yml file to true.
   - Append the network CIDR (e.g., 192.168.0.1/24) to the proxy_noproxy variable in group_vars/all/10-default.yml

4. OpenNESS provides the possibility to synchronize a machine's time with the NTP server.
   To enable NTP synchronization, change ntp_enable in group_vars/all/10-default.yml:
   
   ```shell
   ntp_enable: true
   ```
   
   Servers to be used instead of default ones can be provided using the ntp_servers variable in group_vars/all/10-default.yml:
   
   ```shell
   ntp_servers: ["ntp.local.server"]
   ```

5. Run the deployment helper script with flavor option as minimal

   ```shell
   ./deploy_ne.sh -f minimal
   ```

   Once the deployment succeeds without an error, proceed to run the K8s conformance test with Sonobuoy.

## Run K8s Conformance Tests with Sonobuoy

The conformance tests are run according to the [instructions](https://github.com/vmware-tanzu/sonobuoy/) available on the github.

1. Download and Install Sonobuoy in host configured as Edge Controller:

   ```shell
   wget https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.18.4/sonobuoy_0.18.4_linux_amd64.tar.gz
   tar -xvzf sonobuoy_0.18.4_linux_amd64.tar.gz
   mv sonobuoy /usr/bin/
   chmod +x /usr/bin/sonobuoy
   ```

2. Run Conformance with Sonobuoy:
   ```shell
   sonobuoy run --mode certified-conformance
   ```

3. Check the test run status:
   ```shell
   sonobuoy status
   ```

4. Once the test status prompts 'Complete', download the sonobuoy results file:
   ```shell
   sonobuoy retrieve
   sonobuoy results <tar.gz file>
   ```
   add e2e.log & junit_01.xml files from ./plugins/e2e/results/global/

5. Cleanup the sonobuoy pods
   ```shell
   sonobuoy delete
   ```
