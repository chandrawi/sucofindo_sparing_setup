---


---

<h1 id="setup-ssh-login-without-password-to-server">Setup ssh login without password to server</h1>
<p>sudo mkdir -p /etc/sshtunnel<br>
sudo ssh-keygen -qN “” -f /etc/sshtunnel/id_rsa<br>
ssh-copy-id -i /etc/sshtunnel/id_rsa.pub <a href="mailto:sshtunnel@103.183.74.245">sshtunnel@103.183.74.245</a></p>
<h1 id="setup-ssh-tunnel-service">Setup ssh tunnel service</h1>
<p>sudo nano /etc/systemd/system/sshtunnel.service</p>
<h1 id="copy-following-lines">Copy following lines</h1>
<p>[Unit]<br>
Description=Service to maintain an ssh reverse tunnel<br>
Wants=network-online.target<br>
After=network-online.target<br>
StartLimitIntervalSec=0</p>
<p>[Service]<br>
Type=simple<br>
ExecStart=/usr/bin/ssh -qNn <br>
-o ServerAliveInterval=30 <br>
-o ServerAliveCountMax=3 <br>
-o ExitOnForwardFailure=yes <br>
-o StrictHostKeyChecking=no <br>
-o UserKnownHostsFile=/dev/null <br>
-i /etc/sshtunnel/id_rsa <br>
-R 2202:localhost:22 <br>
<a href="mailto:sshtunnel@103.183.74.245">sshtunnel@103.183.74.245</a> -p 22<br>
Restart=always<br>
RestartSec=60</p>
<p>[Install]<br>
WantedBy=multi-user.target</p>
<h1 id="enable-and-test-ssh-tunnel">Enable and test ssh tunnel</h1>
<p>sudo systemctl enable --now sshtunnel<br>
sudo systemctl status sshtunnel<br>
sudo systemctl disable --now sshtunnel</p>

