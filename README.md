# AutoSec

## Opem four separate terminals in linux

### Terminal 1:
  lute-linux@linux-ubuntu:~/Desktop$ sudo modprobe can
  [sudo] password for lute-linux: 
  lute-linux@linux-ubuntu:~/Desktop$ sudo modprobe can_raw
  lute-linux@linux-ubuntu:~/Desktop$ sudo modprobe vcan
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link del vcan0 2>/dev/null || true
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link del vcan1 2>/dev/null || true
  lute-linux@linux-ubuntu:~/Desktop$ 
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link add dev vcan0 type vcan
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link add dev vcan1 type vcan
  lute-linux@linux-ubuntu:~/Desktop$ 
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link set up vcan0
  lute-linux@linux-ubuntu:~/Desktop$ sudo ip link set up vcan1
  lute-linux@linux-ubuntu:~/Desktop$ 
  lute-linux@linux-ubuntu:~/Desktop$ ip link show vcan0
  6: vcan0: <NOARP,UP,LOWER_UP> mtu 72 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
      link/can 
  lute-linux@linux-ubuntu:~/Desktop$ ip link show vcan1
  7: vcan1: <NOARP,UP,LOWER_UP> mtu 72 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
      link/can 
  lute-linux@linux-ubuntu:~/Desktop$ 

  ### Termianl 2:
  lute-linux@linux-ubuntu:~/Desktop$ start_sim
  [sudo] password for lute-linux: 
  Error: either "dev" is duplicate, or "vcan1" is a garbage.
  [start_sim] Launching ICSim + Controls on vcan1\u2026
  [start_sim] PIDs: icsim=4031 controls=4034
  lute-linux@linux-ubuntu:~/Desktop$ ps aux | egrep "icsim|controls" | grep -v egrep
  lute-li+    4032 10.0  1.5 985080 124388 pts/1   Sl   09:58   0:02 ./icsim vcan1
  lute-li+    4035  2.1  1.5 983804 123320 pts/1   Sl   09:58   0:00 ./controls vcan1
  lute-li+    4073 50.0  0.0   6272  2072 pts/1    S+   09:58   0:00 grep -E --color=auto icsim|controls
  lute-linux@linux-ubuntu:~/Desktop$ 


### Termianl 3:


### Terminal 4:
