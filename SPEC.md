# Container Device Interface Specification

- [Version](#version)
- [Overview](#overview)
- [General considerations](#general-considerations)
- [CDI JSON Specification](#well-known-error-codes)
- [CDI CLI Specification](#well-known-error-codes)

## Version

This is CDI **spec** version **0.1.0**.

#### Released versions

Released versions of the spec are available as Git tags.

| Tag  | Spec Permalink   | Change |
| -----| -----------------| -------|

## Overview

The _Container Device Interface_, or _CDI_ describes a mechanism for container runtimes to create containers which are able to interact with third party devices.

For third party devices, it is often the case that interacting with these devices require container runtimes to expose more than a device node. For example a third part device might require you to have kernel modules loaded, host libraries mounted or specific procfs path exposed/masked.

The _Container Device Interface_ describes a mechanism which allows third party vendors to perform these operations such that it doesn't require changing the container runtime.

The mechanism used is a JSON file (similar the [Container Network Interface (CNI)][cni]) which allows vendors to describe the operations the container runtime should perform on the container's [OCI specification][oci].

The Container Device Interface enables the following two flows:

A. Device Installation
   1. A user installs a third party device driver (and third party device) on a machine.
   2. The device driver installation software writes a JSON file at a well known path (`/etc/cdi/vendor.json`).

B. Container Runtime
   1. A user runs a container with the argument `--device` followed by a device name.
   2. The container runtime reads the JSON file.
   3. The container runtime validates that the device is described in the JSON file.
   4. The container runtime pulls the container image.
   5. The container runtime generates an OCI specification.
   6. The container runtime transforms the OCI specification according to the instructions in the JSON file


[cni]: https://github.com/containernetworking/cni
[oci]: https://github.com/opencontainers/runtime-spec

## General considerations

- The device configuration is in JSON format and can easily be stored in a file.
- The device configuration includes mandatory fields such as "name" and "version".
- Fields in CDI structures are required unless specifically marked optional.

For the purposes of this proposal, we define two terms very specifically:
- _container_ can be considered synonymous with a [Linux _network namespace_][namespaces].
  What unit this corresponds to depends on a particular container runtime implementation. 
- _device_ which will refer to an actual hardware device.

The key words "must", "must not", "required", "shall", "shall not", "should", "should not", "recommended", "may" and "optonal" are used as specified in [RFC 2119][rfc-2119].

[rfc-2119]: https://www.ietf.org/rfc/rfc2119.txt
[namespaces]: http://man7.org/linux/man-pages/man7/namespaces.7.html

## CDI JSON Specification

### JSON Definition

```
{
    "cdiVersion": "0.1.0",
    "kind": "<kind>",
    "kindShort": ["<short-kind>"], (optional)

    "devices": [
        {
            "name": "<name>",
            "nameShort": ["<short-name>", "<short-name>"], (optional)
            "hostPath": "<path>",
            "containerPath": "<path>",

            // Cgroups permissions of the device, candidates are one or more of
            // * r - allows container to read from the specified device.
            // * w - allows container to write to the specified device.
            // * m - allows container to create device files that do not yet exist.
            "permissions": "<permissions>" (optional)
        }
    ],

    "containerSpec": [
        {
            "devices": [ (optional)
                {
                    "hostPath": "<path>",
                    "containerPath": "<path>",

                    // Cgroups permissions of the device, candidates are one or more of
                    // * r - allows container to read from the specified device.
                    // * w - allows container to write to the specified device.
                    // * m - allows container to create device files that do not yet exist.
                    "permissions": "<permissions>" (optional)
                }
            ]
            "mounts": [ (optional)
                {
                    "hostPath": "<source>",
                    "containerPath": "<destination>",
                    "options": "<OCI Mount Options>", (optional)
                }
            ],
            "hooks": [ (optional)
                {
                    "<OCI Hook Name>":
                        {
                            "path": "<path>",
                            "args": ["<arg>", "<arg>"], (optional)
                            "env":  [ "<envName>=<envValue>"], (optional)
                            "timeout": <int> (optional)
                        }
                }
            ]
        }
    ]
}
```

### Field Description

Fields have the following definition and requirements:

#### Version

The `cdiVersion` field specifies a [Semantic Version 2.0](https://semver.org) of the CDI specification used by the vendor.

#### Kind

The `kind` field specifies a label which uniquely identifies the device vendor.
It can be used on the CLI to disambiguate the vendor that matches a device, e.g: `docker/podman run --device vendor.com/device=foo ...`.

_Error handling:_
  * Two or more files with identical `kind` values.
    Container runtimes should surface this error when any device with that `kind` is requested.

#### KindShort
The `kindShort` field is a list of labels that can be used by vendors to disambiguate the vendor but reduce the verbosity of the CLI.
For example: `{"kind": "vendor.com/hwdevice", "kindShort": ["hwdevice"]}` paired with the following CLI, `docker/podman run --device hwdevice=foo ...`.

_Error handling:_
  * Two or more files with identical `kindShort` values.
    Container runtimes should only surface an error if the CLI request is ambiguous.

_Label Format_
The `kind` and `kindShort` labels have two segments: a prefix and a name, separated by a slash (/). The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (\_), dots (.), and alphanumerics between. The prefix must be a DNS subdomain: a series of DNS labels separated by dots (.), not longer than 253 characters in total, followed by a slash (/).

Examples (not an exhaustive list):
  * Valid: `vendor.com/foo`, `foo.bar.baz/foo-bar123.B_az`
  * Invalid: `foo`, `vendor.com/foo/`, `vendor.com/foo/bar`

#### Devices

The `devices` field describes the set of hardware devices that can be refered to by the CLI.

A device has a `name` that can be used to refer to it on the CLI: `docker/podman run --device foo ...`.
A device has a list of `nameShort` that can be used to refer to it on the CLI: `docker/podman run --device foo ...`.

A device will always match a linux device node.
- `names` is a list of names that can be used to refer to a specific device.
- `devicePath` is the path of the device node to be 

**Container Spec**

## CDI CLI Specification
