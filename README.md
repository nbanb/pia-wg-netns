# pia-wg-netns
Multihead implementation of PIA (PrivateInternetAccess) using WireGuard &amp; Linux netns namespaces

#############################################################################
##
##    20211202 : NBA / 14RV-NETWORK : CONNECT PIA VPN
##
##
##        Connect PIA VPN to 9 differents pre-defined Regions :
##
##        - from the same Linux instance  
##        - in diffÃ©rent network namespaces (netns)
##        - using WireGuard to connect PIA (out by DMZ-ext v90)
##          
##        - EXPOSE VPN to local network (in by DMZ-int v80)
##        - It is possible to ROUTE traffic through container IP 
##        - using the SOCKSv5 & HTTP(S) proxy :  
##        - Traffic input in SOCKS5/HTTP out by PIA VPN of selected region
##        - SOCKS5 listening on namespace container IP:1080
##        - HTTP(S) listening on namespace container IP:3128
##        - DNSMASQ listening on namespace container IP:53 (relay DNS request)
##        
##        - 9 predifined Region / update manually to select other regions
##        - To get a list of possible region, please run as root :      
##        - ${PWD##*/}/lst-reg
##        
##        - 10 different possible connections in standard PIA paid contract 
##        - Keeping 1 connection for testing or for direct use
##        - on device like laptop, tab, phone ... 
##        - If you need more free connection, do not connect the 9 
##        - predifined regions, only connect 8 or 7 or less ..; 
##        
##    Scripts forked by NBA from "privateinternetaccess" git repository: 
##    https://github.com/pia-foss/manual-connections.git    
##
##    SEE DISCLAMER FILE IN MAIN DIRECTOY 
##
#############################################################################


DATE of initial release : 2021-12-02


_____________________________________________________
Working files and differents implementations are in : 

nba-ns/
nba-ns/nsip
nba-ns/srv-list-20211127
nba-ns/run-inside-ns/
nba-ns/move-inside-ns/

_______________________
Implementation are in:

nba-ns/run-inside-ns
nba-ns/move-inside-ns

________________
Also include :

A test directory with some stuffs / tests on routing, macvlan, etc.

_____________________
Output & comments :

All scripts had nice output explaning what is happenning when running.
The code is nicely commented with a lot of information
You can modify the code to suites to your need, fork implementations, etc... 

_____________________
Configuration files :

You will find in each implementation directory a file named "var-and-functions-ns".
This file contains all variables.
You NEED TO MODIFY this file to reflect your network 

_____________________________________________________________________________
Two implementation of multi head VPN connection from PIA commercial service : 

You can launch multi instances of the PIA VPN on the same machine using this script.
his script had been designed on a VM machine having 2 vlan networks interface, one on vlan 80 and another
on vlan 90 : vlan 80 is internal DMZ (DMZ-int) and vlan 90 in external DMZ exposed to internet (DMZ-ext)
This VM access internet through vlan 90 (and internet access it through vlan 90 too), and all local networks which are allowed to access this machine access it through internal DMZ: vlan 80. 
To summerize the architecture, this machine is exposed to local network on vlan 80 and is exposed to 
internet on vlan 90 and its default gateway is on vlan 90 :

______________
Quick scheme:

  @ <-----> Firewall <----DMZ-ext-v90----> VM <----DMZ-int-v80----> Firewall <---- Local-net-v20,30,...



To expose all PIA VPN connections to your local network, you can do routing through each container (change 'ipvlan' by 'macvlan' in 'var-and-functions-ns' script, need promiscious mode on the physical network interface) with Policy Based Routing rules from your router or firewall or easier, you car redirect traffic on each container SOCKSv5 proxy. There is also an HTTP(S) proxy listening in each container on port 8080 and port 3128.
You can access all differents namespaces through a socks proxy listening on port 1080 (IP:1080) or using the HTTP proxy on each instance of the PIA VPN you launch so you can do conditionnal routing & proxying from your local network.
(easy to use)

To stay as lightest as possible, the SOCKSv5 proxy we use is 'microsocks' and the HTTP(S) proxy is 'tinyproxy' 
So you need to install both microsock and tinyproxy on the system you will deploy this PIA multihead solution.

