# Tshark-Extracting Files From A Pcap

In this activity I was working with the network analysis tool Tshark, on the Infosec Institute platform to analyze packets and network traffic flows.<br>
<br>
<b>Step 1:</b><br>
<br>
First we'll take a look at a pcap file to see if we can identify anything that is worth further inspection. We will run the command: tshark -r http.cap -Y http.
After running the command we can see that packet 27 contain html/text and packet 38 with the XML file, which may be interesting.<br>
<br>
<a href="https://imgur.com/6g2Qh9T"><img src="https://i.imgur.com/6g2Qh9T.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 2:</b><br>
<br>
Next we'll use the http content filter to only disply packets tjat are carrying files. Command: tshark -r http.cap -Y http.content_type.<br>
<br>
<a href="https://imgur.com/mjemkcY"><img src="https://i.imgur.com/mjemkcY.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 3:</b><br>
<br>
So we've identified packet 27 as being of possible interest, now we're going to try to explore its contents by using verbose mode. Command: tshark -r http.cap -V -Y frame.number==27. The image below is truncated, and the packet displays part of a webpage, but we need to use a different method to see if there is anything useful here.<br>
<br>
<a href="https://imgur.com/shweiOG"><img src="https://i.imgur.com/shweiOG.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 4:</b><br>
<br>
Next we'll try a full hex dump. By doing this we can filter out all of the packet headers and display everything in raw bits making it possible to extract full files. The "x" flag is for a hex dump. Command: tshark -r http.cap -x -Y frame.number==27.<br>
<br>
<a href="https://imgur.com/VSpLOe5"><img src="https://i.imgur.com/VSpLOe5.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 5:</b><br
<br>
Next we're going to use a command to try and dump any objects from this traffic capture into a directory. Command: tshark -r http.cap -q --export-objects http,./dumped. After running the command and checking the folder, we can see a file called download.html, but it seems to be benign. I suppose this is an exercise in patience.<br>
<br>
<a href="https://imgur.com/TeJGjyt"><img src="https://i.imgur.com/TeJGjyt.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 6:</b><br>
<br>
We will be looking at another pcap now that includes a multi-stage malware infection. We will start off by looking at the http.content_type. Right away we see an MS Word file in packet 323. Malware in Word files are often attributed to macros or a downloader.<br>
<br>
<a href="https://imgur.com/CwEPbZD"><img src="https://i.imgur.com/CwEPbZD.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 7:</b><br>
<br>
We're going to run tshark in verbose mode on that packet to see if we can garner a any more information. Command: tshark -r multistage.pcap -V -Y frame.number==323. In the output we can see the attachment filename which is, filename=USPS_invoice_reggie.cage.doc. <br>
<br>
<a href="https://imgur.com/JRdvaP4"><img src="https://i.imgur.com/JRdvaP4.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 8:</b><br>
<br>
The file in question is spread out across many packets, and dump all of the objects from this packet capture may be unnecessary. So we're going to try to zero in on the TCP stream so we can filter out unnecessary information. Command: tshark -r multistage.pcap -V -Y frame.number==323 | grep Stream. The stream index is 0. <br>
<br>
<b>Step 9:</b><br>
<br>
Now that we know the stream index, we're going to use the "z flag" and "follow" to get an ASCII printout of the stream. Command: tshark -q -r multistage.pcap -z follow,tcp,ascii,0 | more. <br>
<br>
<a href="https://imgur.com/jYQUbnq"><img src="https://i.imgur.com/jYQUbnq.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 10:</b><br>
<br>
Next we're going to chain commands together to filter out packet 323 and output and while immediatly reading it again. Command: tshark -r multistage.pcap -w - -Y frame.number==323 | tshark -r -.<br>
<br>
<a href="https://imgur.com/RIVS1r1"><img src="https://i.imgur.com/RIVS1r1.png" title="source: imgur.com" /></a><br>
<br>
<b>Step 11:</b><br>
<br>
Next we're going to chain commands again to print only the file included in the TCP session and then dump it. Command: tshark -r multistage.pcap -w - -Y tcp.stream==0 | tshark -r - -q --export-objects http,docs.<br>
<br>
<a href="https://imgur.com/Rd9MWjp"><img src="https://i.imgur.com/Rd9MWjp.png" title="source: imgur.com" /></a><br>
<br>
<b>Step  12:</b><br>
<br>
Lastly, the file that we dumped has a differnt name, we're going to check it to see if it is the original.<br>
<br>

