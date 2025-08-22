# Vending-Machine-using-Verilog

This repo shall discuss the design and implementation of vending machine in AMD Vivado using Verilog.

The proposed vending machine has 3 products: PrA, PrB, PrC. The price of PrA, PrB, PrC are Rs 5, Rs 10, Rs 20 respectively. The vending machine accepts Rs 5 and Rs 10 denominations. It shall provide change if higher denomination is given. It also returns money if user presses cancel button during the operation.

The design of vending machine follows FSM approach.

sel = 00 implies Pr A

sel = 01 implies Pr B

sel = 10 implies Pr C

coin = 01 implies Rs 5

coin = 10 implies Rs 10

<img width="940" height="471" alt="image" src="https://github.com/user-attachments/assets/6d48472c-7317-44a7-bd0d-4c59d3bdbf81" />

Verilog Code: 

module vending_machine(input clk,reset,cancel, input [1:0]coin,sel, output reg PrA, PrB, PrC, change);

// State Encoding using Parameter

parameter S0=3'b000, S5=3'b001, S10=3'b010, S15=3'b011, S20=3'b100;

reg [2:0]current_state,next_state;

// Sequential logic for state transition

always@(posedge clk or posedge reset)begin

if (reset)

  current_state <= S0; //reset to initial state
  
else

  current_state <= next_state;
  
end 

// Next State Logic (Combinational)

always@(*)begin

 case(current_state)
 
    S0: begin
    
       if (coin == 2'b01) next_state = S5;
       
       else if (coin == 2'b10) next_state = S10;
       
       else next_state = S0;
       
       end 
    
    S5: begin
    
       if (coin == 2'b01) next_state = S10;
       
       else if (coin == 2'b10) next_state = S15;
       
       else if (cancel) next_state = S0;
       
       else next_state = S5;
       
       end 
       
    S10: begin
    
       if (coin == 2'b01) next_state = S15;
       
       else if (coin == 2'b10) next_state = S20;
       
       else if (cancel) next_state = S0;

       else next_state = S10;
       
       end 
       
    S15: begin
    
       if (coin == 2'b01) next_state = S20;
       
       else if (cancel) next_state = S0;
       
       else next_state = S15;
       
       end 

    S20: begin
    
       if (cancel) next_state = S0;
       
       else next_state = S20;
       
       end 
       
    default: next_state = S0; 
 
 endcase
 
end 

//Output Logic

always @ (posedge clk or posedge reset) begin

  if (reset) begin
  
     PrA = 0;

     PrB = 0;
     
     PrC = 0;
     
     change <= 0;
     
  end 
  
  else begin
  
   //  Default values (avoid latches)
   
     PrA = 0;
     
     PrB = 0;
     
     PrC = 0;
     
     change <= 0;
     
    case (current_state)
    
      S5: begin
      
          if (sel == 2'b00) begin
          
           PrA <= 1;
           
           change <=0;
           
          end
          
         end
         
      S10: begin
      
          if (sel == 2'b00) begin
          
           PrA <= 1;
           
           change <= 1;
           
          end
          
          else if (sel == 2'b01) begin
          
           PrB <= 1;
           
           change <= 0;
           
         end
         
        end 
        
      S15: begin
      
          if (sel == 2'b01) begin
          
           PrB <= 1;
           
           change <=1;
           
          end
          
         end
   
      S20: begin
      
          if (sel == 2'b00) begin
          
           PrA <= 1;
           
           change <=1;
           
          end
          
          else if (sel == 2'b01) begin
          
           PrB <= 1;
           
           change <= 1;
           
           end
           
          else if (sel == 2'b10) begin
          
           PrC <= 1;
           
           change <= 0;
           
           end
           
          end
          
       endcase  
       
   if (cancel)
   
     change <= 1;
     
   end
   
  end   
  
endmodule


Testbench Code:

module vending_machine_tb();

// inputs 

reg clk, reset, cancel;
reg [1:0] coin,sel;

// outputs

wire PrA,PrB,PrC,change;

vending_machine uut (.clk(clk), .reset(reset), .cancel(cancel), .coin(coin), .sel(sel), .PrA(PrA), .PrB(PrB), .PrC(PrC), .change(change));

always #5 clk = ~clk;

// Test Procedure

initial begin

 //initialise inputs
clk = 0;
reset = 1;
cancel = 0;
coin = 2'b00;
sel = 2'b00;

// hold reset for 20ns

#20 reset = 0;

// Testcase 1: Insert Rs. 5 & Buy Product A
#10 coin = 2'b01;
#10 sel = 2'b00;
#10 coin = 2'b00;
#20;

// Testcase 2: Insert Rs. 10 & Buy Product B

#10 coin = 2'b10;
#10 sel = 2'b01;
#20;

// Testcase 3: Insert Rs. 5 twice

#10 coin = 2'b01;
#10 coin = 2'b01;
#10 sel = 2'b00;
#20;

// Testcase 4: Insert Rs. 20 & Buy Product C

#10 coin = 2'b10;
#10 coin = 2'b10;
#10 sel = 2'b10;
#20;

#50 $finish;

end 

initial begin

$monitor ("Time = %d, coin = %b, sel = %b, cancel = %b, PrA = %b, PrB = %b, PrC = %b, change = %b",
          $time, coin, sel, cancel, PrA, PrB, PrC, change);
          
end

Schematic after RTL Analysis:

<img width="940" height="473" alt="image" src="https://github.com/user-attachments/assets/35409625-db8b-4002-b85f-7242ac6d4101" />

Schematic after Synthesis:

<img width="940" height="273" alt="image" src="https://github.com/user-attachments/assets/6d801f3f-1af6-4388-aec4-d1caba825281" />

Output after Simulation:

<img width="938" height="220" alt="image" src="https://github.com/user-attachments/assets/516b1f6d-eb39-426c-8610-24e7096e472a" />

This Verilog code was dumped virtually on FPGA Board: Kria K26C SOM and during execution, 5 LUTs, 4 registers, 10 IOBs and 1 clk buffer was utilised.
