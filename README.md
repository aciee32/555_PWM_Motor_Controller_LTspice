555 Timer PWM Motor Controller (LTspice Simulation)

Hey there! This project is a fully functional Pulse Width Modulation (PWM) DC motor controller designed around the classic NE555 timer and simulated in LTspice.

I built this to demonstrate how to efficiently control the speed of a DC motor without wasting power as heat (which is what happens if you just use a standard variable linear resistor).

What's Inside:
555_motor_controller.asc: The complete LTspice schematic file.

images/: Screenshots of the circuit layout and verified working waveforms.

How the Circuit Works:
Instead of changing the voltage level to alter motor speed, this circuit uses PWM. It chops a steady 12V supply into high-frequency pulses. By changing the ratio of "ON" time to "OFF" time (the Duty Cycle), we control the average power delivered to the motor.

1. The 555 "Brain"
The NE555 timer is configured as an astable multivibrator. Usually, a 555 has a fixed duty cycle. To bypass this limitation, I added steering diodes (D1 & D2) around a potentiometer (U2).

Charging Path: Current flows through D1 and the left side of the pot to charge the timing capacitor (C1).

Discharging Path: Current discharges from C1 through the right side of the pot and D2 back into Pin 7 (DIS).

This lets us smoothly adjust the PWM duty cycle from roughly 1% to 99% by simply twisting the pot, while keeping the overall switching frequency stable.

2. The Motor Model
Since LTspice doesn't have a spinny 3D motor component, a real DC motor is modeled using its electrical equivalents:

L1 (1mH): Represents the inductive motor windings.

R1 (5Ω): Represents the internal DC wire resistance of the coils.

V2 (8V): Simulates the Back-EMF (the voltage the motor naturally generates as it physically spins up).

3. The Power Stage & Protection
An IRFZ44N N-Channel MOSFET acts as our heavy-duty electronic switch. Pin 3 of the 555 drives the Gate. When Pin 3 is High, the MOSFET opens up and grounds the motor, letting current flow.

Crucially, D3 (1N4001) is placed across the motor as a Flyback Diode. When the MOSFET suddenly switches off, the inductive magnetic field inside the motor collapses, causing a massive voltage spike. D3 safely recirculates this current, saving the MOSFET from getting fried.

📊 Verified Simulation Waveforms
When you run the simulation (.tran 50m), you get beautiful proof that the motor is running:

V(n007) (Green) - Pin 3 Output: A perfect 0V to 12V square wave commanding the circuit.

V(n005) (Blue) - MOSFET Drain: Shows the switching voltages along with high-frequency ringing spikes, showing the flyback diode clamping down on inductive kickback.

I(L1) (Red) - Motor Current: The ultimate proof of spinning. It forms a perfect sawtooth wave. Current ramps up smoothly when the pulse is high, and coasts down when the pulse is low. Because the current never hits zero, the motor runs continuously and smoothly!

🛠️ Lessons Learned & Debugging Notes (The Hard Way)
Simulators are unforgiving! During development, I hit a few classic electronic simulation hurdles that are worth noting if you are trying to reproduce this:

Watch Your Node Crossings: In LTspice, wires crossing over each other do not connect automatically. Pins 2 and 6 must explicitly share a blue junction dot with the top of Capacitor C1. Conversely, ensure Pin 7 only touches the diode/pot junction and doesn't accidentally bridge into the capacitor line, or it will short out the timing cycle completely.

Potentiometer Subcircuit Syntax: Custom potentiometer models in LTspice can be finicky. The SPICE directive requires explicit curly braces {} to correctly calculate the parameter division. The syntax used here is:

Code snippet
.subckt pot 1 2 3
.param w=limit(wiper,1m,.999)
R1 1 2 {Rtot*(1-w)}
R2 2 3 {Rtot*w}
.ends
Run It Yourself!
Clone this repository.

Open 555_motor_controller.asc in LTspice.

Click the Running Man icon to start the transient analysis.

Right-click the potentiometer text (wiper=0.5) and change it to 0.2 (low speed) or 0.8 (high speed) to observe the duty cycle changes!
