# AutoSec — Connected Vehicle Intrusion Detection System (IDS)

**AutoSec** is a virtual CAN-bus security testbed and intrusion detection gateway for connected vehicle environments. It demonstrates how malicious CAN traffic can destabilize a vehicle system **without protection**, and how a lightweight **IDS bridge (gateway filter)** can detect and block suspicious traffic **before it reaches the “car”**.

**Live demo UI (Protection Toggle):**  
👉 https://autosec.netlify.app/

---

## 🚗 What This Project Demonstrates

A repeatable **before vs after** security demo using ICSim + attack scripts + AutoSec IDS bridge:

- **Demo A (No Protection)**  
  Attacks go directly to the simulated vehicle → UI becomes unstable, CAN traffic spikes.

- **Demo B (Protection ON)**  
  IDS bridge acts as a gateway filter → suspicious frames are blocked before reaching the vehicle → simulation remains more stable.

---

## 🧩 Architecture (High-Level)

AutoSec is designed as a **gateway-style IDS** placed between an attacker-facing CAN interface and the protected in-vehicle CAN network:

- **vcan0 (Untrusted / Attacker Side)**  
  Receives injected, spoofed, and flood traffic from attack scripts.
- **IDS Bridge (Python + python-can)**  
  Observes frames, applies detection rules (rate/jitter, unknown IDs, misuse), and decides **allow vs block**.
- **vcan1 (Protected / Vehicle Side)**  
  Only forwards traffic deemed legitimate to the simulated vehicle (ICSim).
- **ICSim (Vehicle Simulator)**  
  Acts as the “car” receiving CAN frames.
- **Web UI (AutoSec Dashboard)**  
  Demonstrates protection toggling and system status: https://autosec.netlify.app/

---

## 🛠️ What You Need

- Linux VM / Ubuntu with **SocketCAN**
- `can-utils` installed (`candump`, `cansend`, `cangen`, `cansniffer`)
- **ICSim** built (`icsim`, `controls`)
- AutoSec repository in `~/autosEc`
- Python virtual environment at `~/autosEc/.venv` with `python-can`

---

## ⚡ Quick Demo (Before vs After)

### Demo A — No Protection

1. Terminal 1: Bring up buses  
2. Terminal 2: Start ICSim  
3. **Do not** run the IDS bridge  
4. Terminal 4: Run an attack  
   - **Result:** simulator becomes unstable, CAN traffic spikes

### Demo B — Protection ON

1. Terminal 1: Bring up buses  
2. Terminal 2: Start ICSim  
3. Terminal 3: Calibrate + start IDS bridge in protect mode  
4. Terminal 4: Run the same attack  
   - **Result:**  
     - IDS alerts with `action=block`  
     - `vcan1` stays cleaner than `vcan0`  
     - Simulator/UI remains more stable

---

## 🖥️ Terminal Map (Recommended)

Open **5 terminals** (Terminal 5 optional, great for screenshots):

- **Terminal 1:** Bring up `vcan0` (attacker) + `vcan1` (protected)
- **Terminal 2:** Run ICSim + Controls (the “car” on `vcan1`)
- **Terminal 3:** Activate venv + calibrate + run IDS bridge (protect mode)
- **Terminal 4:** Run attacks (against `vcan0`)
- **Terminal 5:** Observer (`candump` / `cansniffer`) to prove the difference

---

## 📋 Step-by-Step Runbook

### Terminal 1 — Create and Enable Virtual CAN Buses

~~~bash
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
~~~

✅ **Expected:** both show `UP` and `link/can`.

---

### Terminal 2 — Start the Car Simulation (ICSim)

**Important:** In the protected architecture, the car runs on `vcan1`.

~~~bash
start_sim
~~~

✅ **Expected:** IC Simulator + CANBus Control Panel windows appear.

---

### Terminal 3 — Activate venv, Calibrate Baseline, Start Protection Bridge

#### 1) Activate Python environment

~~~bash
source ~/autosEc/.venv/bin/activate
~~~

#### 2) Baseline calibration (learn normal IDs + timing)

Let ICSim run normally for ~60 seconds, then:

### 3) Run uvicorn to start the API

