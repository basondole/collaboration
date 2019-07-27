# Setting up Cisco Call Manager Express

The objective is to configure a call manager for users to make internal calls in a SOHO environment. Users can use Cisco IP Phones (SCCP) or third party SIP Phones

The router used as CME is a Cisco 3725 Integrated Services Router running IOS image c3725-adventerprisek9-mz.124-15.T14.bin  

## Topology view  

<img width="732" alt="collaboration1" src="https://user-images.githubusercontent.com/50369643/61995423-090b6900-b091-11e9-8151-6a0c0f5c5f70.png">  


## SIP Configuration

```
voice service voip 
 allow-connections h323 to h323
 allow-connections h323 to sip
 allow-connections sip to h323
 allow-connections sip to sip
 redirect ip2ip
 sip
  bind control source-interface FastEthernet0/1
  bind media source-interface FastEthernet0/1
  registrar server expires max 500 min 60

voice class codec 10
 codec preference 1 g711ulaw

voice register global
 mode cme
 source-address 192.168.251.96 port 5060
 max-dn 20
 max-pool 20
 authenticate register
 authenticate realm local
 create profile

gateway 
 timer receive-rtp 1200

sip-ua 
```

### Registering SIP Phones and Phone Directory entries  
```
voice register dn  1
 number 1112
 name 1112
 allow watch

voice register pool  1
 id mac 0015.6579.F61E
 number 1 dn 1
 presence call-list
 username 1112 password 1112
 codec g711ulaw


voice register dn  2
 number 1113
 allow watch

voice register pool  2
 id mac 0000.0000.0000
 number 1 dn 2
 presence call-list
 username 1113 password 1113
 codec g711ulaw
 ```
 
 ### Configuring dial peer for SIP phones  
 ```
 dial-peer voice 1 voip
 session protocol sipv2
 incoming called-number .
 dtmf-relay rtp-nte
 ```
 
## SCCP Configuration
```
telephony-service
 max-ephones 5
 max-dn 5
 ip source-address 192.168.251.96 port 2000
 create cnf-files
```

### Registering SCCP Phones and Phone Directory entries  
```
ephone-dn 1 dual-line
 number 1000
 label PHONE#1000

ephone 1
 device-security-mode none
 mac-address 0800.2764.4F44
 type CIPC
 button  1:1
```

## Yealink Phone Configuration  
Browse the IP of the Yealink phone to access the configuration wizard  

<img width="913" alt="collaboration2" src="https://user-images.githubusercontent.com/50369643/61995457-95b62700-b091-11e9-9f04-bf3b511006a0.PNG">

## 3CX Softphone configuration

<img width="288" alt="collaboration3" src="https://user-images.githubusercontent.com/50369643/61995871-87b6d500-b096-11e9-8d65-1c4691bd74ed.PNG">


## CIPC Configuration

<img width="297" alt="collaboration6" src="https://user-images.githubusercontent.com/50369643/61996506-a2407c80-b09d-11e9-8c47-d0243b11e3fa.PNG">  


## Verification of phone registration  
### SIP  

```
show sip-ua status registrar
Line          destination      expires(sec)  contact
              call-id
              peer
============================================================
1112          192.168.250.92   344           192.168.250.92
              1548037929@192.168.250.92                     
              40002

1113          192.168.251.88   104           192.168.251.88
              Y2Q4ZjcyN2Y5MjgxYjQzYzJlZTZmM2ZiN2NiYTAzODU.  
              40001

```
#### 3CX
<img width="124" alt="collaboration4" src="https://user-images.githubusercontent.com/50369643/61996343-ef235380-b09b-11e9-8892-f9996f0a6632.PNG">  

#### Yealink
<img width="618" alt="collaboration7" src="https://user-images.githubusercontent.com/50369643/61996702-df0d7300-b09f-11e9-8951-5fd49940f7a2.PNG">  

### SCCP  

