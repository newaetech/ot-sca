# SCA Evaluation of OpenTitan ECC Primitives

## Report Details

See [REPORT.md](REPORT.md) for details of the evaluation.

See the [/jupyter](../../jupyter) folder to recreate and extend the evaluation. Those notebooks should work using ChipWhisperer, which you can start in that directory with:

```console
jupyter notebook
```

See the [ChipWhisperer Documentation](https://chipwhisperer.readthedocs.io/en/latest/starting.html#starting) if you need more help with setup.

## Software SEtup

### Software for Evalation

The following references code currently on the `sca_ecc` branch (available [here](https://github.com/newaetech/opentitan/tree/sca_ecc/sw/device)). Evaluation was conducted using firmware compiled from [this commit](https://github.com/newaetech/opentitan/tree/b0eb66367b9e8dafccc466822da5f4315b83e8f9). 

### P256 Evaluation Code

The code for evaluation is the `ecc_serial` application (code available [here](https://github.com/newaetech/opentitan/blob/sca_ecc/sw/device/sca/ecc_serial.c)). This application directly uses the `p256_ecdsa` application, with minor adjustments
to allow non-random blinding values and arbitrary `k` values to be inserted. The modified application can be seen in the
[p256_ecdsa_sca.s](https://github.com/newaetech/opentitan/blob/sca_ecc/sw/otbn/code-snippets/p256_ecdsa_sca.s) file.

The `p256_ecdsa_setup_rand` function has been modified to avoid overwriting the `rnd` and `k` values. These values are now
set from the high-level `ecc_serial.c` file.

### P384 Evaluation Code

The code for evaluation is the `ecc384_serial` application (code available [here](https://github.com/newaetech/opentitan/blob/sca_ecc/sw/device/sca/ecc384_serial.c)). As there is no full ecc test application (like with the p256 example), this file was created to run only the sign operation.
The code is available in the new [p384_ecdsa_sca.s](https://github.com/newaetech/opentitan/blob/sca_ecc/sw/otbn/code-snippets/p384_ecdsa_sca.s) file.

This is designed only for usage with the SCA experiments, and not as a general-purpose evaluation.

## Hardware Setup
Power traces were collected using ChipWhisperer Husky and a CW310 Bergen target board with a K410T FPGA. The OpenTitan FPGA bitfile was generated from [this commit](https://github.com/newaetech/opentitan/tree/b0eb66367b9e8dafccc466822da5f4315b83e8f9).

The general setup is the same as described in the [CW310/Husky Readme](https://github.com/newaetech/ot-sca/blob/cwhusky-doc/doc/getting_started_bergen.md). We suggest you first confirm the SCA attack on the AES core works before building the ECC examples.

### Loading Software Examples

Note that the console will produce invalid data often, as the clock frequency of the ECC core may be run at non-standard frequencies. If you want to view the debug output (when it works), run:

```console
screen /dev/ttyACM1 115200,cs8,-ixon,-ixoff
```

To load the bitstream, as normal run:

```console
util/fpga/./cw310_loader.py -b build/lowrisc_systems_chip_earlgrey_cw310_0.1/synth-vivado/lowrisc_systems_chip_earlgrey_cw310_0.1.bit
```

To load the evaluation firmware for the p256 curve:

```console
util/fpga/./cw310_loader.py --firmware build-bin/sw/device/sca/ecc_serial_fpga_cw310.bin --set-pll-defaults
```

To load the evaluation firmware for the p384 curve:

```console
util/fpga/./cw310_loader.py --firmware build-bin/sw/device/sca/ecc384_serial_fpga_cw310.bin --set-pll-defaults
```