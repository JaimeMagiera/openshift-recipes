# Installing OKD 4 on vSphere with User Provisioned Infrastructure

This receipe walks you through the process of installing OKD 4 on vSphere with User Provisioned Infrastructure with the help of scripts from the [OCT](https://github.com/JaimeMagiera/oct) repository. For a more detailed understanding of the OKD installation process, please see the [OKD documentation](https://docs.okd.io/latest/installing/installing_vsphere/installing-vsphere.html).

## Prerequisites:
1. An account on vSphere with read/write access rights on datastores and vm networking
1. A deployment machine connected to the WAN. This machine will run the scripts, connecting to vSphere. Be sure to install the screen or tmux terminal multiplexers, as the OpenShift installation process can take some time.
1. Functioning, configurable DNS service
1. Functioning, configurable DHCP service

## Installation Steps

1. [Install govc](https://github.com/vmware/govmomi/tree/master/govc) on your deployment controller machine. 
1. Clone the [OCT](https://github.com/JaimeMagiera/oct) repository to your deployment machine.
1. Run the environment configuration script, which downloads the OpenShift installer, the oc and kubectl command line tools, and the appropriate Fedora Core OS .ova. 
``` console
configureOKDInstallerEnvironment.sh $(pwd)
```
1. Generate an SSH key which will be used for the communication between the installer and the nodes.
``` console
ssh-keygen -t rsa -b 4096 -N '' \
    -f <path>/<file_name>
```
1. uploadImageToVSphere.sh -f fedora-coreos-32.20200907.3.0-vmware.x86_64.ova -d "VMStor2-01" -n fedora-coreos-32.20200907.3.0-vmware.x86_64
1. eval "$(ssh-agent -s)"; ssh-add ~/.ssh/coreadmin_rsa
1. vi install-config.yml add pullSecret
1. prerun.sh
1. buildSpiritus.sh
1. setMAC-Spiritus.sh
1. manageClusterPower.sh -c spiritus 
1. Launch a sub-terminal with tmux or screen.
``` console
tmux  new -s okd
```
1. Run the OpenShift installer, pointing to your current directory for the configuration. 
``` console
bin/openshift-install --dir=$(pwd) wait-for bootstrap-complete  --log-level=info
```
1. oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}} {{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve

