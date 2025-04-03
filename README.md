# usb_midi_device_multistream
Extensions for the TinyUSB USB MIDI device driver enable multiple virtual cables.

The native TinyUSB USB MIDI class device driver for TinyUSB one supports one virtual cable in
and one virtual cable out. This library provides a header file to help you create a USB MIDI
descriptor with up to 16 virtual MIDI IN cables and 16 virtual MIDI OUT cables. It optionally
supports string descriptor labels for all of the MIDI virtual cables. Finally, it provides
the function `tud_midi_demux_stream_read()` that facilitates de-multiplexing each virtual
cable's MIDI messages received from USB Host's USB MIDI OUT endpoint.

This library relies on the Boost project's [preprocessor](https://github.com/boostorg/preprocessor) library.

# Integrating this library with your project
The best way to use this library is to add the source code to your project as a git submodule, add
the Boost project's `preprocessor` library as a git submodule at the same directory level,
add the `usb_midi_device_multistream` subdirectory to your CMakeLists.txt file, and
then link the `usb_midi_device_multistream` library to your targets. 

You will then need to make sure your `tusb_config.h` file devices 'CFG_TUD_NUMCABLES_IN` and `CFG_TUD_NUMCABLES_OUT`.
For example, to create a USB MIDI device that supports 2 virtual MIDI IN cables and 6 virtual MIDI OUT cables,
```
// Number of virtual MIDI cables IN to the host
#define CFG_TUD_MIDI_NUMCABLES_IN 2
// Number of virtual MIDI cables OUT from the host
#define CFG_TUD_MIDI_NUMCABLES_OUT 6
```
You will also need to decide if you want your MIDI descriptor provide string descriptors that label all of the
virtual MIDI cables by defining `CFG_TUD_MIDI_FIRST_PORT_STRIDX`. For example,
```
// Support MIDI port string labels after the serial number string
// Set this to the first available string descriptor number or
// 0 if you do not wish to label the MIDI jacks with strings
#define CFG_TUD_MIDI_FIRST_PORT_STRIDX 4
```

Your `usb_descriptors.c` file configuration descriptor definitions should call the `TUD_MIDI_MULTI_DESCRIPTOR()` macro.
For example, the full speed configuration descriptor would contain:
```
uint8_t const desc_fs_configuration[] =
{
  // Config number, interface count, string index, total length, attribute, power in mA
  TUD_CONFIG_DESCRIPTOR(1, ITF_NUM_TOTAL, 0, CONFIG_TOTAL_LEN, 0x00, 100),

  // Interface number, string index, EP Out & EP In address, EP size
  TUD_MIDI_MULTI_DESCRIPTOR(ITF_NUM_MIDI, 0, EPNUM_MIDI_OUT, (0x80 | EPNUM_MIDI_IN), 64, CFG_TUD_MIDI_NUMCABLES_IN, CFG_TUD_MIDI_NUMCABLES_OUT)
};
```
Finally, if `CFG_TUD_MIDI_FIRST_PORT_STRIDX` is not 0, make sure that `tud_descriptor_string_cb()` is able to return string
descriptors for each virtual cable. For example, for 2 IN cables and 6 OUT cables plus the 3 standard device descriptor strings
and the language ID descriptor
```
// array of pointer to string descriptors
char const* string_desc_arr [] =
{
  (const char[]) { 0x09, 0x04 }, // 0: is supported language is English (0x0409)
  "TinyUSB",                     // 1: Manufacturer
  "TinyUSB Device",              // 2: Product
  "123456",                      // 3: Serials, should use chip ID
  "MIDI IN A",
  "MIDI IN B",
  "MIDI OUT A",
  "MIDI OUT B",
  "MIDI OUT C",
  "MIDI OUT D",
  "MIDI OUT E",
  "MIDI OUT F",
};

static uint16_t _desc_str[32];

// Invoked when received GET STRING DESCRIPTOR request
// Application return pointer to descriptor, whose contents must exist long enough for transfer to complete
uint16_t const* tud_descriptor_string_cb(uint8_t index, uint16_t langid)
{
  (void) langid;

  uint8_t chr_count;

  if ( index == 0)
  {
    memcpy(&_desc_str[1], string_desc_arr[0], 2);
    chr_count = 1;
  }else
  {
    // Note: the 0xEE index string is a Microsoft OS 1.0 Descriptors.
    // https://docs.microsoft.com/en-us/windows-hardware/drivers/usbcon/microsoft-defined-usb-descriptors

    if ( !(index < sizeof(string_desc_arr)/sizeof(string_desc_arr[0])) ) return NULL;

    const char* str = string_desc_arr[index];

    // Cap at max char
    chr_count = (uint8_t) strlen(str);
    if ( chr_count > 31 ) chr_count = 31;

    // Convert ASCII string into UTF-16
    for(uint8_t i=0; i<chr_count; i++)
    {
      _desc_str[1+i] = str[i];
    }
  }

  // first byte is length (including header), second byte is string type
  _desc_str[0] = (uint16_t) ((TUSB_DESC_STRING << 8 ) | (2*chr_count + 2));

  return _desc_str;
}
```
