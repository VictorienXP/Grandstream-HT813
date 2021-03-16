# Grandstream HT813 - France PSTN + FreePBX

This is notes of my settings for using a Grandsteam HT813 with a french PSTN line with FreePBX.

## Version

FreePBX: 15.0.17.24

Grandstream HT813: 1.0.9.1 

## Overall setup

FX0 port -- PSTN line (tested with France Telecom POTS and Freebox Revolution router phone line)

FXS port -- analog phone

WAN port -- your network

LAN port -- whatever device... it will be bridged

I assume that your FreePBX use PJSIP on port 5060 UDP.

In the exemple the phone number is 0123456789.

## Setup in FreePBX

### Analog Phone Extension

Create an extension PJSIP for the analog phone. For the exemple I will use the extension **3210** with secret **your_strong_password_with_letter_and_number_for_extension_3210**

### Trunk Setup

I use a SIP chan_pjsip Trunk.

General

    Trunk Name: 0123456789
    Outbound CallerID: <0556302343>
    Asterisk Trunk Dial Options (Overide): TR

The overide on Asterisk is in order to have a tone when outbound call are placed... there is a 10sec delay (any idea on how to reduce that would be great).

pjsip Settings - General

    Username: 0123456789
    Secret: your_strong_password_with_letter_and_number_trunk
    Authentication: Outbound
    Registration: Receive
    Context: from-trunk

pjsip Settings - Advanced

    DTMF Mode: RFC 4733
    Trust RPID/PAI: Yes
    
pjsip Settings - Codecs

    alaw

I disabled all the other codecs... somehow I sometimes ran into trouble when call were coming in.

## HT813 Setup

Find the device on your network and access it via http. Note: when I enabled https, I got into certificat trouble and I had to reset my box.

Default access: admin/admin

### Basic Settings

Change End User Password.
Change Viewer Password.

    Disable SSH: Yes
    Time Zone: GMT+01:00 Paris
    Device Mode: Bridge
    Reply to ICMP on WAN port: Yes
    Enable LAN DHCP: No
    Unconditional Call Forward to VOIP 
        User ID: 0123456789
        Sip Server: IP_of_FreePBX
        Sip Destination Port: 5060

### Advanced Settings

Change Admin Password.

If you have a VLAN for your VOIP then you can set it here. Be careful to have a DHCP server there or setup a fixed IP address.

    802.1Q/VLAN Tag: your_VLAN_ID
    
Under *Firmware Upgrade and Provisioning*, empty the *Config Server Path*.

    3CX Auto Provision: No
    
    System Ring Cadence: c=1500/3500;
    Dial Tone: f1=440@-10,f2=0@-10,c=0/0;
    Ringback Tone: f1=440@-10,f2=0@-10,c=1500/3500;
    Busy Tone: f1=440@-10,f2=0@-10,c=500/500;
    Reorder Tone: f1=440@-10,f2=0@-10,c=50/50;
    Confirmation Tone: f1=350@-11,f2=440@-11,c=100/100-100/100-100/100;
    Call Waiting Tone: f1=440@-13,c=300/10000;
    Prompt Tone: f1=350@-13,f2=440@-13,c=0/0;
    
    Lock Keypad Update: No
    Disable Voice Prompt: Yes
    Disable Direct IP Call: Yes
    Life Line Mode: Auto
    NTP Server: 2.fr.pool.ntp.org
    
If you want to debug your HT813, then you might want to setup syslog. You can use to small software to setup a syslog server for Windows with a nice GUI. https://github.com/MaxBelkov/visualsyslog/tree/v1.6.4

### FXS Port

    Primary SIP Server: IP_of_FreePBX
    Outbound Proxy: IP_of_FreePBX
    SIP User ID: 3210
    Authenticate ID: 3210
    Authenticate Password: your_strong_password_with_letter_and_number_for_extension_3210
    Register Expiration: 5
    Allow Incoming SIP Messages from SIP Proxy Only: Yes
    SIP REGISTER Contact Header Uses: WAN Address
    Use First Matching Vocoder in 200OK SDP: Yes
    SLIC Setting: EUROPEAN CTR21
    Gain: RX 0dB
    
### FXO Port
   
    Primary SIP Server: IP_of_FreePBX
    Outbound Proxy: IP_of_FreePBX
    SIP User ID: 0123456789
    Authenticate ID: 0123456789
    Authenticate Password: your_strong_password_with_letter_and_number_trunk
    Register Expiration: 5
    Allow Incoming SIP Messages from SIP Proxy Only: Yes
    SIP REGISTER Contact Header Uses: WAN Address
    Preferred Vocoder choice 1->7: PCMA
    Caller ID Transport Type: Relay via SIP P-Asserted-Identity
    Enable PSTN Disconnect Tone Detection: Yes
    PSTN Disconnect Tone: f1=440@-30,f2=440@-30,c=500/500;
    AC Termination Model: Country-based
    Country-based: FRANCE
    Number of Rings: 2 (if you put 1, the system will not get Caller ID)
    PSTN Ring Thru FXS: No
    Stage Method (1/2): 1
    
Once all those setup done, you can reboot your HT813. The FX0 and FXS led should light up once the lines are registered in FreeBPX. You can also check the status in the Status page.

## Anonymous Caller

The HT813 can block anonymous caller... but I decided to not activate the option. I use the Privacy Manager in the Inbound Route of FreePBX.

## Gain and echo problems

If you have gain or echo problems, you should modify your gain settings in the FXS/FXO pages. The change of gain listed above are for my phone lines and might not apply to you.

## Credits