```
show ephone registered


ephone-1 Mac:0800.2764.4F44 TCP socket:[1] activeLine:0 REGISTERED in SCCP ver 20 and Server in ver 8
mediaActive:0 offhook:0 ringing:0 reset:0 reset_sent:0 paging 0 debug:0 caps:11
IP:192.168.251.98 49642 CIPC   keepalive 74 max_line 8
button 1: dn 1  number 1000 CH1   IDLE         CH2   IDLE         

```  
<img width="326" alt="collaboration5" src="https://user-images.githubusercontent.com/50369643/61996381-6c4ec880-b09c-11e9-99d5-7f4de028db88.PNG">


## Testing calls

Call from 1113 (3CX via SIP)  to 1112 (Yealink via SIP)

<pre>
show sip-ua calls 

<b>SIP UAC CALL INFO</b>

Call 1
SIP Call ID                : 4B357B8F-2BE611D6-803FDA4B-963963A8@192.168.251.96
   <b>State of the call       : STATE_RECD_PROCEEDING (4)</b>
   Substate of the call    : SUBSTATE_PROCEEDING_PROCEEDING (2)
   Calling Number          : 1113
   Called Number           : 1112
   Bit Flags               : 0xC00018 0x100 0x280
   CC Call ID              : 44
   Source IP Address (Sig ): 192.168.251.96
   Destn SIP Req Addr:Port : 192.168.250.92:5063
   Destn SIP Resp Addr:Port: 192.168.250.92:5063
   Destination Name        : 192.168.250.92
   Number of Media Streams : 1
   Number of Active Streams: 1
   RTP Fork Object         : 0x0
   Media Mode              : flow-through
   Media Stream 1
     <b>State of the stream      : STREAM_ACTIVE</b>
     Stream Call ID           : 44
     Stream Type              : voice+dtmf (1)
     Negotiated Codec         : No Codec    (0 bytes)
     Codec Payload Type       : 255 (None)
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:17538
     Media Dest IP Addr:Port  : 0.0.0.0:0
     Orig Media Dest IP Addr:Port : 0.0.0.0:0


Options-Ping    ENABLED:NO    ACTIVE:NO
   Number of SIP User Agent Client(UAC) calls: 1

SIP UAS CALL INFO

Call 1
SIP Call ID                : Y2U0YjdmYjQwZjg5ZjVhNzg3Mzc4ODhiZGQwMzhhNjQ.
   <b>State of the call       : STATE_SENT_ALERTING (14)</b>
   Substate of the call    : SUBSTATE_NONE (0)
   Calling Number          : 1113
   Called Number           : 1112
   Bit Flags               : 0xC0001C 0x100 0x404
   CC Call ID              : 43
   Source IP Address (Sig ): 192.168.251.96
   Destn SIP Req Addr:Port : 192.168.251.88:52705
   Destn SIP Resp Addr:Port: 192.168.251.88:52705
   Destination Name        : 192.168.251.88
   Number of Media Streams : 2
   Number of Active Streams: 0
   RTP Fork Object         : 0x0
   Media Mode              : flow-through
   Media Stream 1
     <b>State of the stream      : STREAM_ADDING</b>
     Stream Call ID           : -1
     Stream Type              : voice-only (0)
     Negotiated Codec         : g711ulaw (160 bytes)
     Codec Payload Type       : 0 
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:17032
     Media Dest IP Addr:Port  : 192.168.251.88:40036
     Orig Media Dest IP Addr:Port : 0.0.0.0:0
   Media Stream 2
     State of the stream      : STREAM_DEAD
     Stream Call ID           : -1
     Stream Type              : voice+dtmf (1)
     Negotiated Codec         : No Codec    (0 bytes)
     Codec Payload Type       : 255 (None)
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:0
     Media Dest IP Addr:Port  : 0.0.0.0:0
     Orig Media Dest IP Addr:Port : 0.0.0.0:0


Options-Ping    ENABLED:NO    ACTIVE:NO
   Number of SIP User Agent Server(UAS) calls: 1

</pre>


Call from 1113 (3CX via SIP) to 1000 (CICP via SCCP)

