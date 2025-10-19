<img width="505" height="482" alt="image" src="https://github.com/user-attachments/assets/130ebb70-ab95-4147-8bf9-4679e258103e" />## sky130 CMOS Inverter Characterization Report 

This report documents the characterization of a CMOS inverter using a SPICE simulator(ngspice), following the structure and experiments typical of the sky130 circuit design workflow. The analysis covers the static and transient behavior of the MOSFETs and the resulting inverter circuit.

1. **Introduction / Background**
    This characterization study consists of five main experiments, each designed to extract critical parameters for digital circuit design and static timing analysis (STA):


    ## a.MOSFET IDS  vs VDS Characteristics:
       To observe the fundamental behavior of the NMOS transistor, specifically identifying the linear and saturation regions of operation, which are crucial for modeling the transistor's effective resistance and drive strength in a logic gate.


    ## b.Threshold Voltage (Vt) Extraction:
       To determine the Threshold Voltage (VT) of the NMOS transistor by plotting ID  vs VGS. VT defines the gate voltage required to turn the transistor ON, which directly impacts the switching characteristics of the inverter.


    ## c.CMOS Inverter Voltage Transfer Characteristic (VTC):
        To plot the static behavior of the inverter, VOUT vs VIN. This plot is used to determine the key DC parameters: switching threshold VM, logic levels (VOL, VOH), and noise margins (NML, NML).


    ## d.Transient Behavior:
       Propagation Delays: To measure the dynamic speed of the inverter by applying a pulse input and extracting the rise and fall propagation delays. These delays are the fundamental for library characterization and STA.


    ## e.Variation Studies (Voltage and Device):

       To evaluate the robustness of the inverter's static characteristics against changes in supply voltage (VDD) and device sizing (W/L). This analysis is critical for understanding circuit reliability across process, voltage, and temperature (PVT) corners.

3. **Spice Netlist**

   1.Resistive Load NMOS DC Analysis
      
