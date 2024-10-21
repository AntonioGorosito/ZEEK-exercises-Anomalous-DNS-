# ZEEK-Exercises-Anomalous-DNS

## Objective
The objective of this scenario is to identify irregularities in DNS traffic that may signal potential attacks, allowing for timely intervention and enhanced network security.

## Steps
1- The first thing I do is create Zeek logs from the PCAP file to begin investigating the alert. In this case, the alert indicates "Anomalous DNS Activity." I use the command:

"zeek -C -r dns-tunneling.pcap"

This command runs Zeek on the specified PCAP file, which contains suspicious DNS traffic. The '-C' option is used to disable checksum verification, while the '-r' option allows reading directly from the PCAP file instead of capturing live traffic.
Once Zeek processes the file, it generates multiple log files, such as 'http.log', 'conn.log', 'dns.log', and others that contain key information about connections, HTTP requests, and, of course, DNS queries. This allows me to inspect in more detail the events related to the original alert. The 'dns.log' file is particularly relevant here, as it contains all the DNS queries from the captured traffic.

![Screenshot 2024-10-12 005022](https://github.com/user-attachments/assets/d7eb4099-6f2e-4128-976e-def8766ddc07)

2- Then, we use the 'cat' command on the dns.log file to read what was generated:

"cat dns.log"

This command displays the entire content of the 'dns.log' file, which can be useful for an initial view. However, in many cases, when the file contains a large amount of data, reading it all at once can be unnecessary and overwhelming.

![Screenshot 2024-10-12 005230](https://github.com/user-attachments/assets/3b3416a0-f4d7-45d9-91a9-0e1662c6a0d9)
![Screenshot 2024-10-12 005348](https://github.com/user-attachments/assets/3f44d5f6-d4dd-4304-864c-52a1578a0bf6)

3- Since there is a lot of information, and I'm only interested in obtaining a small portion for a quick view, I use the following command:

"cat dns.log | head -n 10"

This command displays only the first 10 lines of the dns.log file. This way, I can get a quick sample of the content without being overwhelmed by all the records. The 'head -n 10' command is very useful for limiting the amount of information to review, as it allows me to see a controlled part of the file, which is ideal in this case where I want a preliminary view before delving into more detailed analysis.

![Screenshot 2024-10-12 010602](https://github.com/user-attachments/assets/b866234c-4ade-46ea-afb9-20ccd058dff5)

The image shows a preliminary and structured view of the DNS logs, which is an important step in analyzing anomalous activities on the network.

4- Next, I want to see the number of DNS records associated with IPv6 addresses. For this, I use the following command:

"cat dns.log | zeek-cut qtype_name"

This command extracts only the types of DNS queries (such as A, AAAA, CNAME, etc.) from the dns.log file using 'zeek-cut' with the 'qtype_name' column. This gives me a list of all the types of DNS records present in the file, but it only shows the type of query without applying any specific filtering

![Screenshot 2024-10-12 011025](https://github.com/user-attachments/assets/c94bb93a-f17a-4005-babe-1c7381ed82e9)

as a result i get this:

![Screenshot 2024-10-12 011234](https://github.com/user-attachments/assets/1b4ae598-ce9c-41fd-9558-9d1396ea94a9)

5- As a result, it provides a large amount of information, but it is not organized or filtered to give me an accurate count of how many records are specifically associated with IPv6 addresses (which in this case are the AAAA record types). To do this more accurately and organized, I used the following command:

"cat dns.log | zeek-cut qtype_name | sort | uniq -c"

This command does the following:

'sort' organizes the output alphabetically.
'uniq -c' counts how many times each type of query appears in the file, grouping the results by each type of DNS record.

In the end, this command returns an exact count of how many DNS records are linked to IPv6 addresses (that is, those that are of type AAAA), along with the count of other types of records, such as A, CNAME, and others.

![Screenshot 2024-10-12 011357](https://github.com/user-attachments/assets/17bdf255-5087-4ae5-9d3a-3e7b15598557)

6- Next, I want to see the longest connection duration. To achieve this, I use the following command:

"cat conn.log | zeek-cut duration | sort | uniq"

This command does the following:
'zeek-cut' duration extracts the duration column from the conn.log file, which contains the duration of each connection recorded in the file.
'sort' organizes the durations in ascending order.
'uniq' removes duplicate values so that each duration appears only once.

![Screenshot 2024-10-12 013302](https://github.com/user-attachments/assets/f78fb71e-849b-4a5a-b1a2-0f3922e1ad60)

As a result, I obtain a list of the durations of all connections, from the shortest to the longest. Here is the result I get:

![Screenshot 2024-10-12 013403](https://github.com/user-attachments/assets/5354c64b-1269-4e9c-a00d-c681a4dbaa98)

And in the end, it can be observed that the longest duration of all connections was 9.420791 seconds.

7- Then I want to find out the IP address of the source host. First, I identify the field that contains this information, which is 'id.orig_h'. This field stores the source IP address in the logs generated by Zeek.

![Screenshot 2024-10-12 015218](https://github.com/user-attachments/assets/431b17cf-8ad3-4cc1-8e5c-83a8bfa59357)

To find the source IP address, I use the following command:

"cat conn.log | zeek-cut id.orig_h"

This command:
Extracts the 'id.orig_h' column from the conn.log file, where the source IP addresses of all connections are recorded.
This allows me to quickly identify which source IP address is associated with the suspicious traffic.

![Screenshot 2024-10-12 015329](https://github.com/user-attachments/assets/5783136e-53c7-4ed3-b5ff-9b715161690f)

Conclusion:

After analyzing the logs, anomalous DNS activity was detected. Based on the obtained data, we were able to extract the following information:

A total of 320 DNS records were found linked to IPv6 addresses.

The maximum connection duration was 9.42 seconds.

After filtering all unique DNS queries, it was determined that the total number of unique domain queries was 6.

Finally, the source IP associated with this anomalous activity was identified as 10.20.57.3.
