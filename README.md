# VPC-Transit-Gateway


## Objective


The VPC Transit Gateway project focused on implementing and analyzing multi-region cloud networking architecture in AWS. The primary objective was to establish centralized connectivity between distributed VPCs while leveraging flow logs to monitor inter-VPC communication patterns. This hands-on implementation was designed to enhance understanding of cloud network design, traffic routing optimization, and scalable infrastructure management.


### Skills Learned


- Advanced understanding of AWS networking architecture and Transit Gateway concepts.
- Proficiency in analyzing and interpreting VPC flow logs and traffic patterns.
- Ability to design and implement multi-region cloud connectivity solutions.
- Enhanced knowledge of VPC peering and centralized routing strategies.


### Tools Used


- AWS Transit Gateway for centralized VPC connectivity management.
- AWS CloudWatch Logs for capturing and storing VPC flow logs.
- CloudWatch Insights for analyzing and visualizing network traffic patterns.

## Steps

*Ref 1: Network Diagram displaying our traffic flow* <img width="926" height="1007" alt="Screenshot 2025-08-06 145010" src="https://github.com/user-attachments/assets/aa19f9e0-2c79-4347-a744-a3654362ead7" />

*Part 1: Creating VPC Architecture* VPC Architecture- I will set up a multi-VPC Architecture across 3 regions. Each VPC will have a public subnet and contain an EC2 instance with a route table and IGW.*

    1. First VPC in east1(N.Virginia) IP range is 12.0.0.0/16 so we have aroud 65k IP ranges available.
    2. Then I create an IGW for the VPC east-1. igw-my-vpc-east1
    3. Attach IGW to its corresponding VPC
    4. Create our subnet- choose corresponding VPC and add AZ suffix(1a). For our subnet CIDR block change it to 12.0.1.0/24 this gives us 256 IPs. This gives us a public subnet subnet-my-vpc-east1-1a
    5. Create Route Table & populate routes- chose corresponding VPC. Associate VPC with route table.
        1. create routes: 0.0.0.0 destination and igw target(this gives us access to the internet from anywhere) route-table-my-vpc-east1
<img width="1920" height="900" alt="Screenshot 2025-07-31 113037" src="https://github.com/user-attachments/assets/0b153100-ba70-4788-b905-e552b5209e6c" />
    6. Deploy EC2 Servers: key pair name- ec2-my-vpc-east1-key Save key save key as ppk 
       <img width="1920" height="1128" alt="Screenshot 2025-07-31 113736" src="https://github.com/user-attachments/assets/b6f06f12-ca87-4ac2-8d9a-6026ca68e859" />
      b. Network Settings- choose corresponding VPC/subnet, auto assign public IP because we are using a public subnet, Add HTTP/S SG rule and ICMP Security Group rule (both from anywhere) secgroup-my-vpc-east1
       <img width="1920" height="1128" alt="Screenshot 2025-07-31 120741" src="https://github.com/user-attachments/assets/a87e87f9-fd15-48a5-a157-6789719be4be" />
    
    7. Repeat the Steps for the rest of your VPCs
    8. Creating Transit Gateway:  tg-my-vpc-east1  **make sure to auto-accept shared attachments so our VPCs can be peered*
<img width="1920" height="1128" alt="Screenshot 2025-07-31 154450" src="https://github.com/user-attachments/assets/4a81ccb3-2786-4abe-bb3a-ab3c17a89e8f" /> 
        i. Create TG Attachments: tg-attachment-my-vpc-east1   —choose TG ID and attachment type(VPC). -whenever Transit Gateway receives a packet from an attachment, it performs a route lookup operation on the route table associated with that attachment.  A TGW attachment is a connection point that links different network resources to an AWS Transit Gateway(in my demo i have VPC attachments and peering attachments)
        <img width="1920" height="1128" alt="Screenshot 2025-07-31 155407" src="https://github.com/user-attachments/assets/4cc421e6-be0d-4776-aa81-53f4746a987e" />
        ii. repeat for each region
    9. Peer Transit Gateways together: choose the two VPCs you would like to peer conect tg-peering-vpceast1-vpceast2
<img width="1920" height="1128" alt="Screenshot 2025-08-01 114642" src="https://github.com/user-attachments/assets/7cba5771-b22b-413d-8923-2d8ab1b15639" />
        1. Make sure to accept the peering attachment in your acceptor VPC
        2. Create a static route in the acceptor TGW route table, add the CIDR range from sender VPC and choose the peering attachment. We do this because Transit Gateway route tables are not automatically routed and traffic will not flow without specific routes
        <img width="1920" height="1128" alt="Screenshot 2025-08-01 120056" src="https://github.com/user-attachments/assets/d776ad26-1870-4fdf-885f-46d5c2628901" />
        3. create a static route in the sender TGW route table and add CIDR of acceptor VPC and choose the peering attachment 
        <img width="1920" height="1128" alt="Screenshot 2025-08-01 120221" src="https://github.com/user-attachments/assets/bc31d986-5c35-49c9-be56-9e634723fd0c" />
        4. Edit route table of the sender VPC to include CIDR of acceptor VPC, repeat for second region
        <img width="1920" height="1128" alt="Screenshot 2025-08-01 121007" src="https://github.com/user-attachments/assets/2e6a5b83-f743-4043-8cf3-3625c8ebbadc" />
        5. Peering is NOT transitive, we must peer A-B, A-C, and B-C for full connectivity.
        6. verify you can ping your instance in VPC 2 from your instance in VPC 1 
        <img width="1920" height="1128" alt="Screenshot 2025-08-01 123713" src="https://github.com/user-attachments/assets/b8c1cb95-11da-4bcd-84ff-75d5312ef13f" />

