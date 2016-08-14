# <img src="logo.png" alt="kcptun" height="60px" /> 
[![GoDoc][1]][2] [![Release][13]][14] [![Powered][17]][18] [![MIT licensed][11]][12] [![Build Status][3]][4] [![Go Report Card][5]][6] [![Downloads][15]][16] [![Gitter][19]][20]
[1]: https://godoc.org/github.com/xtaci/kcptun?status.svg
[2]: https://godoc.org/github.com/xtaci/kcptun
[3]: https://travis-ci.org/xtaci/kcptun.svg?branch=master
[4]: https://travis-ci.org/xtaci/kcptun
[5]: https://goreportcard.com/badge/github.com/xtaci/kcptun
[6]: https://goreportcard.com/report/github.com/xtaci/kcptun
[7]: https://img.shields.io/badge/license-MIT-blue.svg
[8]: https://raw.githubusercontent.com/xtaci/kcptun/master/LICENSE.md
[9]: https://img.shields.io/github/stars/xtaci/kcptun.svg
[10]: https://github.com/xtaci/kcptun/stargazers
[11]: https://img.shields.io/badge/license-MIT-blue.svg
[12]: LICENSE.md
[13]: https://img.shields.io/github/release/xtaci/kcptun.svg
[14]: https://github.com/xtaci/kcptun/releases/latest
[15]: https://img.shields.io/github/downloads/xtaci/kcptun/total.svg?maxAge=1800
[16]: https://github.com/xtaci/kcptun/releases
[17]: https://img.shields.io/badge/KCP-Powered-blue.svg
[18]: https://github.com/skywind3000/kcp
[19]: https://badges.gitter.im/xtaci/kcptun.svg
[20]: https://gitter.im/xtaci/kcptun?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge

