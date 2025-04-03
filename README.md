# usb_midi_device_multistream
An application USB MIDI class device driver for TinyUSB that supports multiple virtual cables

The native TinyUSB USB MIDI class device driver for TinyUSB one supports one virtual cable in
and one virtual cable out. This library adds a new device driver to TinyUSB that allows you to
add up to 16 virtual cables in and 16 virtual cables out. It also supports string descriptor
labels for all of the MIDI virtual cables.