~~~bash
uvicorn controller:app --app-dir ~/autosEc/code/dashboard/controller --host 0.0.0.0 --port 8000
~~~

~~~bash
python3 ~/autosEc/code/ids/bridge.py calibrate --src vcan1 --seconds 60
~~~

✅ **Expected:** `calib.json` written with ~36 baseline IDs.

#### 4) Start the gateway in protection mode (leave running)

~~~bash
python3 ~/autosEc/code/ids/bridge.py bridge --mode protect --src vcan0 --dst vcan1
~~~

✅ Leave this running — this is your **before-it-hits-the-car** filter.

---

### Terminal 4 — Run Attacks (Attacker Side)

Attacks always target `vcan0`.

#### Flood / DoS

~~~bash
~/autosEc/code/attacks/flood.sh
~~~

Stop with: **CTRL + C**

#### Spoof (example)

~~~bash
~/autosEc/code/attacks/spoof_speed_244.sh
~~~

Stop with: **CTRL + C**

---

### Terminal 5 (Optional) — Observer / Proof Terminal

#### Live view (changes highlighted)

~~~bash
sudo cansniffer -c vcan1
~~~

#### Quick snapshot comparison

~~~bash
candump -n 30 vcan0
candump -n 30 vcan1
~~~

✅ In protect mode, `vcan0` will look noisy during attacks, while `vcan1` stays closer to baseline.

---

## 🎤 Demo Script (What to Say + What to Do)

### Demo A — No Protection

1. Terminal 1: Bring up buses  
2. Terminal 2: Start sim  
3. Terminal 4: Run `flood.sh`  
4. Point out: UI becomes unstable, traffic spikes  

### Demo B — Protection ON

1. Terminal 3: Calibrate  
2. Terminal 3: Start bridge in `--mode protect`  
3. Terminal 4: Run the same `flood.sh`  
4. Point out:  
   - Terminal 3 prints alerts with `action=block`  
   - `vcan1` stays cleaner than `vcan0`  
   - Simulator remains more stable  

<img width="3840" height="2486" alt="image" src="https://github.com/user-attachments/assets/69ca29a6-4975-4e56-bf06-28f6cfc9f1d0" />

---

## 🧯 Common Issues (Fast Fixes)

### `vcan0/vcan1: No such device`

You forgot Terminal 1. Re-run the setup commands.

### `start_sim` shows “dev duplicate / vcan1 is garbage”

`vcan1` already exists. Clean fix:

~~~bash
sudo ip link del vcan0 2>/dev/null || true
sudo ip link del vcan1 2>/dev/null || true
~~~

Then recreate buses.

### `bridge` command only shows `{calibrate, bridge}`

That’s correct. Use:

~~~bash
python3 ~/autosEc/code/ids/bridge.py bridge --mode protect --src vcan0 --dst vcan1
~~~

---

## 🧹 Clean Shutdown (End of Demo)

1. Stop attack scripts (**CTRL + C** in Terminal 4)  
2. Stop bridge (**CTRL + C** in Terminal 3)  
3. Stop simulation:

~~~bash
stop_sim
~~~

4. Optional: remove buses

~~~bash
sudo ip link del vcan0 2>/dev/null || true
sudo ip link del vcan1 2>/dev/null || true
~~~

---

## 📸 Screenshots (For Reports / Supervisor Updates)

Capture:
- ICSim + Control Panel windows visible
- Terminal 3 showing `[ALERT] ... action=block`
- Terminal 5 showing:
  - `candump vcan0` noisy
  - `candump vcan1` calmer

Embed screenshots:

~~~markdown
![ICSim Running](screenshots/icsim_running.png)
![IDS Alerts](screenshots/ids_alerts.png)
![vcan Comparison](screenshots/vcan_comparison.png)
~~~

---

## 🌐 Web Dashboard (Protection Toggle)

The AutoSec web interface demonstrates enabling/disabling protection and visualising system behaviour:

👉 https://autosec.netlify.app/

Use this during demos to conceptually show **Protection OFF vs Protection ON** alongside the terminal-based evidence.

---

## 📜 License

This project is for educational and research purposes as part of a final-year project on connected vehicle cybersecurity.
