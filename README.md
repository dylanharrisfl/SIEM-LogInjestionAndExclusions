# SIEM-LogInjestionAndExclusions
I identified a Honey Pot that needed to be excluded from SIEM alerting when attempts occur from the source IP address of a penetration testing device.


I recently began deploying Penetration Testing efforts for a few environments that currently utilize honeypots I deployed on the network.

These honeypots have a lot of the 'attractive' ports open - like 80, 23, 445, 3389, etc.
With the Penetration Testing doing what it is supposed to do, it attempted connections and exploits on intentional vulnerabilities on those ports. By doing so, it generated a large amount of alerts in our newer SIEM tool.

I needed to better understand how said tool works, and what I can do to exclude these attempts from alerting during Penetration Tests.

So I did a little bit of research and investigation. I found the built in custom detection rule for honeypot connections within the SIEM platform. I saw the query string they utilize to search for these specific logs, being:

message_raw: "New connection" || message_raw: "\"action\"\: \"connection\"" || message_raw: "connection received"


I knew this was similar to an Elasticsearch type of query, so I found the raw log of one of the original detections to get the format:


Raw Log: <30>May 25 18:15:49 avm mysql-honeypot[714]: ["action": "connection", "dest_ip": "0.0.0.0", "dest_port": "3306", "server": "mysql_server", "src_ip": "192.168.100.79", "src_port": "38003", "timestamp": "2026-05-25T18:15:49.951400"]


I tried to then utilize an ‘AND NOT’ function to exclude a “src_ip”. It didn’t seem to understand this so I looked a little deeper. When testing the functionality of the rule, I saw that you could filter results via a "source_address”.


So I tried my query string with “source_address” instead, and it was successful. I could now exclude the Penetration Testing IP. By utilizing this AND NOT funciton to exclude this IP, I ran a test query to see what logs the SIEM would alert on in the past 24 hours. There were none. To test and confirm further, I excluded any alerts from only my computer and saw all the alerts that originated from Penetration Testing IP were now being flagged (as it was no longer excluded from alerting). Here is what the final string looked like:


(message_raw: "New connection" || message_raw: "\"action\"\: \"connection\"" || message_raw: "connection received") AND NOT source_address: "x.x.x.x"

***Note - the x.x.x.x is just a placeholder for the actual IP in this writeup. In the real environment an actual IP address was utilized.

It appears as though this tool normalizes their log injestion in their own way, so when you are querying results you need to specifically use their language to effectively query it, rather than the raw log data's key-value pair data.

I could now deploy this to any similar environment, reducing false positives and allowing my team to focus on valuable data.

 



 
