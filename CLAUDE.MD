# dump1090-mutability

## Project Overview

dump1090-mutability is a Mode S/ADS-B decoder for RTL-SDR software-defined radio devices. It receives and decodes ADS-B messages from aircraft transponders, displaying aircraft positions, altitudes, speeds, and other telemetry data on a web-based map interface.

This is a fork of MalcolmRobb's dump1090, enhanced with:
- 2.4MHz oversampling support
- Debian/Raspbian packaging
- Non-root operation
- FlightAware TSV format support
- External webserver integration (lighttpd)
- Improved network code
- Systemd/init.d integration

**License**: GPL v2 or later

## Architecture

### Core Components

1. **Mode S Decoder** (`mode_s.c`, `mode_ac.c`)
   - Decodes Mode S and Mode A/C messages from aircraft transponders
   - Handles various ADS-B message types (position, velocity, identification, etc.)
   - Error correction using CRC validation

2. **Signal Demodulation** (`demod_2400.c`, `dump1090.c`)
   - Processes raw IQ samples from RTL-SDR
   - Supports both standard (2MHz) and oversampling (2.4MHz) modes
   - Magnitude detection and bit extraction

3. **Aircraft Tracking** (`track.c`)
   - Maintains state for all detected aircraft
   - Calculates positions using CPR (Compact Position Reporting)
   - Tracks altitude, speed, heading changes over time

4. **Network Services** (`net_io.c`, `anet.c`)
   - Multiple output formats: raw, Beast, SBS BaseStation, FlightAware TSV
   - TCP server for real-time data distribution
   - HTTP server (optional) or JSON file output for web interface

5. **Web Interface** (`public_html/`)
   - OpenLayers-based interactive map
   - Real-time aircraft display with filters
   - Historical track display
   - Multiple map layer options (OpenStreetMap, Bing, Mapzen)

### Key Source Files

- `dump1090.c` / `dump1090.h` - Main program, initialization, SDR interface
- `mode_s.c` - Mode S message decoding logic
- `track.c` / `track.h` - Aircraft tracking and state management
- `net_io.c` / `net_io.h` - Network I/O, multiple protocol support
- `demod_2400.c` / `demod_2400.h` - 2.4MHz oversampling demodulator
- `cpr.c` / `cpr.h` - CPR position decoding
- `crc.c` / `crc.h` - CRC calculation and error detection
- `icao_filter.c` / `icao_filter.h` - ICAO address filtering
- `stats.c` / `stats.h` - Statistics collection and reporting
- `convert.c` / `convert.h` - Unit conversion utilities
- `interactive.c` - Interactive command-line display
- `view1090.c` - Standalone viewer application
- `faup1090.c` - FlightAware uploader

### Web Interface Files

- `public_html/script.js` - Main map and aircraft display logic
- `public_html/planeObject.js` - Aircraft object representation
- `public_html/markers.js` - Aircraft marker rendering
- `public_html/layers.js` - Map layer configuration
- `public_html/config.js` - User configuration (coordinates, API keys)
- `public_html/gmap.html` - Legacy Google Maps interface
- `public_html/closest.html` - Display of closest aircraft

## Build System

### Building

```bash
# Debian package build (recommended)
sudo apt-get install librtlsdr-dev libusb-1.0-0-dev pkg-config debhelper
dpkg-buildpackage -b

# Direct make build
make
```

### Build Options

- `HAVE_RTL_BIAST` - Enable bias-tee support for RTL-SDR.com V3 dongles
- Standard CFLAGS for optimization and debugging

### Dependencies

- librtlsdr (RTL-SDR library)
- libusb-1.0 (USB device access)
- pthread (POSIX threads)

## Configuration

### Primary Configuration

- Debian package: `/etc/default/dump1090-mutability`
- Reconfigure: `dpkg-reconfigure dump1090-mutability`

### Default Settings

- Network ports bound to localhost only (change to 0.0.0.0 for remote access)
- Internal HTTP server disabled (use external webserver)
- Runs as dedicated user (not root)
- JSON output to `/run/dump1090-mutability/` (tmpfs)

### Web Configuration

Edit `public_html/config.js` to set:
- Receiver latitude/longitude
- Map center and zoom
- Site name
- API keys for Bing/Mapzen tiles

## Development Guidelines

### Code Style

- Follow existing code style in the file being modified
- Use consistent indentation (spaces, not tabs in most files)
- Add comments for complex logic, especially in decoder/demodulator code

### Testing

- Test with real RTL-SDR hardware when possible
- Verify multiple aircraft are tracked correctly
- Check network output formats haven't changed
- Test web interface in multiple browsers

### Common Development Tasks

#### Adding New Message Types

1. Add decoding logic to `mode_s.c`
2. Update aircraft state in `track.c` if needed
3. Add fields to JSON output in `net_io.c`
4. Update web display in `public_html/script.js`

#### Modifying Network Protocols

1. Update protocol handlers in `net_io.c`
2. Test with downstream consumers (FlightAware, etc.)
3. Document format changes

#### Web Interface Changes

1. Modify JavaScript in `public_html/`
2. Test with JSON output format
3. Check browser console for errors
4. Verify performance with many aircraft

### Debugging

- Enable debug mode: `--debug` flags
- Check log file: `/var/log/dump1090-mutability.log`
- Interactive mode: `--interactive` for real-time display
- Network debugging: monitor ports with `netcat` or `telnet`

## Project Structure

```
dump1090-mutability/
├── debian/              # Debian packaging files
├── public_html/         # Web interface
│   ├── *.js            # JavaScript for map display
│   ├── *.html          # HTML pages
│   ├── ol3/            # OpenLayers library
│   ├── flags-tiny/     # Country flag icons
│   └── db/             # Aircraft database
├── compat/             # Compatibility shims
├── tools/              # Utility scripts
├── *.c, *.h            # C source and headers
├── Makefile            # Build configuration
└── README*.md          # Documentation

```

## Useful Commands

```bash
# Run dump1090 in interactive mode
./dump1090 --interactive --net

# Enable all output formats
./dump1090 --net --net-ro-port 30002 --net-sbs-port 30003

# Use 2.4MHz oversampling
./dump1090 --oversample

# Set receiver location
./dump1090 --lat 40.7128 --lon -74.0060

# Enable stats
./dump1090 --stats-every 60
```

## Resources

- [Original dump1090](https://github.com/antirez/dump1090)
- [MalcolmRobb's fork](https://github.com/MalcolmRobb/dump1090)
- [librtlsdr](https://github.com/mutability/librtlsdr)
- [Mode S decoding reference](http://mode-s.org/)
- [ADS-B specification](https://www.icao.int/safety/acp/Repository/WGW14_WP1100_ADSB_MOPS.pdf)

## Notes for Claude

- This is a low-level C project dealing with RF signal processing
- Changes to decoder/demodulator code require careful testing
- Performance matters - this runs on Raspberry Pi hardware
- The web interface should work without requiring a build step
- Configuration via Debian packaging system when installed as package
- Check existing issues before implementing features
- Maintain compatibility with downstream tools (FlightAware, etc.)
