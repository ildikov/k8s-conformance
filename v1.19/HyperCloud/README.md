# HyperCloud

## Create a Kubernetes Cluster
### Prerequisites
Prepare three nodes with CentOS 7 - one for `master node` and the others for `worker node`
### Setup master node
#### Install Kubernetes, CRI-O, Calico, Kubevirt and HyperCloud
1. Download installer file in our repository and Checkout Git branch
    ```
    git clone https://github.com/tmax-cloud/hypercloud_infra_installer.git
    git checkout v1.19
    ```
2. Modify `k8s.config` to suit your environment
You need to change the value of `apiServer` to the IP address of master node. And set the versions of each components like below.
    ```
    # skip
    apiServer=172.22.5.2
    crioVersion=1.19
    k8sVersion=1.19.4
    kubevirtVersion=0.34.2
    calicoVersion=3.16
    # skip
    ```
1. Execute installer script
    ```
    ./k8s_master_install.sh
    ```
2.  Get the join command for worker node
    ```
    kubeadm token create --print-join-command
    ```
    You can get the result in this format:
    ```
    kubeadm join 192.168.122.50:6443 --token mnp5b8.u7tl2cruk73gh0zh     --discovery-token-ca-cert-hash sha256:662a697f67ecbb8b376898dcd5bf4df806249175ea4a90c2d3014be399c6c18a
    ```
3. If you create a single node Kubernetes cluster, you have to untaint the master node
    ```
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```
4. If the installation process fails, execute uninstaller script then installer script again
    ```
    ./k8s_uninstall.sh
    ./k8s_master_install.sh
    ```
#### Install HyperCloud Storage
- Before installing hypercloud-storage with hcsctl, create yaml files is required for installation and change it to suit your environment.

   ``` shell
   $ hcsctl create-inventory {$inventory_name}
   # Ex) hcsctl create-inventory myInventory
   ```
- Two directories of rook and cdi are created in the created inventory. `./myInventory/rook/*.yaml` are yaml files used for Rook-Ceph installation, and `./myInventory/cdi/*.yaml` are yaml files used for KubeVirt-CDI installation.
- Since all the generated yaml files are for sample provision, you have to use the contents of each yaml file after **modifying to your environment**.<strong> Do not modify the folder and file name. </strong>
- Modify yaml contents created under `./myInventory/rook/` path to fit your environment. Refer to https://rook.github.io/docs/rook/v1.4/ceph-cluster-crd.html
- After modifying the inventory files to suit the environment, install hypercloud-storage with hcsctl.
   ``` shell
   $ hcsctl install {$inventory_name}
   # Ex) hcsctl install myInventory
   ```

    - When installation is completed normally, you can use hypercloud-storage. After installation, you can use Block Storage and Shared Filesystem.
- Verify whether hypercloud-storage is properly installed with rook.test and cdi.test.
    ``` shell
    $ rook.test
    $ cdi.test
    ```
    - To check whether hypercloud-storage can be used normally, various scenario tests are performed.

### Setup worker nodes
#### Install Kubernetes and CRI-O
1. Download installer file in our repository
    ```
    git clone https://github.com/tmax-cloud/hypercloud_infra_installer.git
    ```
2. Execute installer script
    ```
    ./k8s_node_install.sh
    ```
3. Execute the join command
    ```
    kubeadm join 192.168.122.50:6443 --token mnp5b8.u7tl2cruk73gh0zh     --discovery-token-ca-cert-hash sha256:662a697f67ecbb8b376898dcd5bf4df806249175ea4a90c2d3014be399c6c18a
    ```
4. If the installation process fails, execute uninstaller script then installer script again
    ```
    ./k8s_uninstall.sh
    ./k8s_node_install.sh
    ```

## Run Conformance Tests
1. Download Sonobuoy(0.20)
```
wget https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.20.0/sonobuoy_0.20.0_linux_amd64.tar.gz
tar -xzf sonobuoy_0.20.0_linux_amd64.tar.gz
```
2. Run Sonobuoy
```
./sonobuoy run --mode=certified-conformance
```
3. Check the status
```
./sonobuoy status
```
4. Once `sonobuoy status` shows the run as `completed`, download the results tar.gz file
```
./sonobuoy retrieve
```
