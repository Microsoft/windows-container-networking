# Windows Container Networking CNI
[![Go Report Card](https://goreportcard.com/badge/github.com/Microsoft/windows-container-networking)](https://goreportcard.com/report/github.com/Microsoft/windows-container-networking)

## Overview
This repo contains plugins meant for testing/development of latest windows features. Their primary use case right now is with a CRI and containerd

## CNI Plugins Available
* `sdnoverlay`
* `nat`
* `sdnbridge`

## Usage
[Container networking on windows.](https://docs.microsoft.com/en-us/virtualization/windowscontainers/container-networking/architecture) Following windows networking definitions, this plugin works to create the endpoint from parameters defined in the cni.conf, and a attach it to the namespace referred to by CNI_NETNS. There are two assumptions the plugins makes to perform this:
1. The user has already created a network which endpoints can be created under.
2. The namespace has been already been created.
For 2, the container is typically created with a reference to the namespace and it is expected that the container runtime performs this creation step. These plugins strictly work within the V2 windows container networking flow. Endpoint Policies should be in the [V2 schema format](https://docs.microsoft.com/en-us/windows-server/networking/technologies/hcn/hcn-top).


## Releases
Currently you must build the binaries yourself (see below)

* ToDo: Automated Release

## Build
These plugins are made for windows and need to be compiled for windows. However, you can cross-compile them from Linux.

If you have make installed on your system:

`make all` - will build all plugins: `nat.exe`, `sdnbridge.exe`, and `sdnoverlay.exe`
`make <plugin>` - will build an individual plugin

Else:

`GOOS=windows GOARCH=amd64 go build -v -o out/<plugin>.exe plugins/<plugin>/*.go`

### Building inside a Linux container

On a Linux machine, run `make dev`, then `make all`. That will cross-build the Windows binaries in a clean environment.

## Testing
There is a test suite that should be run (`make test`) before any changes. Opening a PR should trigger a Jenkins run that will run all the tests. If you wish to run them locally, you'll need a nanoserver image pulled from docker. 

Currently there are two groups of end-to-end tests shared by all the plugins

* Properties Verification - tests an add command and verifies that the resulting state is as expected. I.e. we attach an endpoint that endpoint has policy x,y,z etc. 
* Connectivity Testing -  creates a container and makes a CNI call for it and verifies that the container has connectivity for pod-to-pod, host-to-pod, pod-to-host, and pod-to-internet

## Errors
66 - General failure in the plugin: indicates an error occured in the plugin with no extra information
71 - Port Already Exists: indicates the port specified in a policy, (usually port mapping), is in use, likely by another endpoint. To fix this error, free the port desired or change the port you wish to use.
81 - Network Not Found: The network specified in the CNI config was not found. The expectation is that the network has been created before invoking the plugin to create an endpoint for a container. Create the network to use the plugin.


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

## Code of Conduct
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
