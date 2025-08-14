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

*Part 1: Creating VPC Architecture* VPC Architecture- I will set up a multi-VPC Architecture across 3 regions. Each VPC will have a public subnet and contain an EC2 instance with a route table and IGW.

    1. First VPC in east1(N.Virginia) IP range is 12.0.0.0/16 so we have aroud 65k IP ranges available. 
    2. Then I create an IGW for the VPC east-1. igw-my-vpc-east1
    3. Attach IGW to its corresponding VPC
    4. Create our subnet- choose corresponding VPC and add AZ suffix(1a). For our subnet CIDR block change it to 12.0.1.0/24 this gives us 256 IPs. This gives us a public subnet subnet-my-vpc-east1-1a
    5. Create Route Table & populate routes- chose corresponding VPC. Associate VPC with route table. 
        1. create routes: 0.0.0.0 destination and igw target(this gives us access to the internet from anywhere) route-table-my-vpc-east1*
      
    
    <img width="1920" height="1128" alt="Screenshot 2025-07-31 113037" src="https://github.com/user-attachments/assets/0b153100-ba70-4788-b905-e552b5209e6c" />
    *6. Deploy EC2 Servers: key pair name- ec2-my-vpc-east1-key Save key save key as ppk *
       <img width="1920" height="1128" alt="Screenshot 2025-07-31 113736" src="https://github.com/user-attachments/assets/b6f06f12-ca87-4ac2-8d9a-6026ca68e859" />
      *b. Network Settings- choose corresponding VPC/subnet, auto assign public IP because we are using a public subnet, Add HTTP/S SG rule and ICMP Security Group rule (both from anywhere) secgroup-my-vpc-east1*
       <img width="1920" height="1128" alt="Screenshot 2025-07-31 120741" src="https://github.com/user-attachments/assets/a87e87f9-fd15-48a5-a157-6789719be4be" />
    
    *7. Repeat the Steps for the rest of your VPCs
    8. Creating Transit Gateway:  tg-my-vpc-east1  **make sure to auto-accept shared attachments so our VPCs can be peered*
    <img width="1920" height="1128" alt="Screenshot 2025-07-31 154450" src="https://github.com/user-attachments/assets/4a81ccb3-2786-4abe-bb3a-ab3c17a89e8f" /> 






  

