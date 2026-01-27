# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Atopile hardware design project for a 125KHz RFID system. It uses an RDM6300 RFID reader, Pro Micro nRF52840 microcontroller, and Wio SX1262 LoRa module to read and transmit RFID data over a Meshtastic mesh network.

## Build Commands

```bash
# Sync dependencies
ato sync

# Build the project
ato build

# Build with locked dependencies (CI mode)
ato build --frozen
```

## Project Configuration

- **Atopile version**: 0.12.4 (specified in ato.yaml)
- **Entry point**: `main.ato:App`
- **Source directory**: `./`
- **Layouts directory**: `./layouts`

## Architecture

This is an Atopile "hardware-as-code" project where electronic circuits are defined programmatically in `.ato` files. The `App` module in `main.ato` serves as the top-level integration point for the design.

Build artifacts are generated in the `build/` directory.
