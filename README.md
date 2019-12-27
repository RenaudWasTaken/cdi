# CDI - The Container Device Interface
## What is CDI?
CDI (Container Device Interface), is a specification, for container runtimes, to support third party devices. 

CDI concerns itself only with enabling container to be device aware and removing allocated resources when the container is deleted. Areas like resource management is explicitly left out of CDI (and is expected to be handled by the orchestrator). Because of this focus, CDI has a wide range of support and the specification is simple to implement.

As well as the specification, this repository contains reference plugins.

## Why is CDI needed?

On Linux, making a container device aware used to be as simple as exposing a device node in that container. As device and software grows more complex, making a container device aware requires more operations to be performed, such as:
    - Host / Device / Container requirement checks 
    - Exposing multiple device nodes from the host (e.g: control nodes)
    - Exposing libraries or executable from the host
    - Hiding procfs entries
    - Performing runtime specific operations (e.g: VM vs linux containers based runtimes)
    - Cleaning up the device adter a container stopped (e.g: scrubbing the memory)

There is also currently no standard to write a plugin for container runtimes. This leads vendors having to write multiple plugins for different runtimes, as well as runtimes and orchestrator implementing their own plugin system.

## Issues and Contributing

[Checkout the Contributing document!](CONTRIBUTING.md)

* Please let us know by [filing a new issue](https://github.com/RenaudWasTaken/cdi/issues/new)
* You can contribute by opening a [pull request](https://help.github.com/articles/using-pull-requests/)
