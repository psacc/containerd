# containerd

containerd is an industry-standard container runtime with an emphasis on simplicity, robustness and portability. It is available as a daemon for Linux and Windows, which can manage the complete container lifecycle of its host system: image transfer and storage, container execution and supervision, low-level storage and network attachments, etc..

containerd is designed to be embedded into a larger system, rather than being used directly by developers or end-users.

### State of the Project

containerd currently has two active branches.
There is a [v0.2.x](https://github.com/docker/containerd/tree/v0.2.x) branch for the current release of containerd that is being consumed by Docker and others and the master branch is the development branch for the 1.0 roadmap and feature set.
Any PR or issue that is intended for the current v0.2.x release should be tagged with the same `v0.2.x` tag.

### Communication

For async communication and long running discussions please use issues and pull requests on the github repo.
This will be the best place to discuss design and implementation.

For sync communication we have a community slack with a #containerd channel that everyone is welcome to join and chat about development.

**Slack:** https://community.docker.com/registrations/groups/4316

## Features

* OCI Image Spec support
* OCI Runtime Spec support
* Image push and pull support
* Container runtime and lifecycle support
* Network primitives for creation, modification, and deletion of interfaces
* Management of network namespaces containers to join existing namespaces
* Multi-tenant supported with CAS storage for global images

## Scope and Principles

Having a clearly defined scope of a project is important for ensuring consistency and focus.
These following criteria will be used when reviewing pull requests, features, and changes for the project before being accepted.

### Components

Components should not have tight dependencies on each other so that they are unable to be used independently.
The APIs for images and containers should be designed in a way that when used together the components have a natural flow but still be useful independently.

An example for this design can be seen with the overlay filesystems and the container execution layer.
The execution layer and overlay filesystems can be used independently but if you were to use both, they share a common `Mount` struct that the filesystems produce and the execution layer consumes.

### Primitives

containerd should expose primitives to solve problems instead of building high level abstractions in the API.
A common example of this is how build would be implemented.
Instead of having a build API in containerd we should expose the lower level primitives that allow things required in build to work.
Breaking up the filesystem APIs to allow snapshots, copy functionality, and mounts allow people implementing build at the higher levels more flexibility.

### Extensibility and Defaults

For the various components in containerd there should be defined extension points where implementations can be swapped for alternatives.
The best example of this is that containerd will use `runc` from OCI as the default runtime in the execution layer but other runtimes conforming to the OCI Runtime specification they can be easily added to containerd.

containerd will come with a default implementation for the various components.
These defaults will be chosen by the maintainers of the project and should not change unless better tech for that component comes out.
Additional implementations will not be accepted into the core repository and should be developed in a separate repository not maintained by the containerd maintainers.

### Releases

containerd will be released with a 1.0 when feature complete and this version will be supported for 1 year with security and bug fixes applied and released.

The upgrade path for containerd is that the 0.0.x patch releases are always backward compatible with its major and minor version.
Minor (0.x.0) version will always be compatible with the previous minor release. i.e. 1.2.0 is backwards compatible with 1.1.0 and 1.1.0 is compatible with 1.0.0.
There is no compatibility guarantees with upgrades from two minor releases. i.e. 1.0.0 to 1.2.0.

There are not backwards compatibility guarantees with upgrades to major versions. i.e 1.0.0 to 2.0.0.
Each major version will be supported for 1 year with bug fixes and security patches.

### Scope

The following table specifies the various components of containerd and general features of container runtimes.
The table specifies whether or not the feature/component is in or out of scope.

| Name | Description | In/Out | Reason |
|------------------------------|--------------------------------------------------------------------------------------------------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| execution | Provide an extensible execution layer for executing a container | in | Create,start, stop pause, resume exec, signal, delete |
| cow filesystem | Built in functionality for overlay, aufs, and other copy on write filesystems for containers | in |  |
| distribution | Having the ability to push and pull images as well as operations on images as a first class API object | in | containerd will fully support the management and retrieval of images |
| low-level networking drivers | Providing network functionality to containers along with configuring their network namespaces | in | Network support will be added via interface and network namespace operations, not service discovery and service abstractions. |
| build | Building images as a first class API | out | Build is a higher level tooling feature and can be implemented in many different ways on top of containerd |
| volumes | Volume management for external data | out | The API supports mounts, binds, etc where all volumes type systems can be built on top of. |
| logging | Persisting container logs | out | Logging can be build on top of containerd because the container’s STDIO will be provided to the clients and they can persist any way they see fit. There is no io copying of container STDIO in containerd. |


containerd is scoped to a single host and makes assumptions based on that fact.
It can be used to build things like a node agent that launches containers but does not have any concepts of a distributed system.

containerd is designed to be embedded into a larger system, hence it only includes a barebone CLI (`ctr`) specifically for development and debugging purpose, with no mandate to be human-friendly, and no guarantee of interface stability over time.

Also things like service discovery are out of scope even though networking is in scope.
containerd should provide the primitives to create, add, remove, or manage network interfaces and network namespaces for a container but IP allocation, discovery, and DNS should be handled at higher layers.

### How is the scope changed?

The scope of this project is a whitelist.
If it's not mentioned as being in scope, it is out of scope.  
For the scope of this project to change it requires a 100% vote from all maintainers of the project.

## Copyright and license

Copyright © 2016 Docker, Inc. All rights reserved, except as follows. Code
is released under the Apache 2.0 license. The README.md file, and files in the
"docs" folder are licensed under the Creative Commons Attribution 4.0
International License under the terms and conditions set forth in the file
"LICENSE.docs". You may obtain a duplicate copy of the same license, titled
CC-BY-SA-4.0, at http://creativecommons.org/licenses/by/4.0/.
