# oneplus-infiniti-linux-firmware

Vendor firmware blobs extracted from a stock OnePlus 15 (project
codename `Infiniti`, Snapdragon 8 Elite Gen 5 / `kaanapali` / `sm8850`)
for use with the mainline Linux kernel.

The directory layout mirrors
[`nubia-sm8850/linux-firmware`](https://github.com/nubia-sm8850/linux-firmware)
so a system installer can copy
`lib/firmware/qcom/kaanapali/OnePlus/Infiniti/` out and either symlink
each blob into the standard `lib/firmware/qcom/kaanapali/` lookup path
the kernel asks for, or override `firmware_class.path` on the kernel
command line.

## What's in here

`lib/firmware/qcom/kaanapali/OnePlus/Infiniti/`:

| File              | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| `adsp.mbn`        | Audio DSP (Hexagon) main image                   |
| `adsp_dtb.mbn`    | Devicetree blob for ADSP                         |
| `adspr.jsn`       | ADSP root PD signature manifest                  |
| `adsps.jsn`       | ADSP "sensors" PD signature manifest             |
| `adspua.jsn`      | ADSP "audio" user PD signature manifest          |
| `adspuo.jsn`      | ADSP "audio output" user PD signature manifest   |
| `cdsp.mbn`        | Compute DSP (Hexagon) main image                 |
| `cdsp_dtb.mbn`    | Devicetree blob for CDSP                         |
| `cdspr.jsn`       | CDSP root PD signature manifest                  |
| `gen80200_zap.mbn`| Adreno A830 "zap" shader (signed GPU bringup)    |
| `modem.mbn`       | Modem (MPSS / Hexagon) main image                |
| `modem_dtb.mbn`   | Devicetree blob for the modem                    |
| `modemr.jsn`      | Modem root PD signature manifest                 |
| `soccp.mbn`       | SoCCP main image (RISC-V always-on co-processor) |
| `soccp_dtb.mbn`   | Devicetree blob for SoCCP                        |

`usr/share/qcom/kaanapali/OnePlus/Infiniti/sensors/`:

| File / dir           | Purpose                                       |
| -------------------- | --------------------------------------------- |
| `config/*.json`      | Per-board / per-sensor config, consumed by    |
|                      | the SLPI sensor stack via `json.lst`.         |
| `config/json.lst`    | Index of which JSONs to load on this device.  |
| `sns_reg.conf`       | Sensor registry config (path layout).         |
| `sns_reg_config`     | Same content as `sns_reg.conf` (vendor copy). |
| `sns_reg_version`    | Registry schema version stamp.                |
| `file1` / `file2`    | Empty stubs for the runtime registry          |
|                      | property store (populated at first boot).     |

OnePlus ships its own SoCCP pair on a dedicated `soccp_firmware`
partition (not on `modem`); the blobs here are a verbatim copy and
differ in size / content from the generic mainline `linux-firmware`
`qcom/kaanapali/soccp.mbn` shipped with the kernel tree. The OnePlus
dump does **not** include a `soccpr.jsn` (root PD signature
manifest) - the mainline `linux-firmware` `soccpr.jsn` is reused for
that. SoCCP is brought out of reset by XBL/SBL, so the kernel takes
the `early_boot=true` / `RPROC_DETACHED` attach path and never
actually re-authenticates the `.mbn` - shipping the OnePlus blobs
is primarily so a recovery path / re-load matches what XBL flashed.

## Source

Everything was extracted from a stock OnePlus 15 (`Infiniti`) build:

- The `adsp` / `cdsp` / `modem` `.mbn` and `_dtb.mbn` files were
  reconstructed from the original split PIL form
  (`<name>.mdt` plus `<name>.b00 .. .bNN`) found in the
  `firmware_mnt/image/kaanapali/` directory of the device by running
  [`pil-squasher`](https://github.com/linux-msm/pil-squasher)
  once per subsystem:

      pil-squasher /out/adsp.mbn ./adsp.mdt
      pil-squasher /out/cdsp.mbn ./cdsp.mdt
      pil-squasher /out/modem.mbn ./modem.mdt
      # ... etc, plus the *_dtb.mbn variants.

- The `.jsn` signature manifests are copied verbatim from the same
  `kaanapali/` image directory, no squashing necessary.
- `gen80200_zap.mbn` is copied verbatim from the device's
  `vendor/firmware/` directory.
- `soccp.mbn` and `soccp_dtb.mbn` are copied verbatim from
  `vendor/soccp_firmware/image/`.
- The `sensors/` tree was assembled from `vendor/etc/sensors/` (the
  JSONs and `sns_reg_config`) and `mnt_vendor/persist/sensors/registry/`
  (`sns_reg_version`). `file1` / `file2` are empty by design, matching
  the layout of [`nubia-sm8850/linux-firmware`](https://github.com/nubia-sm8850/linux-firmware).

## Install

`make install DESTDIR=/` mirrors the `lib/` and `usr/` trees onto the
target rootfs, dropping every blob to mode 0644 under the same path.

For mainline-style firmware loading (kernel asks for
`qcom/kaanapali/<name>.mbn`) the simplest hookup is symlinks:

    cd /lib/firmware/qcom/kaanapali
    ln -sf OnePlus/Infiniti/adsp.mbn         adsp.mbn
    ln -sf OnePlus/Infiniti/adsp_dtb.mbn     adsp_dtb.mbn
    ln -sf OnePlus/Infiniti/cdsp.mbn         cdsp.mbn
    ln -sf OnePlus/Infiniti/cdsp_dtb.mbn     cdsp_dtb.mbn
    ln -sf OnePlus/Infiniti/gen80200_zap.mbn gen80200_zap.mbn
    ln -sf OnePlus/Infiniti/modem.mbn        modem.mbn
    ln -sf OnePlus/Infiniti/modem_dtb.mbn    modem_dtb.mbn
    ln -sf OnePlus/Infiniti/soccp.mbn        soccp.mbn
    ln -sf OnePlus/Infiniti/soccp_dtb.mbn    soccp_dtb.mbn

(`soccpr.jsn` is not provided here; keep the mainline
`linux-firmware` copy for that one.)