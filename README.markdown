

<div align="center">

<a href="https://github.com/LABORA-INF-UFG/my5Gcore"><img width="40%" src="docs/media/img/my5g-logo.png" alt="free5GC"/></a>

![GitHub](https://img.shields.io/github/license/LABORA-INF-UFG/my5GCore?color=blue) ![GitHub release (latest by date including pre-releases)](https://img.shields.io/github/v/release/LABORA-INF-UFG/my5GCore?include_prereleases) ![GitHub All Releases](https://img.shields.io/github/downloads/LABORA-INF-UFG/my5GCore/total) ![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/LABORA-INF-UFG/my5GCore) ![GitHub commit activity](https://img.shields.io/github/commit-activity/y/LABORA-INF-UFG/my5GCore) 
![GitHub repo size](https://img.shields.io/github/repo-size/LABORA-INF-UFG/my5GCore) ![GitHub Workflow Status](https://img.shields.io/github/workflow/status/labora-inf-ufg/my5gcore/My5Gcore%20Workflow) ![GitHub last commit](https://img.shields.io/github/last-commit/LABORA-INF-UFG/my5GCore) ![GitHub contributors](https://img.shields.io/github/contributors/LABORA-INF-UFG/my5GCore)

---

# my5G-core

</div>

**Currently, my5G-core is a fork of the free5GC, with some extensions to facilitate the deployment.**

---

# my5G-core Setup

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Hardware Tested](#hardware-tested)
- [Recommended Environment](#recommended-environment)
- [Installation](#installation)
  - [A. Pre-requisite](#a-pre-requisite)
  - [B. Install Control Plane Entities](#b-install-control-plane-entities)
  - [C. Install User Plane Function (UPF)](#c-install-user-plane-function-upf)
- [Run](#run)
  - [A. Run the Core Network](#a-run-the-core-network)
  - [B. Run the N3IWF (Individually)](#b-run-the-n3iwf-individually)
  - [C. Run all-in-one with outside RAN](#c-run-all-in-one-with-outside-ran)
  - [D. Deploy within containers](#d-deploy-within-containers)
- [Test](#test)
- [More information](#more-information)
- [Release Note](#release-note)
<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Hardware Tested
* There are no gNB and UE for standalone 5GC available in the market yet.
<!-- 
## Questions
For questions and support please use the [official forum](https://forum.free5gc.org). The issue list of this repo is exclusively
for bug reports and feature requests. -->

## Recommended Environment
- Software
    - OS: Ubuntu 18.04
    - gcc 7.3.0
    - Go 1.14.4 linux/amd64
    - kernel version 5.0.0-23-generic or higher (for UPF)

        Notes:  
            - Also tested on Ubuntu 20.04 with  5.4.0-52-generic kernel version.  
            - You can use `go version` to check your current Go version.
- Hardware
    - CPU: Intel i5 processor
    - RAM: 4GB
    - Hard drive: 160G
    - NIC card: 1Gbps ethernet card

- Hardware recommended
    - CPU: Intel i7 processor
    - RAM: 8GB
    - Hard drive: 160G
    - NIC card: 10Gbps ethernet card

## Installation
### A. Pre-requisite

0. Required kernel version `5.0.0-23-generic`. This request is from the module **gtp5g** (will be installed next) . Any more details please check [here](https://github.com/PrinzOwO/gtp5g)
   ```bash
   # Check kernel version
   $ uname -r
   5.0.0-23-generic
   ```
 1. General required packages 
    ```bash
    sudo apt -y update
    sudo apt -y install mongodb wget git net-tools
    sudo systemctl status mongodb
    # if mongodb is not active
    sudo systemctl start mongodb
    ```

2. Require go language

    If another version of Go is installed

    ```bash
    # Please remove the previous Go version
    sudo rm -rf /usr/local/go
    ```
    ```bash
    # Install Go 1.14.4
    wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
    sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
    ```

    Clean installation
    ```bash 
    # Install Go 1.14.4
    wget https://dl.google.com/go/go1.14.4.linux-amd64.tar.gz
    sudo tar -C /usr/local -zxvf go1.14.4.linux-amd64.tar.gz
    mkdir -p ~/go/{bin,pkg,src}
    echo 'export GOPATH=$HOME/go' >> ~/.bashrc
    echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
    echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
    source ~/.bashrc
    ```

3. Required packages for control and user planes
    ```bash
    sudo apt -y update
    sudo apt -y install gcc cmake autoconf build-essential
    sudo apt -y install libtool pkg-config libmnl-dev libyaml-dev 
    go get -u github.com/sirupsen/logrus
    go get -u github.com/calee0219/fatal
    ```
4. Installing kernel module

    Please check Linux kernel version if it is `5.0.0-23-generic` or higher
    ```bash
    uname -r
    ```

    Get Linux kernel module 5G GTP-U
    ```bash
    cd ~
    git clone -b v0.1.0 https://github.com/PrinzOwO/gtp5g.git
    ```
    Install Linux kernel module 5G GTP-U
    ```bash
    cd ~/gtp5g
    make
    sudo make install
    ```
5. Network Setting
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o <dn_interface> -j MASQUERADE
    sudo systemctl stop ufw
    ```

### B. Install my5G-core entities
    
1. Clone my5GCore project
    ```bash
    cd ~
    git clone https://github.com/LABORA-INF-UFG/my5Gcore.git
    cd ~/my5Gcore
    git checkout master
    git submodule sync
    git submodule update --init --jobs `nproc`
    git submodule foreach git checkout master
    git submodule foreach git pull --jobs `nproc`
    ```
    **NOTE: the root folder name for this repository must be `~/my5Gcore`.  If it is changed, compilation will fail.**

2. Run the script to install dependent packages
    ```bash
    cd ~/my5Gcore
    go mod download
    ```
3. Compile network function services in `my5gCore`
    ```bash
    cd ~/my5Gcore
    make all 
    ```
    Customize the NFs as desired. The NFs configuration file is:

    + For UPF  
    ```~/my5Gcore/src/upf/build/config/upfcfg.yaml```
    + For other NFs   
    ```~/my5Gcore/config```

## Run

### A. Run the Core Network 

1. Run network function services individually.  
``` ./bin/<some-NF> [-free5gccfg <core-configuration-file>] [-udmcfg <nf-configuration-file>] & ```

    For example, to run the AMF:

    ```bash
    cd ~/my5Gcore
    ./bin/amf
    ```
    to run with customized settings:
    ```bash
    # AMF
    ./bin/amf -free5gccfg sample/my5g_basic_config/free5GC.conf -amfcfg sample/my5g_basic_config/amfcfg.conf &
    ```
    **Note: The N3IWF needs specific configuration, which is detailed in section B.** 
2. Run whole core network
    ```bash
    # bash
    cd ~/my5Gcore
    ./run.sh
    ```
### B. Run the N3IWF (Individually)

<!-- + To run an instance of the N3IWF, make sure your system is equipped with three network interfaces: the first connects to the AMF, the second connects to the UPF, and the third is for IKE daemon.

    Configure each interface with a suitable IP address.

    Create an interface for IPSec traffic:

    ```bash
    # replace <...> to suitable value
    sudo ip link add ipsec0 type vti local <IKEBindAddress> remote 0.0.0.0 key <IPSecInterfaceMark>
    ```

    Assign an address to this interface, then bring it up:

    ```bash
    # replace <...> to suitable value
    sudo ip address add <IPSecInterfaceAddress/CIDRPrefix> dev ipsec0
    sudo ip link set dev ipsec0 up
    ```

    Run the N3IWF (root privilege is required):

    ```bash
    cd ~/free5gc/
    sudo ./bin/n3iwf
    ``` -->

### C. Run all-in-one with external RAN

<!-- + Refer to this [sample config](./sample/ran_attach_config) if you wish to connect an external RAN with a complete free5GC core network. -->

### D. Deploy within containers

<!-- + [free5gc-compose](https://github.com/free5gc/free5gc-compose/) provides a sample for the deployment of elements within containers. -->

## Test

Start a Wireshark capture on any core-connected interface, applying the filter `'pfcp||icmp||gtp'`.

In order to run the tests, first do this:

```bash
cd ~/free5gc
chmod +x ./test.sh
 ```

The tests are all run from within `~/free5gc`.

a. TestRegistration

```bash
./test.sh TestRegistration
```

b. TestGUTIRegistration

```bash
./test.sh TestGUTIRegistration
```

c. TestServiceRequest

```bash
./test.sh TestServiceRequest
```

d. TestXnHandover

```bash
./test.sh TestXnHandover
```

e. TestDeregistration

```bash
./test.sh TestDeregistration
```

f. TestPDUSessionReleaseRequest

```bash
./test.sh TestPDUSessionReleaseRequest
```

g. TestPaging

```bash
./test.sh TestPaging
```

h. TestN2Handover

```bash
./test.sh TestN2Handover
```

i. TestNon3GPP

```bash
./test.sh TestNon3GPP
```

j. TestReSynchronisation

```bash
./test.sh TestReSynchronisation
```

k. TestULCL

```bash
./test_ulcl.sh -om 3 TestRegistration
```

## More information

For more details, reference the free5gc [wiki](https://github.com/free5gc/free5gc/wiki).

## Release Note

Detailed changes for each release are documented in the [release notes](https://github.com/free5gc/free5gc/releases).