This netlist simulates a single NMOS transistor with a 55 Ohm resistive pull-up, often used for introductory characterization or simple logic gates.

          Title: Resistive Load NMOS DC Analysis (W=0.39u, L=0.15u)

          Model Description
            .param temp=27
            
            Including sky130 library files
            .lib "sky130_fd_pr/models/sky130.lib.spice" tt
            
            Netlist Description
            M1: Drain=Vdd, Gate=n1, Source=0, Bulk=0
            XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39u l=0.15u
            
            R1 n1 in 55
            
            Vdd vdd 0 1.8V
            The input voltage 'in' is fixed for this .op, but the sweep below is active
            Vin in 0 1.8V
            
            Simulation Commands
            .op
            .dc Vin 0 1.8 0.1
            
            .control
                run
                plot v(n1) vs v(in) title "Resistive Load NMOS VTC"
            .endc
            
            .end

   2. Inverter Transient Delay Analysis
   
   This netlist simulates the dynamic behavior of the nominal inverter to measure propagation delays ($t_{pLH}$, $t_{pHL}$).

           Title: CMOS Inverter Transient Delay Analysis (Nominal Sizing)

            Model Description
            .param temp=27

            Including sky130 library files
            .lib "sky130_fd_pr/models/sky130.lib.spice" tt

            Netlist Description
            M1 (PMOS): Drain=out, Gate=in, Source=vdd, Bulk=vdd
            XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84u l=0.15u
            M2 (NMOS): Drain=out, Gate=in, Source=0, Bulk=0
            XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36u l=0.15u

            Vdd vdd 0 1.8V
            Input pulse for transient analysis
            Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

            Simulation Commands
            .tran 0.1ns 10ns

            .control
                run
                plot v(in) v(out) title "CMOS Inverter Transient Response"
            .endc
            
            .end

 3. Inverter VTC Nominal Sizing
    
    This netlist determines the Voltage Transfer Characteristic (VTC) for the nominal $1.8\text{V}$ CMOS inverter.

            Title: CMOS Inverter VTC Nominal Sizing (Wp=0.84/Wn=0.36)
            Model Description
            .param temp=27
            
            Including sky130 library files
            .lib "sky130_fd_pr/models/sky130.lib.spice" tt
            
            Netlist Description
            XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84u l=0.15u
            XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36u l=0.15u
            
            Cload out 0 50fF
            
            Vdd vdd 0 1.8V
            Vin in 0 0V  * Initial value for DC sweep
            
            Simulation Commands
            .op
            .dc Vin 0 1.8 0.01
            
            .control
                run
                setplot dc1
                plot v(out) vs v(in) title "Inverter VTC (Nominal)"
            .endc
            
            .end

 4. Inverter VTC Variation (Stronger PMOS)
    
    This netlist simulates the VTC with a slightly stronger PMOS ($W_P=1\mu$ vs $0.84\mu$ nominal) to observe the effect of device variation on the switching threshold ($V_M$).

            Title: Inverter VTC Variation (Wp=1u, Shifted Vm)
            
            Model Description
            .param temp=27
            
            Including sky130 library files
            .lib "sky130_fd_pr/models/sky130.lib.spice" tt
            
            Netlist Description
            PMOS is wider than nominal
            XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1u l=0.15u
            XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36u l=0.15u
            
            Cload out 0 50fF
            
            Vdd vdd 0 1.8V
            Vin in 0 0V
            
            Simulation Commands
            .op
            .dc Vin 0 1.8 0.01
            
            .control
                run
                setplot dc1
                plot v(out) vs v(in) title "Inverter VTC (Stronger PMOS)"
            .endc
            
            .end


 5. Inverter VTC Variation (High Ratio)
    
    This netlist uses a large PMOS to NMOS ratio, demonstrating a highly skewed inverter design.

        Title: Inverter VTC Variation (Wp=7u, High Ratio)
        
        Model Description
        .param temp=27
        
        Including sky130 library files
        .lib "sky130_fd_pr/models/sky130.lib.spice" tt
        
        Netlist Description
        Highly skewed PMOS
        XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7u l=0.15u
        XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42u l=0.15u
        
        Cload out 0 50fF
        
        Vdd vdd 0 1.8V
        Vin in 0 0V
        
        Simulation Commands
        .op
        .dc Vin 0 1.8 0.01
        
        .control
            run
            setplot dc1
            plot v(out) vs v(in) title "Inverter VTC (High Skewed Ratio)"
        .endc
        
        .end

 6. Inverter VTC VDD Scaling
    
    This netlist uses a control loop to repeatedly run the VTC sweep while systematically decreasing the supply voltage ($V_{DD}$), allowing you to plot the effect of voltage variation on $V_M$ and noise margins.

    Title: Inverter VTC as a Function of Supply Voltage

        Model Description
        .param temp=27
        
        Including sky130 library files
        .lib "sky130_fd_pr/models/sky130.lib.spice" tt
        
        Netlist Description
        XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1u l=0.15u
        XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36u l=0.15u
        
        Cload out 0 50fF
        
        Vdd vdd 0 1.8V
        Vin in 0 0V
        
        Simulation Commands
        NOTE: .op is not necessary here as .dc does the heavy lifting.
        
        .control
            Initialize variables
            let powersupply = 1.8
            let voltagesupplyvariation = 0
        
            Outer loop to sweep VDD
            dowhile voltagesupplyvariation < 6
                alter Vdd = powersupply
                
                Inner loop (DC sweep)
                dc Vin 0 1.8 0.01
        
                Update variables for next iteration
                let powersupply = powersupply - 0.2
                let voltagesupplyvariation = voltagesupplyvariation + 1
            end
        
            Plot all 6 VTC curves from the different DC sweeps (dc1 to dc6)
            plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in \
                xlabel "Input Voltage (V)" ylabel "Output Voltage (V)" title "Inverter DC Characteristics as a Function of Supply Voltage"
        
        .endc
        
        .end

