# AutoSec

## How to demo in a nutshell “before vs after protection”

  ### Demo A (No protection)
  |
  	•	Run Terminal 1 + Terminal 2
  	•	Do NOT run Terminal 4
  	•	Run attack in Attack terminal → sim goes crazy
  
  ### Demo B (Protection ON)
  |
  	•	Run Terminal 4 (bridge --mode protect)
  	•	Run same attack → you should see:
  	•	IDS alerts show action=block
  	•	vcan1 stays cleaner than vcan0
  	•	UI/sim more stable
  
---------------------------------------------------------------------------------------------------------------------------------------------------------------

# AutoSec Runbook (ICSim + Attacks + IDS Bridge)
## Open 5 terminals (Terminal 5 optional) on linux

### Terminal 1

   bring up both buses:
      $ sudo modprobe can can_raw vcan
      $ sudo ip link del vcan0 2>/dev/null || true
      $ sudo ip link del vcan1 2>/dev/null || true
      $ sudo ip link add dev vcan0 type vcan
      $ sudo ip link add dev vcan1 type vcan
      $ sudo ip link set up vcan0
      $ sudo ip link set up vcan1
      
  Check:
    $ ip link show vcan0
    $ ip link show vcan1

-----------------------------------------------------------------------------------------------
    
### Termianl 2 | Start the car sim (ICSim) on vcan0

  Start sim (ICSim must be on vcan1):
    $ start_sim

-----------------------------------------------------------------------------------------------

### Termianl 3 | Activate venv + calibrate IDS from vcan0

  Activate Python environment + start IDS “gateway”
    $ source ~/autosEc/.venv/bin/activate
  
  Let ICSim run normally for ~60 seconds:
    $ python3 ~/autosEc/code/ids/bridge.py calibrate --src vcan1 --seconds 60
  
  Start Protect mode (leave running):
    python3 ~/autosEc/code/ids/bridge.py protect --src vcan0 --dst vcan1

-----------------------------------------------------------------------------------------------

### Terminal 4 (leave running) | Attack |  Start the “gateway” in protect mode

  Attacks should go to vcan0 now (attacker side):
    $ ~/autosEc/code/attacks/flood.sh

  Protect Mode:
     ** “User presses Start → protect mode” (Terminal B) **
       Stop bridge (CTRL+C), then:
         $ python3 ~/autosEc/code/ids/bridge.py bridge --mode protect --src vcan0 --dst vcan1
         |
     ** Re-run the SAME attack (proves blocking) **
        Go back to Terminal 4:
          ~/autosEc/code/attacks/flood.sh
         |  
     ** What you should see now **
  	    •	Terminal B: alerts show action=block
  	    •	Simulator stays much more stable (attack doesn’t “hit” the car)
  	    •	Observer on vcan1 does NOT get flooded the same way
         |
     ** Stop flood: CTRL + C in Terminal 4 **

-----------------------------------------------------------------------------------------------

### Terminal 5 (OBSERVER) | (optional but recommended)

  This is just to visually confirm traffic.
    $ sudo cansniffer -c vcan1
      |
      or quick snapshots:
          $ candump -n 30 vcan0
          $ candump -n 30 vcan1



