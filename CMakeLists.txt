cmake_minimum_required(VERSION 3.13)

add_library(usb_midi_device_multistream INTERFACE)
target_sources(usb_midi_device_multistream INTERFACE
    ${CMAKE_CURRENT_LIST_DIR}/midi_device_multistream.c
)

target_include_directories(usb_midi_device_multistream INTERFACE ${CMAKE_CURRENT_LIST_DIR})