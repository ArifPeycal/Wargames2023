# See You
## Description
> "Our analyst have discovered that a file hs been compromised and transfered to another internal computer. Could you assist us in investigating this incident?"

## Challenge Overview

We were give `artifact.pcapng` to analyze. Upon inspection, we can see a huge percentage of `TCP/TLS` packets with some `HTTP` packets. From my previous experience, we need to have secret master key in order to decrypt TLS traffic. Since the challenge didnt give that file, so the solution most probably not related to `TCP`.

![image](https://github.com/user-attachments/assets/1012ece3-5a8b-4adc-bebe-f7c9d5d9014f)

Even `HTTP` didnt give much information because of `400 (Bad Request)` and `301 (Moved Permanently)` error

![image](https://github.com/user-attachments/assets/4eecce51-4b8c-47b3-aae5-e6323d981bdf)

## Solution
Looking at `UDP`, we can see a pattern where traffics come from port `38884` of `192.168.48.130` to different ports of `192.168.0.111`. This behaviour is similar to `Port Knocking`.

> The idea behind port knocking is to hide a service (like SSH or a web server) by keeping its port closed until a specific sequence of connection attempts (or "knocks") to different ports is made. When the correct sequence of knocks is received, the server temporarily opens the hidden port, allowing access to the service.
 
![image](https://github.com/user-attachments/assets/671f08e3-4e54-4288-a32b-ba479b6364f8)

We can see that the smallest destination port number is `30000`, so we can substract all destination port number with `30000` to get some ASCII value. We can create a script that extract `udp.dstport` where `ip.src` is `192.168.48.130` and `ip.dst` is `192.168.0.111`.

```py
from scapy.all import rdpcap, UDP, IP

def extract_modify_and_save_to_file(pcap_file, output_file):
    packets = rdpcap(pcap_file)
    
    with open(output_file, 'w') as f:
        for packet in packets:
            if UDP in packet and IP in packet:
                if (packet[IP].src == '192.168.48.130' and 
                    packet[IP].dst == '192.168.0.111' and
                    packet[UDP].sport == 38884):
                    
                    dst_port = packet[UDP].dport
                    
                    modified_port = dst_port - 30000
                    
                    char = str(modified_port)
                    
                    f.write(char + ' ')

pcap_file = 'artifact.pcapng' 
output_file = 'output.txt' 

extract_modify_and_save_to_file(pcap_file, output_file)

print(f"Characters have been extracted and saved to {output_file}")
```
The extracted text is decimal numbers that resembles `PNG` file. Send the value to CyberChef and we can render the image.

![image](https://github.com/user-attachments/assets/8e664b73-2764-46f4-89fe-939412cde3fc)

## Flag
```
wgmy{c0e22d2434fa188003be61e9fe404ea6}
```

### Authors Explaination
The description already mentioned about some exfiltration of  a "file" to another internal computer. So you will see some traffic to certain IP with port starting with 30XXX. The attacker use the port to exfiltrate the file and the offset is 30000.  So you can create a script to get all port  and minus with 30000. Combine all and you will get an image file with a flag.
