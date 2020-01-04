# CDI - The Container Device Interface
## What is CDI?
CDI (Container Device Interface), is a specification, for container runtimes, to support third party devices. 

CDI concerns itself only with enabling container to be device aware and removing allocated resources when the container is deleted. Areas like resource management is explicitly left out of CDI (and is expected to be handled by the orchestrator). Because of this focus, CDI has a wide range of support and the specification is simple to implement.

As well as the specification, this repository contains reference plugins.
CDI is based on the Container Networking Interface model and specification.

## Why is CDI needed?

On Linux, making a container device aware used to be as simple as exposing a device node in that container. As device and software grows more complex, making a container device aware requires more operations to be performed, such as:
    - Expose the device to the container. This often requires several operations such as exposing additional device nodes, mounting files from the runtime namespace or hide procfs entries.
    - Perform compatibility checks between the container and the device.
    - Perform runtime specific operations (e.g: VM vs linux containers based runtimes).
    - Clean up the device after a container stopped (e.g: scrubbing the memory).

Finally in the absence of a standard for third party devices, vendors often have to write and maintain multiple plugins for different runtimes.
Additionally runtimes don't uniformly expose a plugin system (or even expose a plugin system at all) leading to duplication of the functionality in higher level abstractions (such as Kubernetes device plugins).

## Examples
```bash
$ mkdir /etc/cdi
$ cat > /etc/cdi/vendor.json <<EOF
{
  "cdiVersion": "0.2.0",
  "kind": "vendor.com/device",
  "devices": [
    { "name": "myDevice", "devicePath": "/dev/vendor0"}
    { "name": "myDevice2", "devicePath": "/dev/vendor1"}
  ],
  "containerSpec": {
    "devices": [
      {"HostPath": "/dev/vendorctl", "ContainerPath": "/dev/vendorctl"},
    ],
    "mounts": [
      {"HostPath": "/bin/vendorBin", "ContainerPath": "/bin/vendorBin"},
      {"HostPath": "/usr/lib/libVendor.so.0", "ContainerPath": "/usr/lib/libVendor.so"}
    ],
    "hooks": [
      {"create-container": {"path": "/bin/vendor-hook"} },
      {"start-container": {"path": "/usr/bin/ldconfig"} },
    ]
  }
}
EOF

```

## Issues and Contributing

[Checkout the Contributing document!](CONTRIBUTING.md)

* Please let us know by [filing a new issue](https://github.com/RenaudWasTaken/cdi/issues/new)
* You can contribute by opening a [pull request](https://help.github.com/articles/using-pull-requests/)
