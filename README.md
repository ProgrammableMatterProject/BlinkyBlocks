# Blinky Blocks

## bb-apploader

The blinkyApploader library aims at loading ihex files to a set of Blinky Blocks through an entry point (serial connection) in the blocks set. This requires the blocks to be set in programming mode. This library is used as dependency for **bb-cli** and **bb-gui**.

## bb-cli

Command line interface for Blinky Blocks serial interactions.

## bb-gui

Crossplatform graphical user interface based on **bb-cli** to handle serial and wireless programming.

## bb-bootloader

C firmware running on the Blinky Blocks hardware including *BB Parallel Programming* protocol.

## blinky-network-to-arduino

Arduino firmware for interfacing external sensors and wireless devices with the Blinky Blocks.

---

Easily deploy Blinky Blocks projects.

This repo gather both CLI and GUI interfaces, and their dependencies.

`git clone --recurse-submodules <thisRepo>`
This will automatically initialize and update each submodule in the repository

If you already cloned the project and forgot to update submodule
`git submodule update --init --recursive`
