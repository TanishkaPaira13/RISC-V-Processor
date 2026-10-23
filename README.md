RISC-V single-cycle CPU
============================================

riscv.v
  Verilog-2001 RTL for the single-cycle RISC-V subset from the supplied PDF.
  Includes the completed mux3 and ALU modules because the PDF did not show a
  complete ALU definition.

testbench.v
  Verilog-2001 testbench. It checks for the expected store of decimal 71 at
  address 84 and includes a timeout.

memfile.dat
  RISC-V instruction words for the program shown in the PDF. The instruction
  memory loads this file with $readmemh, so keep it in the simulator's working
  directory.

provided_image_words.mem
  Separate transcription of the words from the other supplied image. These
  appear to be MIPS instruction encodings and are not used by this RISC-V CPU.

Run with Icarus Verilog (if installed):
  iverilog -g2005 -s testbench -o simv riscv.v testbench.v
  vvp simv

Review notes:
- Converted SystemVerilog constructs (logic, always_comb, always_ff) to
  Verilog-2001 declarations and always blocks.
- Corrected data-memory indexing to a[7:2], the six-bit word index required
  for a 64-word memory. The previous source used a[31:2], which could index
  outside the declared array in simulation.
- This is a single-cycle datapath with five conceptual stages, not a pipelined
  processor.
- The code was reviewed for Verilog-2001 syntax and signal/module consistency,
  but no HDL simulator was available in the environment; compile/simulation
  success has therefore not been independently confirmed here.