3. **Plots and Figures**

   
## Experiment 1:
    The plot shows drain current Id vs. drain-source voltage Vds for multiple gate-source voltages Vgs.

  ![]((https://github.com/Priyanshi1947/Week4_VSD/blob/main/image.png?raw=true))
    
    Observation: For a fixed Vgs, the current initially increases linearly with Vds (Linear Region) before flattening out (Saturation Region).
    Significance: The saturation region is where the transistor acts as a current source, and the linear region is where it acts as a resistor. The onset of saturation is near the knee of each curve.

   
## Experiment 2: 
    The plot shows drain current Id vs. gate-source voltage Vgs at a fixed high Vds .

   ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/image%20(1).png?raw=true)
   
    Annotation: The Threshold Voltage (Vt) is extracted by finding the Vgs point at which the current starts to rise significantly (or by linear extrapolation from the saturation region).

   
## Experiment 3: VTC (Voltage Transfer Characteristic)
    The VTC plots the output voltage Vout vs. input voltage Vin.

   ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/image%20(2).png?raw=true)

   ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/image%20(4).png?raw=true)
   
    Annotation: The sharp transition demonstrates high gain. The Switching Threshold Vm is the point where Vin=Vout. Vm is observed to be around 0.9V for the given design.

    Noise Margin Points: The points VIL (Maximum low input) and VIH (Minimum high input) define the region of safe low and high logic inputs.


## Experiment 4: Transient Waveforms
   
    The plot shows the input pulse (blue) and the resulting output transient (red).

   ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/image%20(3).png?raw=true)
   
    Annotation: Propagation delays are measured between the input and output Vdd/2 crossing points.

   
## Experiment 5: VTC under Vdd Variation
   
    The plot shows VTC curves for different supply voltages Vdd.

   
    ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/image%20(4).png?raw=true)
  
   
    Observation: As Vdd scales down, the VTC curve shifts to the left Vin and the overall voltage swing decreases.

   
        
4. **Tabulated results**

    ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/Screenshot%202025-10-19%20222308.png?raw=true)

    ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/Screenshot%202025-10-19%20222320.png?raw=true)

    ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/Screenshot%202025-10-19%20222328.png?raw=true)

    ![](https://github.com/Priyanshi1947/Week4_VSD/blob/main/Screenshot%202025-10-19%20222343.png?raw=true)


5. **Observations/Analysis**

The Id curves saturate at high Vds.
STA connection: The saturation current determines the maximum drive strength IDsat, which is critical for calculating delay. The delay model relies on this current saturation.

Threshold Voltage Extraction

The current begins to rise significantly only after Vgs exceeds approximately 0.5V.
Vt is the gate-source voltage required to create an inversion layer (channel) of mobile carriers necessary for conduction between the source and drain.
STA connection: Vt determines the delay sensitivity to Vgs and Vdd. A lower Vt increases leakage but generally lowers propagation delay by allowing the transistor to turn ON sooner.

CMOS Inverter VTC and Noise Margins

The rise time is approximately twice the fall time. Also, NMH is slightly larger than NML.
In a standard CMOS inverter, the PMOS transistor is typically wider than the NMOS transistor to compensate for the naturally lower mobility of holes (PMOS carriers) compared to electrons (NMOS carriers). However, the results here suggest the PMOS is stronger than the NMOS. This means the output discharges slower than it charges.
STA connection: Propagation delay is directly measured as the time taken to charge or discharge the output capacitance. The difference in rise and fall times is a direct measure of the asymmetry in drive strength between the pull-up and pull-down networks.

Variation Studies

The VTC shifts as Vdd changes. The DC gain initially increases as Vdd is lowered (1.8 to 1.2 V), but falls drastically at low voltages (1 V).
Vm is determined by the ratio of NMOS to PMOS currents, both of which are functions of Vdd. At very low , the supply voltage is not high enough to adequately drive the transistors, causing the current (and therefore the gain) to drop. Low voltage operation also results in high rise and fall times, and the device may not even operate correctly.
STA connection: $V_{DD}$ variation affects delay and noise margins. Lower $V_{DD}$ reduces power but increases delay ($\tau \propto C_L/I$) and decreases noise margins, which can lead to timing failures or unreliable operation.
Device variations due to manufacturing processes (like etching variations) cause the VTC to shift, changing Vm and the noise margins .
Process variations alter Vt and the effective W/L ratio. For instance, a stronger PMOS shifts the VTC to the left, resulting in a change in Vm.





