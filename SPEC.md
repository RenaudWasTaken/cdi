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

This document describes how container "runtimes" should interact with third party devices in a pluggable way.
The specification described in this document is called the _Container Device Interface_, or _CDI_.

The Container Device Interface works as follows:
A. Installation
   1) A user installs a third party device driver.
   2) The device driver installation software drops a JSON file at a well known path.
B. Runtime
   3) A user runs a container with the argument `--device` followed by a device name.
   4) The container runtime reads the JSON file.
   5) The container runtime validates that the device is descried in the JSON file.
   6) The container runtime pulls the container image.
   7) The container runtime generates an OCI specification.
   8) The container runtime transforms the OCI specification according to the instructions in the JSON file


The key words "must", "must not", "required", "shall", "shall not", "should", "should not", "recommended", "may" and "optional" are used as specified in [RFC 2119][rfc-2119].

[namespaces]: http://man7.org/linux/man-pages/man7/namespaces.7.html
[rfc-2119]: https://www.ietf.org/rfc/rfc2119.txt

## General considerations

- The device configuration is in JSON format and can easily be stored in a file.
- The device configuration includes mandatory fields such as "name" and "version".
- Fields in CDI structures are required unless specifically marked optional.

For the purposes of this proposal, we define two terms very specifically:
- _container_ can be considered synonymous with a [Linux _network namespace_][namespaces].
  What unit this corresponds to depends on a particular container runtime implementation. 
- _device_ which will refer to an actual hardware device.

Whilst there are certain well known fields, runtimes may wish to pass additional information to plugins. These extensions are not part of this specification but are documented as [conventions](CONVENTIONS.md).

## CDI JSON Specification

```
{
  "cdiVersion": "0.4.0",
  "devices": [
      {
          "name": "<name>",
          "devicePath": "<MAC address>",
      }
  ],
  "containerSpec": [
      {
          "devices": [ (optional)
              {
                  "hostPath": "<path>",
                  "containerPath": "<path>",
                  "permissions"
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
                          "env":  [ "<envName>=<envValue>"] (optional)
                      }
              },
          ]
      }
  ],
```

`cdiVersion` specifies a [Semantic Version 2.0](https://semver.org) of CDI specification used by the vendor. 

## CDI CLI Specification
