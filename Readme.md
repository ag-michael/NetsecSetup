
# Netsec Setup

Netsec Setup is a small python script I wrote to do the following:

- Execute and 'wrap' the OpenVPN process, occasionally monitoring it's output and taking actions.
- Execute and 'wrap' a secure dns resolver (such as dnscrypt)
- Take steps to prevent traffic from leaking outside of the VPN tunnel using IPtables and iproute2
- Monitor a predefined IP address for latency changes and TCP connectivity.  If the latency is less than a minimal latency value (measured against the OpenVPN gateway during startup), assume traffic is leaking, execute a preconfigured command and restart the OpenVPN client.
- On the preconfigured network interface (such as wlan0), randomize it's mac address at startup and install IPTables rules to ensure the only traffic in and out of that interface is towards the OpenVPN  gateway.
- //TODO: Possibly a few more things in the future like secure NTP, arp monitoring and setup of a secondary VPN tunnel through the primary tunnel for DNS and other network services (to compartmentalize the privacy risk paused by VPN providers: deduplicating DNS,OSCP and other common network noise might help.)

It is intended to be executed at boot time by the init process (specifically to be executed from /etc/netconfig.d/ by something like 'wicked'  for openSUSE). It can also be integrated as part of any network setup process, or as part of it's own init script that depends on the network service.

# Configuration

This is what the default configuration looks like:
```
{
	"logfile":"/var/log/Netsec.log",
	"watchdog_host":"1.1.1.1",
	"watchdog_port":80,
	"real_interface":"wlan0",
	"ovpn_ip":"192.168.0.1",
	"ovpn_cmdline":["/usr/sbin/openvpn","--config","/etc/openvpn/myvpn.ovpn"],
	"dnscrypt_cmd":["dnscrypt-proxy","-R","cs-de"],
	"real_defgwy":"10.0.0.1",
	"latency_mindelta":10,
	"leakcmd":["/bin/true"]
}
```
### Configuration parameters:
- logfile: Path to the log file
- watchdog_host: This is the IP address that will be used to monitor connectivity and latency changes.
- watchdog_port: This is the port on the watchdog_host IP used to monitor TCP connectivity. 
- real_interface: This is the name of the network interface used to connect to the OpenVPN gatway.
- ovpn_ip: The IP address of the OpenVPN gateway
- ovpn_cmdline: The command-line used to execute OpenVPN 
- dnscrypt_cmd: The command-line used to execute dnscrypt
- real_defgwy: The default gateway that would be used to reach the openVPN gateway.
- latency_mindelta: This value is in milli-seconds, the watchdog IP can have  a latency as much as latency_mindelta less than the OpenVPN gateway's pre-measured latency.
- leakcmd: This command is executed when a possible leak is detected.

## Usage
It takes one argument: The path to the json configuration file.
```
/etc/netconfig.d/NetsecSetup /etc/netsec.json
```

## N.B.
I wrote this as a "weekend project" to help me integrate the different steps I normally take to achieve some form of network security for my personal internet traffic.  Suggestions and help to improve this script is always welcome.
