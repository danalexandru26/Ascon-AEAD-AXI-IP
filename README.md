# verilog_ascon_reference

Standalone Vivado project for bringing up the **official Ascon-AEAD128 hardware core**
([rprimas/ascon-verilog](https://github.com/rprimas/ascon-verilog), final NIST SP 800-232)
on the Zynq-7020, before it is integrated into the CANsec-over-CAN-FD design.

| | |
|---|---|
| Board | Digilent Arty Z7-20 |
| Part | `xc7z020clg400-1` |
| Tool | Vivado 2025.2.1 |
| Clock | 100 MHz (`timing.xdc`, 10 ns on `clk`) |
| Top (current) | `ascon_core` |

## Layout

```
verilog_ascon_reference/
├── verilog_ascon_reference.xpr        Vivado project file (open this)
├── verilog_ascon_reference.srcs/
│   ├── sources_1/
│   │   ├── new/ascon_core_all.sv      old `include umbrella, not used by the project
│   │   └── bd/ASCON_AXI/ASCON_AXI.bd  block design: Zynq PS + SmartConnect + reset + Ascon AXI IP
│   └── constrs_1/new/timing.xdc       100 MHz clock constraint
├── verilog_ascon_reference.ip_repo/
│   └── ASCON_AXI4_LITE_1_0/           custom AXI-Lite IP (hdl/, xgui/, component.xml, drivers/)
├── README.md
└── .gitignore
```

The generated folders (`.cache`, `.runs`, `.sim`, `.hw`, `.gen`, `.ip_user_files`)
are not tracked. Vivado recreates them when you open the project and run synthesis.

## Ascon core source (outside this folder)

The core RTL is **not copied** into this project. The `.xpr` points to:

```
D:\Verilog_Ascon_Reference\ascon-verilog\vivado\ascon_core_flat.sv
```

which lives in a clone of upstream `rprimas/ascon-verilog` at commit `20381a4`.

- `ascon_core_flat.sv` is the five upstream RTL files (`config.sv`, `functions.sv`,
  `register.sv`, `asconp.sv`, `ascon_core.sv`) concatenated in that order, **unchanged**.
  This is needed because Vivado compiles each file on its own, and upstream files
  do not `include` each other.
- The same `vivado\` folder holds the testbench `tb_ascon_core.sv` and the test
  vectors `ascon_vectors.txt`.

The path is relative in the `.xpr`, so keep the same folder structure
(`D:\FPGA_Dev\...` and `D:\Verilog_Ascon_Reference\...`) on any machine you open this on.

## Results so far

- **Simulation:** passes all 1289 test vectors (variants V1 / V2 / V3).
- **Out-of-context implementation** (V1, 100 MHz, `Performance_ExplorePostRoutePhysOpt`):

  | WNS | WHS | LUT | FF |
  |---|---|---|---|
  | +0.617 ns | +0.154 ns | 1,683 | 474 |

## Work in progress: AXI-Lite IP

`ASCON_AXI4_LITE` v1.0 is an AXI4-Lite slave IP that will wrap the core. It started
from the Vivado-generated AXI4-Lite template and will get a separate sequencer
module `ascon_seq` that drives the core. It lives in
`verilog_ascon_reference.ip_repo/ASCON_AXI4_LITE_1_0/`, and the project's IP
repository setting points there with a relative path, so it moves with the project.

It is already placed in the `ASCON_AXI` block design together with the Zynq PS,
an AXI SmartConnect and the processor reset block.

## Opening the project

1. Make sure the Ascon clone exists at `D:\Verilog_Ascon_Reference\ascon-verilog`.
2. Open `verilog_ascon_reference.xpr` in Vivado 2025.2.
3. Run Synthesis / Implementation as usual. Vivado will regenerate the ignored folders.

## License

The Ascon core is by R. Primas et al. and is released under CC0 (see the upstream
repository). Project files in this folder are by Dan.