____________________________
Required external packages :

- cURL
- jq
- wg-quick 
- wireguard
- microsocks
- tinyproxy
- logrotate

On debian, you can easyly install all theses packages using apt command :
apt update
apt install curl jq wireguard wireguard-dkms wireguard-tools microsocks tinyproxy





========================
- First implementation :                                                                 

________________________________________________
 WireGuard created from "root" namespace : 

According to WireGuard documentation, this "nba" implementation create namespaces for each 
VPN region (predifined & update-able in script) and launch WireGuard.    
WireGuard interface is created in the root namespace so it's original socket stays in the root namespace
even if you move the WireGuard interface to another namespace later.
In this implementation, the 'pia WireGuard' interface is created in the root namespace first and move to
the 'pia-$region' namespace which had been previously created.


This mode could permit lots of cool features of WireGuard
see : https://www.wireguard.com/netns/

It had been developped first to test WireGuard namespaces features and is maintained for compatibility and
futurs fork using all cool isolations features WireGuard and Linux netns namespaces provides.

Files :

nba-ns/move-inside-ns/ca.rsa.4096.crt
nba-ns/move-inside-ns/get_region-ns
nba-ns/move-inside-ns/get_token.sh
nba-ns/move-inside-ns/list_region
nba-ns/move-inside-ns/lst-reg
nba-ns/move-inside-ns/pia-wg-down
nba-ns/move-inside-ns/pia-wg-up
nba-ns/move-inside-ns/port_forwarding.sh
nba-ns/move-inside-ns/rt-nft-ns
nba-ns/move-inside-ns/run-pia-ns
nba-ns/move-inside-ns/nsip
nba-ns/move-inside-ns/connect_to_wireguard_with_token-ns
nba-ns/move-inside-ns/var-and-functions-ns
nba-ns/move-inside-ns/files/default.html
nba-ns/move-inside-ns/files/stats.html
nba-ns/move-inside-ns/files/logrotate-tiny
nba-ns/move-inside-ns/files/tinyproxy-pia-nb.conf



========================
Second implementation : 

________________________________________________
 WireGuard created directly in " namespace :

This "nba" implementation create namespaces for each VPN region (predifined & update-able in script) and 
launch WireGuard in a Linux netns container.
So in this mode, each container become the WireGuard root container and hold the WireGuard socket.
This mode is better for multihead PIA VPN instance where you use them as is.
If you want to benefit from WireGuard cool features using Linux netns namespaces, you will prefer to use 
the first described mode.
If you just want to use and fork traffic conditionnaly through the 9 predifined location from the same 
machine, this is the best implementation for you.


Files : 

nba-ns/run-inside-ns/ca.rsa.4096.crt
nba-ns/run-inside-ns/get_region-ns
nba-ns/run-inside-ns/get_token.sh
nba-ns/run-inside-ns/list_region
nba-ns/run-inside-ns/lst-reg
nba-ns/run-inside-ns/nsip
nba-ns/run-inside-ns/pia-wg-up
nba-ns/run-inside-ns/pia-wg-down
nba-ns/run-inside-ns/port_forwarding.sh
nba-ns/run-inside-ns/run-pia-ns
nba-ns/run-inside-ns/rt-nft-ns
nba-ns/run-inside-ns/connect_to_wireguard_with_token-ns
nba-ns/run-inside-ns/var-and-functions-ns
nba-ns/run-inside-ns/files/default.html
nba-ns/run-inside-ns/files/stats.html
nba-ns/run-inside-ns/files/logrotate-tiny
nba-ns/run-inside-ns/files/tinyproxy-pia-nb.conf



_________________________
Change predifined regions

To update script to change the 9 predifined region, you can have a list of PIA regions / servers which
have less than 0.5s of latency (most of the world, if not all the world !) when running 'lst-reg' script
The scripts had been designed for you just need to update a few lines in 'pia-wg-up' and 'pia-wg-down' 
scripts to change regions and output.


________________________________
Modify firewall & routing rules

Nftables filtering & routing ruleset are currently stored in each implementation directory in file :
rt-nft-ns
You need to adapt it to your need and to reflect your network security policies



Enjoy !
nbanba
