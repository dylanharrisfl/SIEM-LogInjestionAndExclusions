# SIEM - Log Ingestion and Exclusions

Identifying and excluding a honeypot from SIEM alerting during penetration testing activity.

---

I recently began deploying penetration testing efforts for a few environments that utilize honeypots I deployed on the network. These honeypots have a lot of the "attractive" ports open like 80, 23, 445, 3389, etc.

With the penetration testing doing what it is supposed to do, it attempted connections and exploits on those intentional vulnerabilities. By doing so, it generated a large amount of alerts in our newer SIEM tool. I needed to better understand how the tool works and what I could do to exclude these attempts from alerting during penetration tests.

So I did a little bit of research and investigation. I found the built-in custom detection rule for honeypot connections within the SIEM platform and looked at the query string it uses to search for these specific logs:

```
message_raw: "New connection" || message_raw: "\"action\"\: \"connection\"" || message_raw: "connection received"
```

I knew this was similar to an Elasticsearch type of query, so I pulled the raw log from one of the original detections to get the format:

```
<30>May 25 18:15:49 avm mysql-honeypot[714]: ["action": "connection", "dest_ip": "0.0.0.0", "dest_port": "3306", "server": "mysql_server", "src_ip": "x.x.x.x", "src_port": "38003", "timestamp": "2026-05-25T18:15:49.951400"]
```

I tried using an `AND NOT` function to exclude a `src_ip`. It didn't seem to understand this, so I looked a little deeper. When testing the functionality of the rule, I noticed you could filter results via a `source_address` field instead.

I tried my query string with `source_address` and it worked. I could now exclude the penetration testing IP. I ran a test query to see what logs the SIEM would alert on in the past 24 hours with the exclusion applied and there were none. To confirm further, I removed my machine from the exclusion and saw all the alerts originating from the penetration testing IP were now being flagged as expected. Here is what the final query looked like:

```
(message_raw: "New connection" || message_raw: "\"action\"\: \"connection\"" || message_raw: "connection received") AND NOT source_address: "x.x.x.x"
```

> **Note:** `x.x.x.x` is a placeholder for the actual IP used in the real environment.

It appears this tool normalizes log ingestion in its own way, so when querying results you need to use their specific field names rather than the raw log's key-value pairs. `src_ip` exists in the raw log, but the SIEM exposes it as `source_address` after normalization.

I can now deploy this to any similar environment, reducing false positives and allowing my team to focus on the data that actually matters.



 
