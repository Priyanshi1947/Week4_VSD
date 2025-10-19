## sky130 CMOS Inverter Characterization Report 

This report documents the characterization of a CMOS inverter using a SPICE simulator(ngspice), following the structure and experiments typical of the sky130 circuit design workflow. The analysis covers the static and transient behavior of the MOSFETs and the resulting inverter circuit.

1. **Introduction / Background**
    This characterization study consists of five main experiments, each designed to extract critical parameters for digital circuit design and static timing analysis (STA):


        MOSFET IDS  vs VDS Characteristics: To observe the fundamental behavior of the NMOS transistor, specifically identifying the linear and saturation regions of operation, which are crucial for modeling the transistor's effective resistance and drive strength in a logic gate.


        Threshold Voltage (Vt) Extraction: To determine the Threshold Voltage (VT) of the NMOS transistor by plotting ID  vs VGS. VT defines the gate voltage required to turn the transistor ON, which directly impacts the switching characteristics of the inverter.


        CMOS Inverter Voltage Transfer Characteristic (VTC): To plot the static behavior of the inverter, VOUT vs VIN. This plot is used to determine the key DC parameters: switching threshold VM, logic levels (VOL, VOH), and noise margins (NML, NML).


        Transient Behavior: Propagation Delays: To measure the dynamic speed of the inverter by applying a pulse input and extracting the rise and fall propagation delays. These delays are the fundamental for library characterization and STA.


        Variation Studies (Voltage and Device): To evaluate the robustness of the inverter's static characteristics against changes in supply voltage (VDD) and device sizing (W/L). This analysis is critical for understanding circuit reliability across process, voltage, and temperature (PVT) corners.