*Part 2: Logging- I will use the VPC Transit GW Flow Logs as well as Cloud Watch Logs to capture network traffic across my architecture*
    1. Create a Cloud Watch log group for our traffic my-vpc-east1-log-group
    2. Create our IAM role and Trust relationship for VPC flow logs  vpc-tgw-flow-logs-cw. By default users do not have permissions to work with flow logs — Our IAM role is trusting the VPC flow logs service and is allowing the service to interact with CloudWatch Logs
    <img width="1920" height="1128" alt="Screenshot 2025-08-04 112455" src="https://github.com/user-attachments/assets/a96631d8-b72e-40c2-b316-75d86ecf7a44" />
    3. Create the permission policy for our role vpc-tgw-flow-logs-cw-policy
    <img width="1920" height="1128" alt="Screenshot 2025-08-04 112727" src="https://github.com/user-attachments/assets/8daca841-7856-4410-ad33-754162836a3f" />
    4. Create a flow log for our VPC (repeat for all VPCs) my-vpc-east1-flow-log 
        i. we want our interval to be every minute and to send logs to cloud watch, choose our log group as the destination, and choose our role as the service role
        <img width="1920" height="1128" alt="Screenshot 2025-08-04 115136" src="https://github.com/user-attachments/assets/ff848dad-ac70-4d64-bba6-25d86b33e80e" />
        ii. View some the created logs, generate traffic to create more logs
            a. we can run some commands to create traffic among our instances
            b. while true; do ping -c 10 [OTHER_EC2_IP]; sleep 5; done (this will send 10 packets, wait for 5 seconds and then repeat continuously)
            c. ping -s 1000 [OTHER_EC2_IP] -c 100 ( this will send 100 pings each of size 1000 bytes to stress the network)
            d. while true; do curl -s http://[EC2_IP_REGION2]/; sleep 1; done (this will make continuous HTTP request to an EC2 instance)
            e. wrk -t2 -c100 -d30s http://[EC2_IP_REGION2]/ (this will use 2 threads to send maintain 100 concurrent HTTP connections for 30 seconds)


*Part 3: Visualization/Analysis*
    1. View our log metrics in CloudWatch Dashboard,  by default CloudWatch gives us insights into the number of bytes in/out as well as the number of packets in/out
    2. We are able to see that our Security Groups have rejected some traffic
    <img width="1920" height="1128" alt="Screenshot 2025-08-08 111752" src="https://github.com/user-attachments/assets/73caec0c-c359-4b63-9c47-b2cfb24163e9" />
    3. An example of an ACCEPT packet showing HTTP traffic across AZ’s
    <img width="1920" height="1128" alt="Screenshot 2025-08-08 112432" src="https://github.com/user-attachments/assets/3b964299-5356-4e56-a861-5146805129bf" />
    4. By default TGW dashboard gives us visibilty into the bytes in/out as well as the packets in/out. 
    <img width="1920" height="1128" alt="Screenshot 2025-08-08 120550" src="https://github.com/user-attachments/assets/24adbb92-187a-4835-8c4a-442d84a99968" />
    5. Going further we can write a query in Cloudwatch that visualizes the flow logs by their action and protocol 
        i. stats sum(bytes) as totalBytes, count(*) as flowCount, count(distinct srcaddr) as uniqueSources, count(distinct dstaddr) as uniqueDestinations by bin(1h) as hourly, action, protocol | sort hourly asc | limit 1000
        <img width="1920" height="1128" alt="Screenshot 2025-08-08 143307" src="https://github.com/user-attachments/assets/223b3a8e-48ee-48e6-a218-47e9ab785c38" />
    6. CloudWatch also has built in queries we can use- this one gives us visibility into the top 20 source IP addresses with highest number of rejected requests.
    <img width="1920" height="1128" alt="Screenshot 2025-08-08 144738" src="https://github.com/user-attachments/assets/58795bfe-d73a-451a-96a5-d4791f8c9842" />
    <img width="1920" height="1128" alt="Screenshot 2025-08-08 144754" src="https://github.com/user-attachments/assets/f88c017b-ca34-47b1-adf7-911477e123f3" />
    7. These visualizations from our flow logs are very insightful and can further help us monitor our network and update our security if need be. 

*Part 4: Going Beyond* Am I able to give better visibility into my network traffic?

    1. Create an S3 bucket to export our CloudWatch data into by using a firehose data stream
    2. Configure Athena tables and queries on our log data
    3. Connect Athena to Quicksight and import our data set
    4. Design a Quicksight dashboard for further visibility
    
    
    
    Questions to Consider:
    
    1. What are the best practices for securing inter-VPC communication?
        1. When creating our security group/ NACL rules we can choose to only allow traffic from the CIDR ranges of our VPCs to reduce the chances of outside traffic coming in. 
    2. How does Transit Gateway improve upon traditional VPC peering for complex networks?
        1. Traditional peering allows for a 1:1 connection, which can become complex when there are multiple VPCs. TGW allows fo us to create a centralized hub for many VPCs under a region. We can peer regions together to create a global network
    3. How can you use flow logs to identify potential security issues or network bottlenecks?
        1. Flow logs give us visibility into any potential unusual ports or protocols that are being utilized on our network. We can also search for high traffic spikes and analyze connection failures. Flow logs can be integrated with CloudWatch Insights, Athena, Splunk, DataDog or other 3rd party SIEM’s. A 3rd party tool such as Kentik can also be implemented for better visualization into your network traffic
    
    TAKEAWAYS
    Transit Gateway is limited by a regional scope. You can connect many VPCs within the same region. BUT if you want to connect VPCs accross regions each region will need its own TGW and then you can peer the TGWs together accross the region level. 
    
    
    
    
    
    
    
    
    
    
    
    





















  

