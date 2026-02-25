AutoSec — Demo Runbook (ICSim + Attacks + IDS Bridge)

A repeatable before vs after demo for AutoSec:
	•	Demo A (No protection): attacks reach the “car” (ICSim) and the UI becomes unstable.
	•	Demo B (Protection ON): the IDS bridge acts like a gateway filter; suspicious traffic is blocked before it reaches the “car”.

⸻

What you need
	•	Linux VM / Ubuntu with SocketCAN
	•	can-utils installed (candump, cansend, cangen, cansniffer)
	•	ICSim built (icsim, controls)
	•	AutoSec repo in ~/autosEc
	•	Python venv at ~/autosEc/.venv with python-can

⸻

Quick demo in a nutshell (before vs after)

Demo A — No protection
	1.	Run Terminal 1 (bring up buses)
	2.	Run Terminal 2 (start ICSim)
	3.	Do not run the IDS bridge
	4.	Run an attack (Terminal 4) → sim goes crazy

Demo B — Protection ON
	1.	Run Terminal 1
	2.	Run Terminal 2
	3.	Run Terminal 3 (calibrate + start bridge in protect mode)
	4.	Run the same attack (Terminal 4) → you should see:
	•	IDS alerts with action=block
	•	vcan1 stays cleaner than vcan0
	•	UI/sim remains more stable

⸻

Terminal map (recommended)

Open 5 terminals (Terminal 5 is optional but great for screenshots).
	•	Terminal 1: bring up vcan0 (attacker side) + vcan1 (protected side)
	•	Terminal 2: run ICSim + Controls (the “car” lives on vcan1)
	•	Terminal 3: activate venv + calibrate + run IDS bridge (protect mode)
	•	Terminal 4: run attacks (against vcan0)
	•	Terminal 5: observer (candump / cansniffer) to prove the difference

⸻

Step-by-step runbook

Terminal 1 — Create and enable both virtual CAN buses

sudo modprobe can can_raw vcan

# Clean up from previous runs
sudo ip link del vcan0 2>/dev/null || true
sudo ip link del vcan1 2>/dev/null || true

# Create buses
sudo ip link add dev vcan0 type vcan
sudo ip link add dev vcan1 type vcan

# Bring them up
sudo ip link set up vcan0
sudo ip link set up vcan1

# Verify
ip link show vcan0
ip link show vcan1

✅ Expected: both show UP and link/can.

⸻

Terminal 2 — Start the car simulation (ICSim)

Important: In the “protected” architecture, the car runs on vcan1.

From anywhere:

start_sim

✅ Expected: the IC Simulator + CANBus Control Panel windows appear.

⸻

Terminal 3 — Activate venv, calibrate baseline, start protection bridge

1) Activate Python environment

source ~/autosEc/.venv/bin/activate

2) Baseline calibration (learn normal IDs + timing)

Let ICSim run normally for ~60 seconds, then:

python3 ~/autosEc/code/ids/bridge.py calibrate --src vcan1 --seconds 60

✅ Expected: calib.json written with ~36 IDs (ICSim baseline).

3) Start the “gateway” in protection mode (leave running)

python3 ~/autosEc/code/ids/bridge.py bridge --mode protect --src vcan0 --dst vcan1

✅ Leave this running. This is your before-it-hits-the-car filter.

⸻

Terminal 4 — Run attacks (attacker side)

Attacks always target vcan0.

Flood / DoS

~/autosEc/code/attacks/flood.sh

Stop with:

CTRL + C

Spoof (example)

~/autosEc/code/attacks/spoof_speed_244.sh

Stop with:

CTRL + C


⸻

Terminal 5 (Optional) — Observer / proof terminal

This terminal is purely for screenshots and proof.

Live view (changes highlighted)

sudo cansniffer -c vcan1

Quick snapshot comparison

candump -n 30 vcan0
candump -n 30 vcan1

✅ In protect mode, vcan0 will look noisy during attacks, while vcan1 stays closer to baseline.

⸻

Demo script (what to say + what to do)

Demo A — No protection
	1.	Terminal 1: bring up buses
	2.	Terminal 2: start sim
	3.	Terminal 4: run flood.sh
	4.	Point out: UI becomes unstable / traffic spikes

Demo B — Protection ON
	1.	Terminal 3: calibrate
	2.	Terminal 3: start bridge in --mode protect
	3.	Terminal 4: run the same flood.sh
	4.	Point out:
	•	Terminal 3 prints alerts with action=block
	•	vcan1 stays cleaner than vcan0
	•	sim remains more stable

⸻

Common issues (fast fixes)

“vcan0/vcan1: No such device”

You forgot Terminal 1. Re-run Terminal 1 setup commands.

start_sim shows “dev duplicate / vcan1 is garbage”

That means vcan1 already exists. It’s usually safe to ignore, but the clean fix is:

sudo ip link del vcan0 2>/dev/null || true
sudo ip link del vcan1 2>/dev/null || true

…and then recreate them (Terminal 1).

bridge command says only {calibrate, bridge} exists

That’s correct — use:

python3 ~/autosEc/code/ids/bridge.py bridge --mode protect --src vcan0 --dst vcan1

(not protect as a separate subcommand).

⸻

Clean shutdown (end of demo)
	1.	Stop attack scripts (CTRL + C in Terminal 4)
	2.	Stop bridge (CTRL + C in Terminal 3)
	3.	Stop simulation:

stop_sim

	4.	Optional: remove buses

sudo ip link del vcan0 2>/dev/null || true
sudo ip link del vcan1 2>/dev/null || true


⸻

What to screenshot for reports / supervisor updates
	•	ICSim + Control Panel windows visible
	•	Terminal 3 showing [ALERT] ... action=block
	•	Terminal 5 showing candump vcan0 noisy vs candump vcan1 calmer
