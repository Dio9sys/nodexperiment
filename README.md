# nodexperiment
An experiment in monitoring network traffic on a tor relay.

### OBJECTIVE

I wanted to see what the network traffic would look like on a tor relay in contrast to a computer with minimal traffic.

I also wanted to learn more about how wireshark works, how tor operates and I wanted to write a python script that could
give me statistics on the kinds of traffic I was seeing.

Watch this space; that script will be here as soon as it's finished!

### METHODOLOGY

This experiment was run on an Ubuntu Server 19.10 virtual machine being hosted on Windows 7 through VirtualBox 6.1.12.

The control group was established by monitoring traffic for 24 hours without any user-generated web traffic other than
running `curl http://www.textfiles.com` to verify that traffic was being monitored.

The experimental group was established by establishing the server as a tor middle relay which, for the sake of transparency,
was listed under the moniker `NootNootThePlanet` and with a contact address at a disposable email address to avoid getting
spam on my email address.  This tor relay was configured through the `/etc/tor/torrc` file to run as an uncapped non-exit relay.
This experimental tor relay was run for approximately 3 hours, as verified by a timer I set after I started monitoring traffic.

In both cases, the traffic was monitored by running `tshark -i enp0s3 >` and then either `control.txt` or `experimental.txt`
after ensuring that the enp0s3 virtual ethernet port was set to monitoring mode through `airmon-ng`

### EXPERIMENTAL COMPLICATIONS

Originally, I was going to establish a tor bridge, under the idea that the data would be more interesting and my IP address
would then not be listed publicly.  However, that proved problematic when I ran into complications getting tor and obfs to
work in tandem, and it was decided that I could get more data faster by just setting up a standard relay.  There were also
ethical concerns regarding interpreting data from a relay traditionally set up to further obscure users in oppressive governments.

### ETHICAL CONCERNS

The tor network is a widely used service; while in the US it is commonly associated with hackers, pirates and drug dealers, it
is equally important in helping individuals under oppressive governments or under targeted surveillance to communicate with the
greater internet.  This is why it's a popular choice among human rights activists and whistleblowers.

As such,the full data from this experiment will not be publicly releaseed.  Throughout this file, snippets of the data will
shown with obscured IP addresses, to preserve the privacy of other relays and bridges on the network.

If you are interested in knowing aggregate data for the entire network, you can find that information at https://metrics.torproject.org/bandwidth.html.

### EXPERIMENTAL RESULTS

#### Control data

First, let's look and see what my server looks like on a regular day.  It doesn't get a lot of traffic because I don't use it for much:

```
    1 0.000000000    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx DNS 88 Standard query 0xe2f8 AAAA www.textfiles.com OPT
    2 0.114351530 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    DNS 142 Standard query response 0xe2f8 AAAA www.textfiles.com SOA dns1.easydns.com OPT
    3 0.151101839    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TCP 74 59256 → 80 [SYN] Seq=0 Win=64240 Len=0 MSS=1460 SACK_PERM=1 TSval=1079793420 TSecr=0 WS=128
    4 0.238947358 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 60 80 → 59256 [SYN, ACK] Seq=0 Ack=1 Win=65535 Len=0 MSS=1460
    5 0.239006393    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TCP 54 59256 → 80 [ACK] Seq=1 Ack=1 Win=64240 Len=0
    6 0.239258956    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx HTTP 135 GET / HTTP/1.1 
```

