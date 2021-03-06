Scenario Test
========================

This page explains how to set up a scenario test environment and run the test.

## Prerequisites

Assume you finished setting up [Golang](https://golang.org/doc/install) and [Docker](https://docs.docker.com/installation/ubuntulinux/) on Ubuntu 14.04 Server VM.
We recommend allocating memory more than 8GB to the VM.
Because this scenario test runs a lot of test cases concurrently.

## <a name="section0"> Check
Please check if Golang and Docker is installed correctly and
make sure the $GOPATH is defined.

```shell
$ go version
go version go1.5.1 linux/amd64

$ echo $GOPATH
/home/yokoi-h/work

$ sudo docker version
Client:
 Version:      1.9.0
 API version:  1.21
 Go version:   go1.4.2
 Git commit:   76d6bc9
 Built:        Tue Nov  3 17:43:42 UTC 2015
 OS/Arch:      linux/amd64

Server:
 Version:      1.9.0
 API version:  1.21
 Go version:   go1.4.2
 Git commit:   76d6bc9
 Built:        Tue Nov  3 17:43:42 UTC 2015
 OS/Arch:      linux/amd64

```

## <a name="section1"> Set up dependencies
Execute the following commands inside the VM to install the dependencies:

1. Install pip and [pipework](https://github.com/jpetazzo/pipework).

 ```shell
 $ sudo apt-get update
 $ sudo apt-get install git python-pip python-dev iputils-arping bridge-utils lv
 $ sudo wget https://raw.github.com/jpetazzo/pipework/master/pipework -O /usr/local/bin/pipework
 $ sudo chmod 755 /usr/local/bin/pipework
 ```
 <br>

1. Get docker images.
 Download docker images pertaining to GoBGP testing.

 ```shell
 $ sudo docker pull golang:1.5
 $ sudo docker pull osrg/quagga
 $ sudo docker pull osrg/gobgp
 ```
 <br>

1. Clone gobgp and install python libraries.

 ```shell
 $ mkdir -p $GOPATH/src/github.com/osrg
 $ cd $GOPATH/src/github.com/osrg
 $ git clone https://github.com/osrg/gobgp.git
 $ cd ./gobgp/test
 $ sudo pip install -r pip-requires.txt
 ```
<br>

## <a name="section2"> Install local source code
You need to install local source code into gobgp docker container.
You also need this operation at every modification to the source code.

```
$ cd $GOPATH/src/github.com/osrg/gobgp
$ sudo fab -f ./test/lib/base.py make_gobgp_ctn --set tag=gobgp
```


## <a name="section3"> Run test

1. Run all test

 You can run all scenario tests with run_all_tests.sh.
 If all tests passed, you can see "all tests passed successfully" at the end of the test.

 ```shell
 $ cd $GOPATH/src/github.com/osrg/gobgp/test/scenario_test
 $ ./run_all_tests.sh
 ...
 OK
 all tests passed successfully
 ```
 <br>
2. Run each test

 Gobgp have a scenario test shown in the following.
 You can run scenario tests individually with each test file.

 ```shell
 $ cd $GOPATH/src/github.com/osrg/gobgp/test/scenario_test
 $ sudo -E PYTHONPATH=$GOBGP/test python <scenario test name>.py
 ...
 OK
 ```

 what kind of scenaio tests:
 - bgp_router_test
 - bgp_zebra_test
 - evpn_test
 - flow_spec_test
 - global_policy_test
 - ibgp_router_test
 - route_reflector_test
 - route_server_ipv4_v6_test
 - route_server_malformed_test
 - route_server_policy_grpc_test
 - route_server_policy_test
 - route_server_test

## <a name="section4"> Clean up
A lot of containers are created during the test.
Let's clean up.
```
$ sudo docker rm -f $(sudo docker ps -a -q)
```
