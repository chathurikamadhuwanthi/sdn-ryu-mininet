# 🧠 SDN Lab: When the Switch Stops Thinking

> "What if the switch didn't make decisions at all?"

A small Software-Defined Networking lab where **a Python program is the brain** and a virtual switch is just the muscle.

Built with **Ryu** (controller), **Mininet** (virtual network), and **Open vSwitch**, using OpenFlow 1.3.

---

## 🗺️ The Idea

In a normal network, every router and switch makes its own decisions. In SDN, the decision-making (**control plane**) moves into one central controller, and the switch only forwards packets (**data plane**).

```
        [ Ryu Controller ]   <- the brain (Python)
               |
          [ Switch s1 ]      <- the muscle
          /     |     \
        h1      h2     h3    <- virtual hosts
```

---

## 🧪 Two Experiments

### Level 1: Learning Switch (`learning_switch.py`)
The switch knows nothing at first. When an unknown packet arrives, it asks the controller (**PacketIn**). The controller learns the MAC address, then installs a rule on the switch (**FlowMod**).

### Level 2: Firewall (`firewall.py`)
Same controller, plus a few lines of Python that block ICMP (ping). No CLI on the switch, no ACLs on each device. Just code.

---

## 📊 What I Observed

| Test | Result |
|---|---|
| `pingall` (learning switch) | 0% dropped |
| First ping | ~33 ms (went through the controller) |
| Later pings | ~0.08 ms (switch used the installed rule) |
| `ping` with firewall | 100% packet loss |

The first packet is slow because the controller has to decide. After that, the switch forwards on its own using the installed rule.

---

## 🚀 Run It Yourself

```bash
# 1. Start the controller (Python 3.9 venv recommended)
ryu-manager learning_switch.py --verbose     # or firewall.py

# 2. In another terminal, start the network
sudo mn --topo single,3 \
  --controller remote,ip=127.0.0.1,port=6653 \
  --switch ovsk,protocols=OpenFlow13

# 3. Test it
mininet> pingall
mininet> h1 ping -c 3 h2
mininet> sh ovs-ofctl -O OpenFlow13 dump-flows s1
```

---

## 📸 Screenshots

**Learning switch: first ping goes through the controller, later pings are fast**

![Learning switch](images/learning-switch.png)

**Firewall: the controller blocks ICMP**

![Firewall: controller blocks ICMP](images/firewall-controller.png)

**Firewall: ping gets 100% packet loss**

![Firewall: ping gets 100% loss](images/firewall-ping.png)

---

## 💡 What I Learned

- The control plane and data plane can live in different places
- The first packet of a flow goes to the controller; the rest never do
- A firewall rule is just a flow entry with no actions (drop)
- Setting up the tools (Python versions, VM networking) taught me as much as the code

## 🔭 Next Ideas

- [ ] Test that TCP traffic (`iperf`) still passes through the firewall
- [ ] Load balancer across two servers
- [ ] Traffic monitoring dashboard with Flask
- [ ] Block specific ports instead of all ICMP

---

Built by **WL. Chathurika Madhuwanthi** · University of Sri Jayewardenepura
