# Zenoh-Pico examples for Zephyr RTOS

## Purpose
This repository contains some proof-of-concept applications that show how to build and use [Zenoh-Pico](https://github.com/eclipse-zenoh/zenoh-pico) on [Zephyr RTOS](https://www.zephyrproject.org)

## Usage:
This repository can be cloned and used like any generic Zephyr application.
The `CMakeLists.txt` is currently set up to fetch and build Zenoh-Pico library and an example `z_pub` application. Nevertheless, it can be easily modified or extended for other application code.

For example, to build `z_pub` for the `rpi_pico` target:
```
$ west build -b rpi_pico .
```
