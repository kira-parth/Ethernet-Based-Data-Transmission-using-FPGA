# Ethernet-Based-Data-Transmission-using-FPGA


This project contains an ethernet mac which is compatible with the Digilent Nexys 4 DDR FPGA board. This MAC uses an RMII interface to transmit/receive data, and SMI to set/read the control registers on the PHY. It normalizes the RX/TX data into a stream of bytes and performs the CRC check. It supports a native stream protocol as well as AXI stream for sending and receiving packets. This project is packaged into an IP core to be included in other projects. There is also a very simple example included. This code may work or be easily adapted to work with other FPGA boards. I have a Nexys 4 DDR so that is what I tested with and what I support.

Building
As is this project can be imported as an ip core. If you want to further develop this project, or run the example you can use project.tcl to create a Vivado project. The project can be created without an example which is useful for packiaging the it as an ip core.

vivado -mode tcl -source project.tcl -tclargs 1 #With example

vivado -mode tcl -source project.tcl -tclargs 0 #No example

Clocking
This core requires two 50 MHz clocks. The phy is driven by clk_phy over the eth_clkin port. This should be at a 45 degree phase to clk_mac which drives all logic in the mac. All user logic should be synchronous to clk_mac.

Reset
The phy takes a rst_n (active low) signal from the user. This resets the mac logic as well as the phy. The phy requires that rst_n be asserted for at least 100 microseconds.

RX
On the RX side there is a native mode and an AXI stream mode.

Native Interface
This is the default mode for the mac. It is enabled when the AXIS_INTERFACES parameter is set to 0.

rx_vld - Signals when the other signals are valid.
rx_dat - 8 bits of packet data.
rx_sof - Signals the start of the frame.
rx_eof - Signals the end of frame, rx_dat is not valid when this is asserted.
rx_err - Signals an error on receive, if asserted with rx_eof this indicates a crc check error.
AXI Stream Interface
This interface sit on top of the native interface. It is enabled when the AXIS_INTERFACES parameter is set to 1.

rx_axis_mac_tdata - 8 bits of packet data.
rx_axis_mac_tvalid - Signals when the other signals are valid.
rx_axis_mac_tlast - Signals the end of the frame, unlike the native interface there is valid data on eof.
rx_axis_mac_tuser - Signals if there was an error, or crc error on the last byte.
TX
On the TX side there is a native mode and an AXI stream mode.

Native Interface
This is the default mode for the mac. It is enabled when the AXIS_INTERFACES parameter is set to 0. All values should be held constant until tx_ack is asserted at which point the values should be toggled for the next clock cycle. Too start the frame assert tx_vld and tx_sof. The mac will begin transmitting the ethernet preamble and first byte of data. It will assert tx_ack whenever it is ready for the next byte which must be provided on the next clock cycle. When tx_vld and tx_eof is asserted the mac will begin sending the crc code.

tx_vld - Signals when the other signals are valid.
tx_dat - 8 bits of packet data.
tx_sof - Signals the start of the frame.
tx_eof - Signals the end of frame, tx_dat is not used when this is asserted.
tx_ack - Signals that the above signals have been consumed and should be toggled.
AXI Stream Interface
This interface sit on top of the native interface. It is enabled when the AXIS_INTERFACES parameter is set to 1.

tx_axis_mac_tdata - 8 bits of packet data.
tx_axis_mac_tvalid - Signals when there is valid data.
tx_axis_mac_tlast - Signals the end of the frame, unlike the native interface tdata is consumed on the eof
tx_axis_mac_tready - Signals when the core is ready for the next byte. Data must be provided during a transmit when this is asserted.
SMI / MDIO
This core provides the user read/write access to the control/status registers on the phy. It also periodically checks the values in register[1] and uses it to update the speed_100, full_duplex, link_up, remote_fault, and auto_neg_done outputs. When it is not checking that register it allows the user to insert their own reads and writes. Note that this core assumes the phy is at address 1 which is the default on the Nexys 4 DDR board.

reg_vld - Indicates that the input signals are valid.
reg_addr - Address of the register.
reg_write - Assert this for a write operation or de-assert it for a read.
reg_wval - 16 bit value being written, ignored on read.
reg_rval - 16 bit value read from the register, valid on reg_ack, ignored on write.
reg_ack - Signals that the register operation is complete. All input signals should be held constant until reg_ack is asserted.
Example
Right now the example project is very simple. This is what I used for testing the core, and demonstrates some of its basic functionality. As of right now it allows you to see incoming packets using chipscope, broadcast a packet by pressing the up button on your board, and reads the value on the control register chosen using the switches and display it on the leds.

TODO / Limitations
Support 10 Mbps Mode
Currently this mac only supports 100 Mbps transmit and receive. I was unable to force it into 10 Mbps mode for testing using the registers so I never implemented it.

Support Half Duplex Mode
Currently only supports full duplex transmit and receive. Yet again I could not test this so I didn't implement it.

Fix mode configuration straps
The mode configuration straps which are supposed to be latched on the positive edge of reset signal to the phy do not seem to have any effect. I am not sure why.
