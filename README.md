# CNC-Conf-Cisco-Junos-L2VPN-P2P
This is an example for provisioning a point-to-point (p2p) l2vpn/l2circuit service between juniper mx and cisco ios-xr router using Crosswork Network Controller (CNC).

CNC provides ieft standard l2vpn tsdn core funcion package (CFP) leveraging NSO. This CFP can be extended to configure services in mulivendor enviroment without any extra development work for visualizing in CNC, i.e the visualization of the service work automatically in the topology view. In this example this is between a Juniper MX and a Cisco XR router. We leverage dCloud kitchen sink session, the devices used here are juniper vmx and cisco ios-xrv9K (CML) routers. 

# Configuration Example for l2vpn p2p (in CNC, the option is VPWS): 

## Cisco IOS XR:

<pre><code>
interface GigabitEthernet0/0/0/1.101 l2transport
 description T-SDN Interface
 encapsulation dot1q 512
!

l2vpn
 ignore-mtu-mismatch
 pw-class TEST
  encapsulation mpls
   control-word
   transport-mode vlan
  !
 !
 xconnect group TEST
  p2p TEST
   interface GigabitEthernet0/0/0/1.101
   neighbor ipv4 198.19.1.99 pw-id 101
    pw-class TEST
   !
  !
 !
!
</code></pre>

## Juniper MX:

<pre><code>
set interfaces ge-0/0/1 vlan-tagging
set interfaces ge-0/0/1 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 101 encapsulation vlan-ccc
set interfaces ge-0/0/1 unit 101 vlan-id 512
set interfaces ge-0/0/1 unit 101 family ccc
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 virtual-circuit-id 101
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 ignore-encapsulation-mismatch
set protocols l2circuit neighbor 198.19.1.4 interface ge-0/0/1.101 ignore-mtu-mismatch
set protocols mpls label-switched-path lsp_to_pe2 to 198.19.1.4
</code></pre>

# Enviroment Setup:

</br> **o** Requires mulivendor enviroment. In this case we need a network with cisco xr routers and juniper mx routers.
</br> **o** Users can create their environment in the lab with juniper vmx and vXR-9K or CML (Cisco Modeling Lab).
</br> **o** Install CNC, CDG, SR-PCE, NSO (with CNC core function packs).
</br> **o** In this case, we used dCloud kitchen sink demo session (custom content), which has all CNC enviroment installed and Cisco-Juniper network prepared.

# Steps to make it work with CNC:

</br> 1. Git clone the repository the Cisco Automation BU prepared (tsdn-juniper) in your NSO VM.
</br> https://github.com/maddn/tsdn-juniper.git
</br> We will be extending this tsdn-juniper repository for l2vpn configuration and add it into our current repository here.

</br> 2. Copy the "flat-l2vpn-juniper"  folder into the package folder for your NSO enviroment. Follow the **Installation** section of that repository to install the "flat-l2vpn-juniper" package.

</br> 3. Using the juniper config syntax stated in the above example, use NSO cli to config the commands and use ***commit dry-run outformat xml*** to get the configuration template and replace the required parameters with input variable.

</br> 4. Save the config templete inside ***~/ncs-run/packages/flat-l2vpn-juniper/templates*** folder as ***cisco-flat-L2vpn-fp-junos-p2p-l2circuit-template.xml***. 
</br> This file has been added with this repository.

</br> 5. Go to the folder ***~/ncs-run/packages/flat-l2vpn-juniper/python/flat_l2vpn_juniper*** and edit the ***Junos.py*** to add the p2p section inside the ***def conf_l2vpn(self, site, local):*** function.

<pre><code>
class Junos:
    def conf_l2vpn(self, site, local):
        self.log.info(f"Configuring Flat L2VPN/SR on Junos for service {site.pe}")

        if self.service.service_type == "evpn-vpws":
            l2vpn_vars = ncs.template.Variables()
            l2vpn_vars.add("LOCAL_NODE", "true" if local is True else "false")
            l2vpn_template = ncs.template.Template(site)
            l2vpn_template.apply("cisco-flat-L2vpn-fp-junos-template", l2vpn_vars)
        
        """ Add this section""" 
        elif self.service.service_type == "p2p":
            l2vpn_vars = ncs.template.Variables()
            l2vpn_vars.add("LOCAL_NODE", "true" if local is True else "false")
            l2vpn_template = ncs.template.Template(site)
            l2vpn_template.apply("cisco-flat-L2vpn-fp-junos-p2p-l2circuit-template", l2vpn_vars)
       """ End of section """
</code></pre>

</br> This file has been added in this repository.