A tool for converting tcp stream into kcp+udp stream, :zap: ***[download address](https://github.com/xtaci/kcptun/releases/latest)***:zap:

![kcptun](kcptun.png)

***kcptun is based on [kcp-go](https://github.com/xtaci/kcp-go)***   

### *QuickStart* :lollipop:
```
Server Side: ./server_linux_amd64 -t "127.0.0.1:1080" -l ":554" -mode fast2  // forwarding to local port 1080
Client Side: ./client_darwin_amd64 -r "SERVERIP:554" -l ":1080" -mode fast2  // listening on port 1080
```

### *Performance* :lollipop:
<img src="fast.png" alt="fast.com" height="256px" />       
* Speed tested with: https://fast.com
* WAN Link Speed: 100M ADSL
* WIFI: 5GHz TL-WDR3320

### *Usage* :lollipop:
![client](client.png)
![server](server.png)

### *Applications* :lollipop:   
1. Real-time gaming.
2. Cross-ISP data exchange in PRC.
3. Other lossy network.

### *Parameters* :lollipop: 
***Both sides must agree on the following parameters:***
* datashard
* parityshard
* nocomp
* key
* crypt

other parameters can be set independently.

*How to optimize*：
> Step 1：Increase client rcvwnd & server sndwnd simultaneously & gradually。       
> Step 2：Try download something and observer, if the bandwidth usage is close the limit then stop, otherwise goto step 1.     

***NOTICE: if too much retranmission happens, it's quite possible the windows are too large***

### *Traffic Control* :lollipop: 
***Intended audience : for those server's bandwidth is quite limited.***      

Example: To limit outgoing bandwidth to 32mbit/s on server. 
```
root@kcptun:~# cat tc.sh
tc qdisc del dev eth0 root
tc qdisc add dev eth0 root handle 1: htb
tc class add dev eth0 parent 1: classid 1:1 htb rate 32mbit
tc filter add dev eth0 protocol ip parent 1:0 prio 1 handle 10 fw flowid 1:1
iptables -t mangle -A POSTROUTING -o eth0  -j MARK --set-mark 10
root@kcptun:~#
```

### *DSCP* :lollipop: 
Differentiated services or DiffServ is a computer networking architecture that specifies a simple, scalable and coarse-grained mechanism for classifying and managing network traffic and providing quality of service (QoS) on modern IP networks. DiffServ can, for example, be used to provide low-latency to critical network traffic such as voice or streaming media while providing simple best-effort service to non-critical services such as web traffic or file transfers.

DiffServ uses a 6-bit differentiated services code point (DSCP) in the 8-bit differentiated services field (DS field) in the IP header for packet classification purposes. The DS field and ECN field replace the outdated IPv4 TOS field.[1]

setting each side with ```-dscp value```.

### *Embeded Mode* :lollipop: 
Latency:     
*fast3 >* ***[fast2]*** *> fast > normal > default*        
Payload Ratio:     
*default > normal > fast >* ***[fast2]*** *> fast3*       
Parameters in middle is balanced for latency & payload ratio, the faster you get the more wasteful you are.
Manual control is supported with hidden parameters, you must understand KCP protocol before doing this.
```
 -mode manual -nodelay 1 -resend 2 -nc 1 -interval 20
```

### *Forward Error Correction* :lollipop: 
In coding theory, the Reed–Solomon code belongs to the class of non-binary cyclic error-correcting codes. The Reed–Solomon code is based on univariate polynomials over finite fields.

It is able to detect and correct multiple symbol errors. By adding t check symbols to the data, a Reed–Solomon code can detect any combination of up to t erroneous symbols, or correct up to ⌊t/2⌋ symbols. As an erasure code, it can correct up to t known erasures, or it can detect and correct combinations of errors and erasures. Furthermore, Reed–Solomon codes are suitable as multiple-burst bit-error correcting codes, since a sequence of b + 1 consecutive bit errors can affect at most two symbols of size b. The choice of t is up to the designer of the code, and may be selected within wide limits.

![reed-solomon](rs.png)

Setting parameters of RS-Code with ```-datashard 10 -parityshard 3``` 

### *Snappy Stream Compression* :lollipop: 
> Snappy is a compression/decompression library. It does not aim for maximum
> compression, or compatibility with any other compression library; instead,
> it aims for very high speeds and reasonable compression. For instance,
> compared to the fastest mode of zlib, Snappy is an order of magnitude faster
> for most inputs, but the resulting compressed files are anywhere from 20% to
> 100% bigger.

> Reference: http://google.github.io/snappy/

disable compression by setting ```-nocomp``` on both side.

> Tips: Turning off compression may reduce latency.

### *SNMP* :lollipop:
```go
// Snmp defines network statistics indicator
type Snmp struct {
	BytesSent        uint64 // payload bytes sent
	BytesReceived    uint64
	MaxConn          uint64
	ActiveOpens      uint64
	PassiveOpens     uint64
	CurrEstab        uint64
	InErrs           uint64
	InCsumErrors     uint64 // checksum errors
	InSegs           uint64
	OutSegs          uint64
	OutBytes         uint64 // udp bytes sent
	RetransSegs      uint64
	FastRetransSegs  uint64
	EarlyRetransSegs uint64
	LostSegs         uint64
	RepeatSegs       uint64
	FECRecovered     uint64
	FECErrs          uint64
	FECSegs          uint64 // fec segments received
}
```

Sending a signal by ```kill -SIGUSR1 pid``` will give SNMP information for KCP，useful for fine-grained adjustment.
Of which ```RetransSegs,FastRetransSegs,LostSegs,OutSegs``` is the most useful.

### *Donations* :dollar:
![donate](donate.png)          

All donations to this project will be used on the R&D of [gonet/2](http://gonet2.github.io/).

### *References* :paperclip:
1. https://github.com/skywind3000/kcp -- KCP - A Fast and Reliable ARQ Protocol.
2. https://github.com/klauspost/reedsolomon -- Reed-Solomon Erasure Coding in Go.
3. https://en.wikipedia.org/wiki/Differentiated_services -- DSCP.
4. http://google.github.io/snappy/ -- A fast compressor/decompressor.
5. https://www.backblaze.com/blog/reed-solomon/ -- Reed-Solomon Explained.
6. http://www.qualcomm.cn/products/raptorq -- RaptorQ Forward Error Correction Scheme for Object Delivery.
7. https://en.wikipedia.org/wiki/PBKDF2 -- Key stretching.
8. http://blog.appcanary.com/2016/encrypt-or-compress.html -- Should you encrypt or compress first?
9. https://github.com/hashicorp/yamux -- Connection multiplexing library.
10. https://tools.ietf.org/html/rfc6937 -- Proportional Rate Reduction for TCP.
11. https://tools.ietf.org/html/rfc5827 -- Early Retransmit for TCP and Stream Control Transmission Protocol (SCTP).
12. http://http2.github.io/ -- What is HTTP/2?
13. http://www.lartc.org/ -- Linux Advanced Routing & Traffic Control