(note, everywhere that is showed as `xxx.xxx.xxx.xxx` is an IPv4 address that I've redacted)

So, what does this mean?

First, you can see my pc communicating with a DNS server to see if www.textfiles.com is a real website that exists and, if so, where to find it.

Then it gives an answer from the easydns server showing where to find the IP address for the website.

After that, you see three TCP communications: SYN, SYN ACK, and ACK.  This is the 3-way handshake that TCP does to verify that you're on a functional connection.  The machine making the request sends ACK, the machine responding sends SYN ACK, and then the initial machine sends ACK ACK (in this case, shortened to ACK) to verify that communication works both ways.

After the TCP handshake, you see an HTTP 135 GET request.  This is my machine asking the textfiles server to send whatever data is has for public consumption to my machine.  Because I sent the request through Curl, it was at this point that I received a print out of the HTML source for Textfiles.  Had I been using a browser, the browser would have then interpreted that data and displayed the website on my screen.

But wait, there's more!

We've seen what IPv4 looks like.  Let's see what IPv6 looks like.
```
   18 477.535294239 xxxx::xxxx:xxxx:xxxx:xxxx → xxxx::xxxx      ICMPv6 70 Router Solicitation from xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
   19 477.535391845 xxxx::xxxx:xxxx:xxxx:xxxx → xxxx::xxxx:xxxx:xxxx ICMPv6 86 Neighbor Solicitation for :: from xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
```
(any addresses of xxxx:xxxx:xxxx:xxxx:xxxx:xxxx are redacted IPv6 addresses)

This is different from the traffic we had seen before.  This is ICMP traffic doing two things.
First, Another router in the area has checked with my router to see if it still exists.
Then, we received a neighbor solicitation.  Neighbor solicitation is used for machine-to-machine communication, to make sure that the link layer address of my machine is still the same one that my neighbor previously had saved.  If mine has changed, then my machine would send the new address to the neighbor so they could route traffic as needed.

### Experimental data

In contrast, let's see what some of the experimental data looks like.  In 3 hours, I captured 20,949 packets.

```
    1 0.000000000 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TLSv1.2 597 Application Data
    2 0.000697147    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TLSv1.2 597 Application Data
    3 0.001044727 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 60 443 → 37176 [ACK] Seq=544 Ack=544 Win=65535 Len=0
    4 0.220041162 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TLSv1.2 597 Application Data
    5 0.220489730    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxxTLSv1.2 597 Application Data
    6 0.220723765 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 60 443 → 37176 [ACK] Seq=1087 Ack=1087 Win=65535 Len=0
    7 0.590988029 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TLSv1.2 597 Application Data
    8 0.591260675    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TLSv1.2 597 Application Data
    9 0.591581756 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 60 443 → 37176 [ACK] Seq=1630 Ack=1630 Win=65535 Len=0
   10 0.844714641    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TLSv1.2 597 Application Data
```

The first thing that stands out is the sheer speed at which packets came in.  After each line number is the time after initial scanning that the packet came in.  The first 10 requests were all sent within one second.  This is why the tor project recommends placing a cap on your relay if you're running on a home network.  A tor relay regularly sees orders of magnitude more traffic than a standard computer connected to the internet.

The other thing to notice is the nature of the requests.  You can't tell because of the redacted IP addresses, but each pair of TLS Application Data requests comes first from a separate relay to my machine, and then from my machine to the separate relay.  After this communication is done over port 597, we then see the separate relay send an ACK over TCP port 443, which is generally reserved for HTTPS data.

From this, we can reasonably surmise that the other connections on the tor network are checking over port 597 to see if our machine is still part of the network and, after verifying, they then send a TCP request over SSL with the data that needs sent along the network.

Let's go futher in this file and see what happens later.  This connection with the other node is maintained for a little over 1000 packets, before we get to some ubuntu update DNS queries, and then this:
```
10568 1775.243887323 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 1514 80 → 45702 [PSH, ACK] Seq=224841 Ack=145 Win=65535 Len=1460 [TCP segment of a reassembled PDU]
10569 1775.243902513    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TCP 54 45702 → 80 [ACK] Seq=145 Ack=226301 Win=65535 Len=0
10570 1775.244900743 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 2974 80 → 45702 [PSH, ACK] Seq=226301 Ack=145 Win=65535 Len=2920 [TCP segment of a reassembled PDU]
10571 1775.244918022    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TCP 54 45702 → 80 [ACK] Seq=145 Ack=229221 Win=65535 Len=0
10572 1775.245246732 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 1514 80 → 45702 [PSH, ACK] Seq=229221 Ack=145 Win=65535 Len=1460 [TCP segment of a reassembled PDU]
10573 1775.245257391    xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx TCP 54 45702 → 80 [ACK] Seq=145 Ack=230681 Win=65535 Len=0
10574 1775.245425564 xxx.xxx.xxx.xxx → xxx.xxx.xxx.xxx    TCP 1514 80 → 45702 [PSH, ACK] Seq=230681 Ack=145 Win=65535 Len=1460 [TCP segment of a reassembled PDU]
```

What does all this mean?

First, we should note that these are all from the same back and forth between my machine and a single other relay.  You can see communication over TCP ports 54 and 45702 to TCP port 80 and vice versa.  You see the remote relay sending us PSH and ACK requests, and you see our response of ACK back to the remote relay.

What does [TCP segment of a reassembled PDU] mean?

Wireshark generally tries to guess at what any given packet is and what it's doing.  Sometimes a packet will be too large to send over TCP and will get broken into segments, and then the segments get sent separately and reassembled later.  TCP segment of a reassembled PDU likely means that someone was sending data too large to fit in a standard TCP packet - maybe downloading a file, or playing an online game - and the data got broken apart before being sent over the tor network to be reassembled by the end user machine.

## Conclusions

From this experiment, I learned a few things:

1. I learned how better to work with wireshark and its terminal-based port, tshark
2. I learned how to set up a tor relay
3. I gained a greater understanding and appreciation of how tor works on layer 3
4. I gained a general understanding of what tor traffic looks like and how to recognize if a machine is being used as a relay.

Final conclusion: it may be possible to identify if a machine is being used as a tor relay based on total kind and volume of network traffic.  This may be applicable in cases of network troubleshooting at home (is your roomate running a relay in her room and forgot to tell the group chat?), or in cases of shadow IT in the office, such as someone turning a testing server into a tor relay without proper authorization.

## Further research

I plan on writing a python script that can read wireshark text output and give statistics on unique connections and requests sent.  I would also be interested in cross-referencing the IP addresses in the data gathered with the public list of tor relays to see who all I was connecting to, and where they were located geographically.