<pre>
show sip-ua calls 
SIP UAC CALL INFO

   Number of SIP User Agent Client(UAC) calls: 0

SIP UAS CALL INFO

Call 1
SIP Call ID                : NWQ3YTVhZDk2OGE2NzU0MWQxYjFkMThjY2QwNmE2Njc.
   <b>State of the call       : STATE_SENT_ALERTING (14)</b>
   Substate of the call    : SUBSTATE_NONE (0)
   Calling Number          : 1113
   Called Number           : 1000
   Bit Flags               : 0xC0001C 0x100 0x404
   CC Call ID              : 14
   Source IP Address (Sig ): 192.168.251.96
   Destn SIP Req Addr:Port : 192.168.251.88:52705
   Destn SIP Resp Addr:Port: 192.168.251.88:52705
   Destination Name        : 192.168.251.88
   Number of Media Streams : 2
   Number of Active Streams: 0
   RTP Fork Object         : 0x0
   Media Mode              : flow-through
   Media Stream 1
     <b>State of the stream      : STREAM_ADDING</b>
     Stream Call ID           : -1
     Stream Type              : voice-only (0)
     Negotiated Codec         : g711ulaw (160 bytes)
     Codec Payload Type       : 0 
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:16522
     Media Dest IP Addr:Port  : 192.168.251.88:40018
     Orig Media Dest IP Addr:Port : 0.0.0.0:0
   Media Stream 2
     State of the stream      : STREAM_DEAD
     Stream Call ID           : -1
     Stream Type              : voice+dtmf (1)
     Negotiated Codec         : No Codec    (0 bytes)
     Codec Payload Type       : 255 (None)
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:0
     Media Dest IP Addr:Port  : 0.0.0.0:0
     Orig Media Dest IP Addr:Port : 0.0.0.0:0


Options-Ping    ENABLED:NO    ACTIVE:NO
   Number of SIP User Agent Server(UAS) calls: 1
</pre>

After picking up on CIPC  
<pre>
show sip-ua calls 
SIP UAC CALL INFO

   Number of SIP User Agent Client(UAC) calls: 0

SIP UAS CALL INFO

Call 1
SIP Call ID                : NWQ3YTVhZDk2OGE2NzU0MWQxYjFkMThjY2QwNmE2Njc.
   <b>State of the call       : STATE_ACTIVE (7)</b>
   Substate of the call    : SUBSTATE_NONE (0)
   Calling Number          : 1113
   Called Number           : 1000
   Bit Flags               : 0xC0401C 0x100 0x4
   CC Call ID              : 14
   Source IP Address (Sig ): 192.168.251.96
   Destn SIP Req Addr:Port : 192.168.251.88:52705
   Destn SIP Resp Addr:Port: 192.168.251.88:52705
   Destination Name        : 192.168.251.88
   Number of Media Streams : 2
   Number of Active Streams: 1
   RTP Fork Object         : 0x0
   Media Mode              : flow-through
   Media Stream 1
     <b>State of the stream      : STREAM_ACTIVE</b>
     Stream Call ID           : 14
     Stream Type              : voice-only (0)
     Negotiated Codec         : g711ulaw (160 bytes)
     Codec Payload Type       : 0 
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:16522
     Media Dest IP Addr:Port  : 192.168.251.88:40018
     Orig Media Dest IP Addr:Port : 0.0.0.0:0
   Media Stream 2
     State of the stream      : STREAM_DEAD
     Stream Call ID           : -1
     Stream Type              : voice+dtmf (1)
     Negotiated Codec         : No Codec    (0 bytes)
     Codec Payload Type       : 255 (None)
     Negotiated Dtmf-relay    : inband-voice
     Dtmf-relay Payload Type  : 0
     Media Source IP Addr:Port: 192.168.251.96:0
     Media Dest IP Addr:Port  : 0.0.0.0:0
     Orig Media Dest IP Addr:Port : 0.0.0.0:0


Options-Ping    ENABLED:NO    ACTIVE:NO
   Number of SIP User Agent Server(UAS) calls: 1
</pre>

