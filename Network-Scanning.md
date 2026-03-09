| Port Scan Type                 | Example Command                                       |
| ------------------------------ | ----------------------------------------------------- |
| TCP Null Scan                  | `sudo nmap -sN MACHINE_IP`                            |
| TCP FIN Scan                   | `sudo nmap -sF MACHINE_IP`                            |
| TCP Xmas Scan                  | `sudo nmap -sX MACHINE_IP`                            |
| TCP Maimon Scan                | `sudo nmap -sM MACHINE_IP`                            |
| TCP ACK Scan                   | `sudo nmap -sA MACHINE_IP`                            |
| TCP Window Scan                | `sudo nmap -sW MACHINE_IP`                            |
| Custom TCP Scan                | `sudo nmap --scanflags URGACKPSHRSTSYNFIN MACHINE_IP` |
| Spoofed Source IP              | `sudo nmap -S SPOOFED_IP MACHINE_IP`                  |
| Spoofed MAC Address            | `--spoof-mac SPOOFED_MAC`                             |
| Decoy Scan                     | `nmap -D DECOY_IP,ME MACHINE_IP`                      |
| Idle (Zombie) Scan             | `sudo nmap -sI ZOMBIE_IP MACHINE_IP`                  |
| Fragment IP data into 8 bytes  | `-f`                                                  |
| Fragment IP data into 16 bytes | `-ff`                                                 |



| Option                   | Purpose                                  |
| ------------------------ | ---------------------------------------- |
| `--source-port PORT_NUM` | specify source port number               |
| `--data-length NUM`      | append random data to reach given length |

| Option     | Purpose                               |
| ---------- | ------------------------------------- |
| `--reason` | explains how Nmap made its conclusion |
| `-v`       | verbose                               |
| `-vv`      | very verbose                          |
| `-d`       | debugging                             |
| `-dd`      | more details for debugging            |


TCP syn Scan - ```nmap -sS MACHINE_IP```

Host discovery only - sn

ARP scan - PR

ICMP echo scan - PE

ICMP timestamp scan - PP

ICMP mask adress scan - PM

TCP syn scan - PS

TCP ack scan - PA

UDP scan - PU

TCP connect scan - sT

TCP syn scan - sS

TCP FIN scan - sF                // The following three flags gives response to closed ports

TCP NULL scan - sN               // All flags are set to NULL

TCP xmas scan - sX               // Contains 3 flags FIN,URG and PSH   

UDP scan - sU

UDP scan combined with stealth - sU sS

NO DNS - n

query DNS even for offline hosts - R

Enable fast mode and decrease the number of scanned ports from 1000 to 100 most common ports - F

scan the ports in consecutive order instead of random order - r

Service Version scan - sV

OS detection - O

Aggresive scan - A

Idle Scan (Zombie Scan) - sI <Zombie> <target>

ACK Scan - sA     // this type of scan is more suitable to discover firewall rule sets and configuration

Windows scan - sW   // Pretty much the sam as ack scan these 2 give same response regardless of the state of the port but it helps to identify ports which are not blocked by 
firewall

Maimon scan - sM   // Contains FIN and ACK flags

-T0  Paranoid
-T1  Sneaky
-T2  Polite
-T3  Normal (default)
-T4  Aggressive
-T5  Insane

port list: -p22,80,443

port range: -p1-1023

Alternatively, you can choose to control the packet rate using --min-rate <number> and --max-rate <number>. For example, --max-rate 10 or --max-rate=10 ensures that your scanner is not sending more than ten packets per second.

Moreover, you can control probing parallelization using --min-parallelism <numprobes> and --max-parallelism <numprobes>. Nmap probes the targets to discover which hosts are live and which ports are open; probing parallelization specifies the number of such probes that can be run in parallel. For instance, --min-parallelism=512 pushes Nmap to maintain at least 512 probes in parallel; these 512 probes are related to host discovery and open ports.


Custom scan : 

  If you want to experiment with a new TCP flag combination beyond the built-in TCP scan types, you can do so using --scanflags . For instance, if you want to set SYN, RST, and FIN simultaneously, you can do so using --scanflags RSTSYNFIN . As shown in the figure below, if you develop your custom scan, you need to know how the different ports will behave to interpret the results in different scenarios correctly.


Spoofing ips: 

  In general, you expect to specify the network interface using -e and to explicitly disable ping scan -Pn. Therefore, instead of nmap -S SPOOFED_IP 10.48.165.50, you will need to issue nmap -e NET_INTERFACE -Pn -S SPOOFED_IP 10.48.165.50 to tell Nmap explicitly which network interface to use and not to expect to receive a ping reply. It is worth repeating that this scan will be useless if the attacker system cannot monitor the network for responses. 


You can launch a decoy scan by specifying a specific or random IP address after -D. For example, nmap -D 10.10.0.1,10.10.0.2,ME 10.48.165.50 will make the scan of 10.48.165.50 appear as coming from the IP addresses 10.10.0.1, 10.10.0.2, and then ME to indicate that your IP address should appear in the third order. Another example command would be nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME 10.48.165.50, where the third and fourth source IP addresses are assigned randomly, while the fifth source is going to be the attacker’s IP address. In other words, each time you execute the latter command, you would expect two new random IP addresses to be the third and fourth decoy sources.




Spoofing mac-addresses:

  When you are on the same subnet as the target machine, you would be able to spoof your MAC address as well. You can specify the source MAC address using --spoof-mac SPOOFED_MAC. This address spoofing is only possible if the attacker and the target machine are on the same Ethernet (802.3) network or same WiFi (802.11).



Fragmented Packets

Nmap provides the option -f to fragment packets. Once chosen, the IP data will be divided into 8 bytes or less. Adding another -f (-f -f or -ff) will split the data into 16 byte-fragments instead of 8. You can change the default value by using the --mtu; however, you should always choose a multiple of 8.

To properly understand fragmentation, we need to look at the IP header in the figure below. It might look complicated at first, but we notice that we know most of its fields. In particular, notice the source address taking 32 bits (4 bytes) on the fourth row, while the destination address is taking another 4 bytes on the fifth row. The data that we will fragment across multiple packets is highlighted in red. To aid in the reassembly on the recipient side, IP uses the identification (ID) and fragment offset, shown on the second row of the figure below.


--data-length NUM


--reason 	explains how Nmap made its conclusion

-v 	verbose

-vv 	very verbose

-d 	debugging

-dd 	more details for debugging