</br> 6. To make this example work, we had to modify some template for NSO Core Function Package for L2vpn (**cisco-flat-L2vpn-fp-internal**). 
</br>    i. Go to the folder ***~/ncs-run/packages/cisco-flat-L2vpn-fp-internal/templates***.
</br>   ii. Edit both ***cisco-flat-L2vpn-fp-cli-evpn-vpws-template.xml*** and ***cisco-flat-L2vpn-fp-cli-p2p-template.xml*** with the content below:

        <l2vpn xmlns="http://tail-f.com/ned/cisco-ios-xr">
          <ignore-mtu-mismatch/>
          <pw-class>
            ......< skipping some sections>.....
            <encapsulation>
              <mpls>
                 ......< skipping some sections>.....
                <transport-mode>
                   <vlan/>
                </transport-mode>
               </mpls>
            </encapsulation>
          </pw-class>
          ......< skipping some sections>.....
        </l2vpn>

</br> 7. Reload the packages in NSO (***packages reload***)

# Provision P2P L2VPN service between Juniper and Cisco devices in CNC.

</br> 1. Select L2VPN under Services Section.
</br>
![image](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/assets/39807939/ff6001d9-59ac-4029-a199-d761a79255ac)


</br> 2. Either use GUI hitting "+" sign or import a configuation file in json format. 
</br>
![image](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/assets/39807939/6669cdd8-b40d-4a2b-9f36-c576d2d05666)


</br> 3. We will import a json configuration file for the service. The json file has been added in this repository.
</br>
![image](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/assets/39807939/751a759a-2fa9-4a93-93e8-7baaf85f674d)


</br> 4. Check the imported parameters and hit "commit changes" ( optional, also can do a dry-run before commit).
</br>
![image](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/assets/39807939/7792b16a-2876-49c5-91f3-33500ebf4d45)


</br> 5. Visualize the service in CNC:
</br> ![image](https://github.com/tchowdhu/CNC-Conf-Cisco-Junos-L2VPN-P2P/assets/39807939/e636c3fa-6b63-4cd1-9f68-cd008c24a41b)


# Verifying ouput on routers:

## Cisco IOS-XR (show l2vpn xconnect):

<pre><code>
Legend: ST = State, UP = Up, DN = Down, AD = Admin Down, UR = Unresolved,
        SB = Standby, SR = Standby Ready, (PP) = Partially Programmed,
        LU = Local Up, RU = Remote Up, CO = Connected, (SI) = Seamless Inactive

XConnect                   Segment 1                       Segment 2                
Group      Name       ST   Description            ST       Description            ST    
------------------------   -----------------------------   -----------------------------
TEST       TEST       UP   Gi0/0/0/1.512          UP       198.19.1.99     101    UP    
----------------------------------------------------------------------------------------
</code></pre>
## Juniper MX (show l2circuit connections):

<pre><code>
Layer-2 Circuit Connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NP -- interface h/w not present   
MM -- mtu mismatch               Dn -- down                       
EM -- encapsulation mismatch     VC-Dn -- Virtual circuit Down    
CM -- control-word mismatch      Up -- operational                
VM -- vlan id mismatch		 CF -- Call admission control failure
OL -- no outgoing label          IB -- TDM incompatible bitrate 
NC -- intf encaps not CCC/TCC    TM -- TDM misconfiguration 
BK -- Backup Connection          ST -- Standby Connection
CB -- rcvd cell-bundle size bad  SP -- Static Pseudowire
LD -- local site signaled down   RS -- remote site standby
RD -- remote site signaled down  HS -- Hot-standby Connection
XX -- unknown

Legend for interface status  
Up -- operational            
Dn -- down                   
Neighbor: 198.19.1.4 
    Interface                 Type  St     Time last up          # Up trans
    ge-0/0/1.512(vc 101)      rmt   Up     Aug 23 18:06:20 2023           1
      Remote PE: 198.19.1.4, Negotiated control-word: Yes (Null)
      Incoming label: 80002, Outgoing label: 80007
      Negotiated PW status TLV: No
      Local interface: ge-0/0/1.512, Status: Up, Encapsulation: VLAN
      Flow Label Transmit: No, Flow Label Receive: No

</code></pre>

# Demo Recording:

https://cisco.sharepoint.com/sites/AmericasMIGArchitecture/_layouts/15/stream.aspx?id=%2Fsites%2FAmericasMIGArchitecture%2FStrategy%20Documents%2FBradRoom%2FTahsin%20VPWS%20CNC%2D20230804%201542%2D2%2Emp4

# Note for developers/BU:

Feel free to use/edit this and add to the tsdn-juniper package.

# Note for Users:

</br>For quick usability, added the "flat-l2vpn-juniper" folder with all those modification mentioned above. Users can just import it, add to the nso package folder and install it for provisioning.
</br> Perform the cisco-flat-L2vpn-fp-internal related modifcation though as stated above.

# Credits (Contributions):

</br>1. Tahsin Chowdhury -tchowdhu@cisco.com (Automation)
</br>2. Bradley Ripalov - brriapol@cisco.com (IP/Router)
</br>***Original TSDN Juniper Repo: Michael Maddern - maddn@cisco.com (https://github.com/maddn/tsdn-juniper)***

