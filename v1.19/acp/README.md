# Alauda Container Platform (ACP)
A Cloud-Native Application Platform based on Kubernetes

## Install ACP

### Download the package

[Contact Alauda](mailto:hello@alauda.io) to download the ACP installation package.

### Run the following command to install the ACP:

```sh
$ tar xzvf -C /opt cpaas-installer.tgz
$ cd /opt/cpaas/installer
$ bash setup.sh
```

## Run Conformance Test

1. On one of the worker node, install sonobuoy:

```sh
$ wget https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.20.0/sonobuoy_0.20.0_linux_amd64.tar.gz
$ tar -xzf sonobuoy_0.20.0_linux_amd64.tar.gz
```

2. Run sonobuoy:

```sh
$ ./sonobuoy run
```

3. Watch logs:

```sh
$ ./sonobuoy logs
```

4. Check status:

```sh
$ ./sonobuoy status
```

5. Retrieve results:

Wait for around 50 minutes for the test to be finished, the status will show `complete` to indicate that the test has been finished, then run the following command to extract the test results.

```sh
$ ./sonobuoy retrieve .
$ mkdir ./results; tar xzf *.tar.gz -C ./results
```
