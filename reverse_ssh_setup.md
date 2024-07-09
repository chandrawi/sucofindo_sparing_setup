---


---

<h2 id="mengatur-login-ssh-tanpa-password-dari-data-logger-ke-server">Mengatur login ssh tanpa password dari data logger ke server</h2>
<ol>
<li>
<p>Buat folder baru untuk menyimpan berkas konfigurasi</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">mkdir</span> -p /etc/sshtunnel
</code></pre>
</li>
<li>
<p>Buat pasangan kunci RSA yang akan digunakan dalam ssh</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> ssh-keygen -qN <span class="token string">""</span> -f /etc/sshtunnel/id_rsa
</code></pre>
</li>
<li>
<p>Salin kunci RSA ke server. Ada dua alternatif cara:</p>
<ul>
<li>Dengan perintah ssh-copy-id (ganti ip 103.183.74.245 dengan IP server)<pre class=" language-bash"><code class="prism  language-bash">ssh-copy-id -i /etc/sshtunnel/id_rsa.pub sshtunnel@103.183.74.245
</code></pre>
</li>
<li>Dengan salin manual isi kunci publik RSA dan tempel. Pertama tampilkan kunci publik yang telah dibuat<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">cat</span> /etc/sshtunnel/id_rsa.pub
</code></pre>
Berikut adalah contoh tampilan hasil perintah di atas<pre><code>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCXG0RpBevI8vvWLhus7do5yBitDFGQTpYAvmyztn/NwEnIX4zW+mYpOo8XPMnBo/u8FBbDCuOYNcSOcfh/uoT4wt8gdwGdMAm/CdrsQgaiR6Vw+gCwBaNNjYcjjyR04zIowyakQidqWUmPeeC9lWRms794T33A8qxNBtoRjub8Z8W19Jp6/Ajla4TqfDwXgbeg6aN/Ew++wUbjIynXXTkL7nU/ypnZZdOX+3/VIIv8bS5BNdV5DLMvngv/yvXQPBeSxhO1thy6TQ09g+gdj6XlDMUyIy4ZAlamjO07ZE55Ttf3GFNlJQTW73MyQ85p6wEyaCHPa21+zRIG/QobEMoMTZTO4SIHk38yWzjltTe92SN68GV9EeZJibEkxZAXPO2F3tKcQTopcOyiLqMssqSWs2chE+gklynu7L6upHK5Mep5F5UU4vllHW9vjGSSPKe9rY3x2tJH8L8I6Nzk9BEMcmOejf6onk9v83Up17y6Kd5OlHS5hxOydbimGXdmX90= root@clem-abb
</code></pre>
Selanjutnya salin dan tempel kunci publik di atas pada pada berkas ssh authorized_keys pada server.</li>
</ul>
</li>
<li>
<p>Uji masuk ke server tanpa menggunakan password</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">ssh</span> -i /etc/sshtunnel/id_rsa gundala@103.183.74.245
</code></pre>
<p>Jika semua berjalan dengan baik maka dengan perintah di atas akan membawa masuk sesi ssh ke server tanpa password.</p>
</li>
</ol>
<h2 id="mengatur-service-ssh-tunnel">Mengatur service ssh tunnel</h2>
<p>Setelah berhasil masuk sesi ssh ke server tanpa password, kita dapat membuat service sshtunnel sehingga komputer dapat diakses dengan ssh melalui perantara server yang memiliki IP publik. Berikut langkah-langkahnya:</p>
<ol>
<li>
<p>Buat berkas kofigurasi service baru</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> <span class="token function">nano</span> /etc/systemd/system/sshtunnel.service
</code></pre>
</li>
<li>
<p>Salin kode skrip berikut dengan beberapa penyesuaian.</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token punctuation">[</span>Unit<span class="token punctuation">]</span>
Description<span class="token operator">=</span>Service to maintain an <span class="token function">ssh</span> reverse tunnel
Wants<span class="token operator">=</span>network-online.target
After<span class="token operator">=</span>network-online.target
StartLimitIntervalSec<span class="token operator">=</span>0

<span class="token punctuation">[</span>Service<span class="token punctuation">]</span>
Type<span class="token operator">=</span>simple
ExecStart<span class="token operator">=</span>/usr/bin/ssh -qNn \
  -o ServerAliveInterval<span class="token operator">=</span>30 \
  -o ServerAliveCountMax<span class="token operator">=</span>3 \
  -o ExitOnForwardFailure<span class="token operator">=</span>yes \
  -o StrictHostKeyChecking<span class="token operator">=</span>no \
  -o UserKnownHostsFile<span class="token operator">=</span>/dev/null \
  -i /etc/sshtunnel/id_rsa \
  -R 4202:localhost:22 \
  sshtunnel@103.183.74.245 -p 22
Restart<span class="token operator">=</span>always
RestartSec<span class="token operator">=</span>60

<span class="token punctuation">[</span>Install<span class="token punctuation">]</span>
WantedBy<span class="token operator">=</span>multi-user.target
</code></pre>
<p>Pada baris <code>-R 4202:localhost:22 \</code> ganti port 4202 dengan port lain sehingga tidak bertabrakan dengan sshtunnel service yang lain.<br>
Pastikan juga IP server pada baris <code>sshtunnel@103.183.74.245 -p 22</code> sudah sesuai dengan server yang dituju.</p>
</li>
<li>
<p>Setelah berkas kofigurasi service telah sesuai, jalankan perintah berikut untuk memulai service sshtunnel.</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> systemctl <span class="token function">enable</span> --now sshtunnel
</code></pre>
</li>
<li>
<p>Cek apakah service sshtunnel sudah aktif dan berjalan.</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> systemctl status sshtunnel
</code></pre>
<p>Akan muncul tampilan mirip sebagai berikut jika sshtunnel berhasil aktif</p>
<pre><code>● sshtunnel.service - Service to maintain an ssh reverse tunnel
	 Loaded: loaded (/etc/systemd/system/sshtunnel.service; enabled; vendor preset: enabled)
	 Active: active (running) since Tue 2024-07-09 16:23:23 WIB; 3h 11min ago
   Main PID: 3662276 (ssh)
	  Tasks: 1 (limit: 2186)
	 Memory: 1.7M
		CPU: 538ms
	 CGroup: /system.slice/sshtunnel.service
			 └─3662276 /usr/bin/ssh -qNn -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -o ExitOnForwardFailure=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i /etc/&gt;
Jul 09 16:23:23 clem-abb systemd[1]: Started Service to maintain an ssh reverse tunne
</code></pre>
</li>
<li>
<p>Untuk menyudahi service sshtunnel jalankan perintah berikut.</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span> systemctl disable --now sshtunnel
</code></pre>
<p>Untuk kembali memulai service sshtunnel jalankan perintah pada poin <code>(3)</code> . Service sshtunnel akan berjalan otomatis setiap kali komputer restart.</p>
</li>
</ol>

