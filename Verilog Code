// Supported instructions: LW, SW, ADD, SUB, AND, OR, SLT, ADDI, BEQ, JAL.

module top(input clk, input reset, output [31:0] writedata, output [31:0] dataadr, output memwrite);
    wire [31:0] pc, instr, readdata;
    riscv cpu(clk, reset, pc, instr, memwrite, dataadr, writedata, readdata);
    imem instruction_memory(pc[7:2], instr);
    dmem data_memory(clk, memwrite, dataadr, writedata, readdata);
endmodule

module dmem(input clk, input we, input [31:0] a, input [31:0] wd, output [31:0] rd);
    reg [31:0] RAM [0:63];
    // Six-bit word index for a 64-word memory.
    assign rd = RAM[a[7:2]];
    always @(posedge clk)
        if (we) RAM[a[7:2]] <= wd;
endmodule

module imem(input [5:0] a, output [31:0] rd);
    reg [31:0] RAM [0:63];
    initial $readmemh("memfile.dat", RAM);
    assign rd = RAM[a];
endmodule

module riscv(input clk, input reset, output [31:0] pc, input [31:0] instr,
             output memwrite, output [31:0] aluresult, output [31:0] writedata,
             input [31:0] readdata);
    wire alusrc, regwrite, jump, pcsrc, zero;
    wire [1:0] memtoreg, immsrc;
    wire [2:0] alucontrol;
    controller c(instr[30], instr[14:12], instr[6:0], zero,
                 memtoreg, memwrite, pcsrc, alusrc, regwrite, jump, immsrc, alucontrol);
    datapath dp(clk, reset, memtoreg, pcsrc, alusrc, regwrite, immsrc, alucontrol,
                zero, pc, instr, aluresult, writedata, readdata);
endmodule

module controller(input f7b5, input [2:0] funct3, input [6:0] op, input zero,
                  output [1:0] memtoreg, output memwrite, output pcsrc,
                  output alusrc, output regwrite, output jump,
                  output [1:0] immsrc, output [2:0] alucontrol);
    wire [1:0] aluop;
    wire branch;
    maindec md(op, memtoreg, memwrite, branch, alusrc, regwrite, jump, immsrc, aluop);
    aludec ad(f7b5, op[5], funct3, aluop, alucontrol);
    assign pcsrc = (branch & zero) | jump;
endmodule

module maindec(input [6:0] op, output [1:0] memtoreg, output memwrite,
               output branch, output alusrc, output regwrite, output jump,
               output [1:0] immsrc, output [1:0] aluop);
    reg [10:0] controls;
    assign {regwrite, immsrc, alusrc, branch, memwrite, memtoreg, jump, aluop} = controls;
    always @(*) begin
        case (op)
            7'b0110011: controls = 11'b1_xx_0_0_0_00_0_10; // R-type ALU
            7'b0010011: controls = 11'b1_00_1_0_0_00_0_10; // ADDI
            7'b0000011: controls = 11'b1_00_1_0_0_01_0_00; // LW
            7'b0100011: controls = 11'b0_01_1_0_1_xx_0_00; // SW
            7'b1100011: controls = 11'b0_10_0_1_0_xx_0_01; // BEQ
            7'b1101111: controls = 11'b1_11_0_0_0_10_1_00; // JAL
            default:    controls = 11'bx_xx_x_x_x_xx_x_xx;
        endcase
    end
endmodule

module aludec(input f7b5, input op5, input [2:0] funct3, input [1:0] aluop,
              output reg [2:0] alucontrol);
    wire addSubType = f7b5 & op5;
    always @(*) begin
        case (aluop)
            2'b00: alucontrol = 3'b010; // ADD for loads/stores/JAL
            2'b01: alucontrol = 3'b110; // SUB for BEQ
            default: begin
                case ({addSubType, funct3})
                    4'b0000: alucontrol = 3'b010; // ADD / ADDI
                    4'b1000: alucontrol = 3'b110; // SUB
                    4'b0111: alucontrol = 3'b000; // AND
                    4'b0110: alucontrol = 3'b001; // OR
                    4'b0010: alucontrol = 3'b111; // SLT
                    default: alucontrol = 3'bxxx;
                endcase
            end
        endcase
    end
