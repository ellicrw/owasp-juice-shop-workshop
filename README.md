OWASP Juice Shop Pentest Lab — Windows 11 (Docker) + Kali (VMware Host-Only / VMnet1)

Contents

Overview

Requirements




Run Juice Shop on Windows 11 (Docker)




Configure VMware (Host-Only / VMnet1)




PowerShell — find the VMnet1 (host) IP




Kali — open Firefox → http://<vmnet1-ip>:3000




Verify & quick test commands

Snapshots, safety & troubleshooting

Overview

Host: Windows 11 — Juice Shop served from Docker on port 3000.

Attacker: Kali Linux VM (VMware) — network mode Host-Only (VMnet1).

Goal: From Kali, open Firefox / curl / nmap → http://<vmnet1-ip>:3000 to practice OWASP Top 10.

Requirements

Windows 11 (admin access)

Docker Desktop for Windows (or Docker Engine)

VMware Workstation Player / Pro (or VMware Fusion)

Official Kali Linux VMware image

Basic familiarity with PowerShell and VMware UI

1) Run OWASP Juice Shop on Windows 11 (Docker — recommended)

Open PowerShell as Administrator and run:

# pull latest image
docker pull bkimminich/juice-shop

# run Juice Shop mapped to all interfaces on port 3000
docker run --rm -d -p 0.0.0.0:3000:3000 bkimminich/juice-shop

Confirm on the Windows host: open http://localhost:3000 — the app should load.

Add a Windows Firewall rule (PowerShell admin):

New-NetFirewallRule -DisplayName "JuiceShop 3000" -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow

Important: make sure you ran the container with 0.0.0.0:3000:3000. If it’s bound to 127.0.0.1, the guest won't reach it.

2) Configure VMware — Host-Only (VMnet1)

We’ll use Host-Only (VMnet1) so the Kali VM and the host can communicate while keeping the lab isolated from your network.

Power off the Kali VM.

In VMware: Select the Kali VM → Settings → Network Adapter.

Choose Host-Only (VMnet1). Apply and close.

VMnet1 is typically called “VMware Network Adapter VMnet1” on Windows. We’ll use that adapter’s IPv4 address as the host IP Kali will hit.

3) PowerShell — find the VMnet1 (host) IP

Open PowerShell (Admin) and run:

# quick: show Windows adapter list so you can find the VMware Host-Only (VMnet1) entry
ipconfig /all

Find the adapter named “VMware Network Adapter VMnet1” and copy its IPv4 Address (example: 192.168.56.1). That is the host-side VMnet1 IP — Kali can reach services bound to the host via that IP.

4) Start Kali (Host-Only) and open Firefox → http://<vmnet1-ip>:3000

Start the Kali VM (network set to Host-Only / VMnet1).

In Kali, confirm networking:

ip addr show
# or ping the host IP you copied
ping -c 3 <vmnet1-host-ip>

Open Firefox in Kali. In the address bar enter:

http://<VMnet1-host-ip>:3000

Example: http://192.168.56.1:3000

If the Juice Shop page loads in Kali’s Firefox, Kali is talking to the host Juice Shop correctly.

5) Verify from Kali — quick testing commands

From Kali terminal:

# simple HTTP check
curl -I http://<VMnet1-IP>:3000

# full HTML output
curl -s http://<VMnet1-IP>:3000 | head -n 20

# nmap port check
nmap -Pn -p 3000 <VMnet1-IP>

Basic lab tooling (for your practice only): Burp Suite, nmap, gobuster, sqlmap, wfuzz, etc.

Snapshots, safety & troubleshooting
Snapshots

Take a Windows snapshot before big changes (Docker install, config).

Take a Kali snapshot after you configure it as your attacker baseline.

Revert snapshots between exercises to reset state.

Troubleshooting checklist

Host loads http://localhost:3000 but Kali cannot:

Ensure Docker used -p 0.0.0.0:3000:3000 (not 127.0.0.1). Re-run if needed.

Confirm VM network is Host-Only (VMnet1) (VM settings).

Confirm the host VMnet1 adapter IP from ipconfig and use that IP in Kali.

Confirm Windows Firewall allows inbound TCP on port 3000 (see firewall rule above).

If ipconfig shows VMnet8 or VMnet1 mismatch: double-check the VM adapter type in VMware (Host-Only = VMnet1).

If curl returns HTML but Firefox times out: check Kali’s browser proxy settings (is Burp running?).

Safety & ethics

This lab is for education on machines you own or are authorized to test. Do not attack systems you don’t own or have explicit permission to test. Follow local laws and institutional policies.
