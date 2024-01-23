# pypsa-help
BESS not found in state_of_charge DataFrame
here is my code
import pypsa
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Initialize PyPSA network
network = pypsa.Network()

# Define time snapshots for a day (24 hours)
snapshots = pd.date_range('2024-01-01 00:00', periods=24, freq='H')
network.set_snapshots(snapshots)

# Add HV and LV buses
network.add("Bus", "HV Bus", v_nom=10)  # High voltage bus (10 kV)
network.add("Bus", "LV Bus", v_nom=0.4)  # Low voltage bus (0.4 kV)

# Transformer setup
S_nom_transformer = 20e6  # Transformer capacity (20 MVA)
network.add("Transformer",
            "Transformer HV-LV",
            bus0="HV Bus", bus1="LV Bus", 
            x=0.05,
            s_nom=S_nom_transformer)

# Add slack generator at HV bus
network.add("Generator", 
            "Slack Generator",
            bus="HV Bus",
            p_nom=30e6, 
            control="Slack")  # Nominal power of 30 MW

# Time varying load
base_load = 10e6  # Base load of 10 MW
peak_load = 18e6  # Peak load of 18 MW
load_profile = np.full(24, base_load)
load_profile[17:21] = peak_load  # Increased load during peak hours
load_series = pd.Series(load_profile, index=snapshots)

# Add load to LV bus
network.add("Load", "Load 1", bus="LV Bus", p_set=load_series)

# Add a 1 MW BESS
network.add("StorageUnit", 
            "BESS", bus="LV Bus", 
            p_nom=1e6, state_of_charge_initial=0.5, 
            p_max_pu=1, p_min_pu=-1, 
            max_hours=1, cyclic=True)

# Assign operational parameters for BESS
bess_schedule = pd.Series([-1 if 0 <= snap.hour < 6 else 0 for snap in snapshots], index=snapshots)
network.storage_units_t.p_set['BESS'] = bess_schedule

# Run power flow calculation
network.pf()

# Plot voltage at LV Bus in per unit
voltages_pu = network.buses_t.v_mag_pu.loc[:, "LV Bus"]
plt.figure(figsize=(12, 6))
plt.plot(snapshots, voltages_pu, marker='o', color='b')
plt.title('Voltage Profile at LV Bus Over Time (Per Unit)')
plt.xlabel('Time')
plt.ylabel('Voltage (p.u.)')
plt.xticks(rotation=45)
plt.grid(True)
plt.axvspan('2024-01-01 17:00', '2024-01-01 21:00', color='red', alpha=0.3)
plt.show()

# Check if BESS state of charge is available
if 'BESS' in network.storage_units_t.state_of_charge.columns:
    soc_bess = network.storage_units_t.state_of_charge['BESS']
    plt.figure(figsize=(12, 6))
    plt.plot(snapshots, soc_bess, marker='o', color='g')
    plt.title('State of Charge of BESS Over Time')
    plt.xlabel('Time')
    plt.ylabel('State of Charge (%)')
    plt.xticks(rotation=45)
    plt.grid(True)
    plt.show()
else:
    print("BESS not found in state_of_charge DataFrame. Please check the setup.")


    when i run the code i gget the folowing 
    runfile('C:/Users/m/Documents/Python Scripts/bess+weak.py', wdir='C:/Users/m/Documents/Python Scripts')
c:\users\m\documents\python scripts\bess+weak.py:16: FutureWarning:

'H' is deprecated and will be removed in a future version, please use 'h' instead.

WARNING:pypsa.components:StorageUnit has no attribute cyclic, ignoring this passed value.
INFO:pypsa.pf:Performing non-linear load-flow on AC sub-network SubNetwork 0 for snapshots DatetimeIndex(['2024-01-01 00:00:00', '2024-01-01 01:00:00',
               '2024-01-01 02:00:00', '2024-01-01 03:00:00',
               '2024-01-01 04:00:00', '2024-01-01 05:00:00',
               '2024-01-01 06:00:00', '2024-01-01 07:00:00',
               '2024-01-01 08:00:00', '2024-01-01 09:00:00',
               '2024-01-01 10:00:00', '2024-01-01 11:00:00',
               '2024-01-01 12:00:00', '2024-01-01 13:00:00',
               '2024-01-01 14:00:00', '2024-01-01 15:00:00',
               '2024-01-01 16:00:00', '2024-01-01 17:00:00',
               '2024-01-01 18:00:00', '2024-01-01 19:00:00',
               '2024-01-01 20:00:00', '2024-01-01 21:00:00',
               '2024-01-01 22:00:00', '2024-01-01 23:00:00'],
              dtype='datetime64[ns]', name='snapshot', freq='h')
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.016064 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031267 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031291 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.032735 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.014537 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.015825 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031191 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.017087 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.015941 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031078 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031963 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.015640 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.015649 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031233 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031619 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031271 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031244 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.016094 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031516 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031004 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031682 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031492 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.031239 seconds
INFO:pypsa.pf:Newton-Raphson solved in 4 iterations with error of 0.000000 in 0.015009 seconds
BESS not found in state_of_charge DataFrame. Please check the setup.