endmodule

module datapath(input clk, input reset, input [1:0] memtoreg, input pcsrc,
                input alusrc, input regwrite, input [1:0] immsrc,
                input [2:0] alucontrol, output zero, output [31:0] pc,
                input [31:0] instr, output [31:0] aluresult,
                output [31:0] writedata, input [31:0] readdata);
    wire [31:0] pcnext, pcplus4, pcbranch, immext, srca, srcb, result;
    flopr #(32) pcreg(clk, reset, pcnext, pc);
    adder pcadd4(pc, 32'd4, pcplus4);
    adder pcaddbranch(pc, immext, pcbranch);
    mux2 #(32) pcmux(pcplus4, pcbranch, pcsrc, pcnext);
    regfile rf(clk, regwrite, instr[19:15], instr[24:20], instr[11:7], result, srca, writedata);
    imext ext(instr[31:7], immsrc, immext);
    mux2 #(32) srcbmux(writedata, immext, alusrc, srcb);
    alu alu_unit(srca, srcb, alucontrol, aluresult, zero);
    mux3 #(32) resultmux(aluresult, readdata, pcplus4, memtoreg, result);
endmodule

module imext(input [31:7] instr, input [1:0] immsrc, output reg [31:0] immext);
    always @(*) begin
        case (immsrc)
            2'b00: immext = {{21{instr[31]}}, instr[30:20]};
            2'b01: immext = {{21{instr[31]}}, instr[30:25], instr[11:7]};
            2'b10: immext = {{20{instr[31]}}, instr[7], instr[30:25], instr[11:8], 1'b0};
            2'b11: immext = {{12{instr[31]}}, instr[19:12], instr[20], instr[30:21], 1'b0};
            default: immext = 32'bx;
        endcase
    end
endmodule

module regfile(input clk, input we3, input [4:0] ra1, input [4:0] ra2,
               input [4:0] wa3, input [31:0] wd3, output [31:0] rd1, output [31:0] rd2);
    reg [31:0] rf [0:31];
    always @(posedge clk)
        if (we3 && (wa3 != 5'd0)) rf[wa3] <= wd3;
    assign rd1 = (ra1 != 5'd0) ? rf[ra1] : 32'b0;
    assign rd2 = (ra2 != 5'd0) ? rf[ra2] : 32'b0;
endmodule

module adder(input [31:0] a, input [31:0] b, output [31:0] y);
    assign y = a + b;
endmodule

module flopr #(parameter WIDTH = 8)(input clk, input reset, input [WIDTH-1:0] d, output reg [WIDTH-1:0] q);
    always @(posedge clk or posedge reset)
        if (reset) q <= {WIDTH{1'b0}};
        else q <= d;
endmodule

module mux2 #(parameter WIDTH = 8)(input [WIDTH-1:0] d0, input [WIDTH-1:0] d1,
                                  input s, output [WIDTH-1:0] y);
    assign y = s ? d1 : d0;
endmodule

module mux3 #(parameter WIDTH = 8)(input [WIDTH-1:0] d0, input [WIDTH-1:0] d1,
                                  input [WIDTH-1:0] d2, input [1:0] s, output [WIDTH-1:0] y);
    assign y = s[1] ? d2 : (s[0] ? d1 : d0);
endmodule

module alu(input [31:0] a, input [31:0] b, input [2:0] alucontrol,
           output reg [31:0] result, output zero);
    always @(*) begin
        case (alucontrol)
            3'b000: result = a & b;
            3'b001: result = a | b;
            3'b010: result = a + b;
            3'b110: result = a - b;
            3'b111: result = ($signed(a) < $signed(b)) ? 32'd1 : 32'd0;
            default: result = 32'bx;
        endcase
    end
    assign zero = (result == 32'b0);
endmodule
