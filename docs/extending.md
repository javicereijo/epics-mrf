Extending the device support
============================

This device support is designed in a modular fashion, so that it is very easy to
extend it to additional devices or even buses.


Adding a new device type
------------------------

I most cases, adding support for a new device type is as simple as adding a few
lines of code.

The record (`.db`) file and the Autosave request file (`.req`) for each device
are generated by shell scripts. These scripts can be found in `mrfApp/Db` and
are named

- `generate_evg_autosave_request_file.sh` (generates the Autosave request files
  for EVG devices),
- `generate_evg_records.sh` (generates the record files for EVG devices), and
- `generate_evr.sh` (generates bot the record files and the Autosave request
  files for EVR devices).

Due to the fact that the different devices are very similar and mainly differ in
details like the number of inputs and outputs, it is usually sufficient to
adjust the respective lines in the script that generate the records.

If you have to add entirely new records (e.g. due to a device introducing new
features that are not present in other devices), you can do so without having to
write code. You only have to add a record definition that specifies the memory
address and the type (`uint16` or `uint32`) of the register. Please have a look
at the existing record definitions for examples.

If needed, you can only use certain bits of a register for the record's value.
Have a look at the `DBus:Status` record for the EVG (in `evg-common.inc.db`) for
an example.

There are additional options that can be specified as part of the address string
when needed:

- `no_read_on_init`: This option has the effect that the record's value is not
  read from the device on IOC initialization. This flag is only supported for
  output records. For input records, the value is only read from the device when
  the record is processed.
- `no_verify`: This option has the effect that the value written to the device
  is not verified by reading back from the device. This flag implies
  `no_read_on_init`. This flag is only supported for output records.

For arrays, there are two additional options:

- `element_distance`: This option specifies the distance between two elements
  stored in the array. The distance is measured in bytes. This is useful when
  the data of different arrays is interleaved in the device's memory (the
  sequence RAMs of the EVG are examples for such arrays).
- `changed_elements_only`: This option specifies that when the arrays is
  changed, only the changed elements shall be written to the hardware. If this
  option is not specified, the complete array is written to the hardware. This
  flag is only supported for output records.

When adding support for one of the following devices, you will not have to add
an IOC shell function, because they are already implemented in the device
support (only the record files and panels are missing):

- cPCI-EVG-220 (`mrfMmapCpciEvg220Device`)
- cPCI-EVG-230 (`mrfMmapCpciEvg230Device`)
- cPCI-EVG-300 (`mrfMmapCpciEvg300Device`)
- cPCI-EVR-220 (`mrfMmapCpciEvr220Device`)
- cPCI-EVR-230 (`mrfMmapCpciEvr230Device`)
- cPCI-EVR-300 (`mrfMmapCpciEvr300Device`)
- cPCI-EVRTG-300 (`mrfMmapCpciEvrtg300Device`)
- PCIe-EVR-300 (`mrfMmapPcieEvr300Device`)
- PMC-EVR-230 (`mrfMmapPmcEvr230Device`)
- PXIe-EVG-300 (`mrfMmapPxieEvg300Device`)
- PXIe-EVR-300 (`mrfMmapPxieEvr300Device`)

In fact, all these functions are simply aliases for the
`mrfMmapMtcaEVR300Device` function. For a device that is controlled via UDP/IP,
you can most likely simply use the existing `mrfUdpIpEvgDevice` or
`mrfUdpIpEvrDevice` functions. The only difference between these two functions
is the memory offset used when addressing registers inside the device.

If your want to add support for a different device, you might have to an an IOC
shell function definition to `mrfApp/mrfEpicsMmapSrc/mrfRegistrarMmap.cpp` (for
a device using the Linux kernel driver) or
`mrfApp/mrfEpicsUdpIpSrc/mrfRegistrarUdpIp.cpp` (for a device using the UDP/IP
based protocol).

Last but not least, you will have to extend the panels to accommodate for the
new device. Depending on how this device differs from other devices, this might
be as simple as creating a copy of the sub-panel for inputs and outputs and
adjusting the number of inputs and outputs in these panels, or you might have to
create a new main panel.


Adding support for a new bus
----------------------------

If you want to add support for a new type of bus (e.g. VMEbus), you can do so by
implementing a respective interface. Existing record files can be reused because
the configuration in those files (and in fact most of the EPICS side of this
device support) does not have to know anything about the backend actually
implementing the communication with the hardware.

To create such a backend, create a new sub-directory in the `mrfApp` directory
(e.g. `mrfMyBusSrc`) and add your source files that implement the
`MrfMemoryAccess` interface there. You can use the classes
`MrfMmapMemoryAccess` (the implementation using the Linux kernel driver) and
`MrfUdpIpMemoryAccess` (the implementation for the UDP/IP based protocol) as
examples.

When you have implemented the interface, you have to create an IOC shell
function that takes care of creating an instance of your class and registering
it. This IOC shell function should be put in another sub-directory of `mrfApp`
(e.g. `mrfMyBusEpicsSrc`). You can have a look at
`mrfApp/mrfEpicsMmapSrc/mrfRegistrarMmap.cpp` and
`mrfApp/mrfEpicsUdpIpSrc/mrfRegistrarUdpIp.cpp` for examples.