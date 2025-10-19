## sky130 CMOS Inverter Characterization Report 

This report documents the characterization of a CMOS inverter using a SPICE simulator(ngspice), following the structure and experiments typical of the sky130 circuit design workflow. The analysis covers the static and transient behavior of the MOSFETs and the resulting inverter circuit.

1. **Introduction / Background**
    This characterization study consists of five main experiments, each designed to extract critical parameters for digital circuit design and static timing analysis (STA):


    a.MOSFET IDS  vs VDS Characteristics: To observe the fundamental behavior of the NMOS transistor, specifically identifying the linear and saturation regions of operation, which are crucial for modeling the transistor's effective resistance and drive strength in a logic gate.


    b.Threshold Voltage (Vt) Extraction: To determine the Threshold Voltage (VT) of the NMOS transistor by plotting ID  vs VGS. VT defines the gate voltage required to turn the transistor ON, which directly impacts the switching characteristics of the inverter.


    c.CMOS Inverter Voltage Transfer Characteristic (VTC): To plot the static behavior of the inverter, VOUT vs VIN. This plot is used to determine the key DC parameters: switching threshold VM, logic levels (VOL, VOH), and noise margins (NML, NML).


    d.Transient Behavior: Propagation Delays: To measure the dynamic speed of the inverter by applying a pulse input and extracting the rise and fall propagation delays. These delays are the fundamental for library characterization and STA.


    e.Variation Studies (Voltage and Device): To evaluate the robustness of the inverter's static characteristics against changes in supply voltage (VDD) and device sizing (W/L). This analysis is critical for understanding circuit reliability across process, voltage, and temperature (PVT) corners.

2. **Spice Netlist**

   1. Resistive Load NMOS DC Analysis
This netlist simulates a single NMOS transistor with a 55 Ohm resistive pull-up, often used for introductory characterization or simple logic gates.

        * Title: Resistive Load NMOS DC Analysis (W=0.39u, L=0.15u)

            * Model Description
            .param temp=27
            
            * Including sky130 library files
            .lib "sky130_fd_pr/models/sky130.lib.spice" tt
            
            * Netlist Description
            * M1: Drain=Vdd, Gate=n1, Source=0, Bulk=0
            XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39u l=0.15u
            
            R1 n1 in 55
            
            Vdd vdd 0 1.8V
            * The input voltage 'in' is fixed for this .op, but the sweep below is active
            Vin in 0 1.8V
            
            * Simulation Commands
            .op
            .dc Vin 0 1.8 0.1
            
            .control
                run
                plot v(n1) vs v(in) title "Resistive Load NMOS VTC"
            .endc
            
            .end
            
