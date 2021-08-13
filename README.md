# Overview #
The Fortinet Fortiweb WAF For AWS solution provides protection against requests that attempt to make your web application server. The following diagram represents the architecture that you can build using the solution implementation guide and the accompanying AWS CloudFormation template. Unlike deploying WAF in a traditional environment, deploying WAF on the cloud is recommended to separate public and private subnets. This solution accepts user traffic through ALB or NLB and distributes it to FortiWeb servers located in different availability zones. Fortiweb checks the request according to the set rules, and then uses its own distribution function to continue to send the processed traffic to the Web server that is located in the private network and needs to be included.

# Protection scene #
Web applications are vulnerable to various attacks. These attacks include specially crafted requests aimed at exploiting vulnerabilities or controlling servers. Large-scale attacks designed to destroy websites; or bad bots and crawlers designed to crawl and steal web content. The solution utilizes AWS CloudFormation to configure quickly and easily, and helps prevent the following common attacks:

**SQL injection:** The attacker inserts malicious SQL code into the web request to extract data from the database. This solution is designed to block web requests that contain potentially malicious SQL code.

**Cross-site scripting**: Also known as XSS, attackers use vulnerabilities in benign websites to inject malicious client site scripts into legitimate users' Web browsers. This solution aims to check common elements in incoming requests to identify and prevent XSS attacks.

**HTTP flooding**: Web servers and other back-end resources face the risk of distributed denial of service (DDoS) attacks such as HTTP flooding. When the web request from the client exceeds a configurable threshold, this solution will automatically trigger rate-based rules. Alternatively, implement this threshold by processing AWS WAF logs using AWS Lambda functions or Amazon Athena queries.

**Scanner and probe**: Malicious source scans and probes whether there are vulnerabilities in Internet-oriented web applications. They send a series of requests that generate HTTP 4xx error codes, and you can use this history to help identify and block malicious source IP addresses. This solution creates an AWS Lambda function or Amazon Athena query that automatically parses Amazon CloudFront or Application Load Balancer access logs, calculates the number of incorrect requests from unique source IP addresses per minute, and updates AWS WAF to prevent high Further scan error rate of the address of the address-the error rate at which the defined error threshold is reached.

**Known source of attackers (IP reputation list)**: Many organizations maintain reputation lists of IP addresses operated by known attackers (for example, spammers, malware distributors, and botnets). The solution uses the information in these reputation lists to help you block requests from malicious IP addresses.

**Zombies and crawlers**: Operators of publicly accessible web applications must trust that clients accessing their content can accurately identify themselves and can use the service as expected. However, some automated clients (such as content crawlers or bad bots) distort themselves to bypass restrictions. This solution can help you identify and stop bad crawlers.

# Features #
## Standard safety capability ##
**Protection against OWASP TOP10 threats**: 
built-in multiple protection strategies, you can choose to protect against SQL injection, XSS cross-site, Web Shell, backdoor, command injection, illegal HTTP protocol requests, common web server vulnerability attacks, unauthorized access to core files, path circulation, and scanning Protection, etc.

**Threat Intelligence**: Massive malicious IP blacklist, including botnets, anonymous agents, phishing websites, spam and other malicious IP blocking capabilities
Protection against basic malicious crawlers: block malicious access constructed by libcurl, Python scripts, etc.

**HTTP/HTTPS access control**: IP access control, URL access control, target system management background protection

## Advanced security capabilities ##
**Prevent malicious CC attacks**: effectively intercept CC attacks based on comprehensive intelligent analysis such as HTTP access request frequency/TCP connection frequency/human-machine identification
zero day 

**high-precision security defense**: Based on a two-layer machine learning engine to build an abnormal threat detection model, it has a high accuracy in identifying web attacks, and can automatically detect unknown attacks such as 0day, effectively reducing attack false alarms.
## Business security capabilities ##
**Advanced anti-reptile:** Based on user-agent, IP, client event and AI-based machine recognition technology to accurately identify reptiles.

**Website anti-hotlinking**: prevent website resources from being maliciously linked and used by other websites

**Anti-vulnerability scanning:** to detect attacks, use tools to scan the website for vulnerabilities, and accurately confirm through human-computer interaction, and finally accurately intercept the attacker
system structure


**Public private network and private subnet:** This scheme creates two different subnets, public subnet and private subnet. It is recommended that the Web server protected by Fortiweb be deployed in a private subnet.

**ALB and NLB:** The front-end load balance of the Fortiweb server in this solution can be AWS Application Loadblance or Network Load Balance.

**Automated deployment:** This solution provides Clouformation templates. Users can use the ability of infrastructure as code to automate the deployment of the solution. They can choose to create single-node or multi-node clusters to form basic protection capabilities for Web services.

**High-availability architecture:** In order to meet the needs of users for web security, high availability and redundancy, the solution provides an Active-Active-High volume deployment mode to create Fortiweb Master nodes and Fortiweb Slave nodes in different availability zones. If the Master node fails, the slave node will automatically be upgraded to the Master node to ensure high availability of system functions.

![Archtecture](assets/Architect_diagram.png)
**Data flow:** If Internet users want to access a Web server located in a private subnet

In the first step, the traffic will pass through the Internet Gateway (IGW).
In the second step, it will be forwarded through load balancing (can be ALB type or NLB type)
In the third step, the request will be forwarded to the FortieWeb server, and the request will be detected according to the configured WAF rules. If you find that the request is malicious, you can choose to log or block the request.
In the fourth step, normal requests will be sent to the protected Web server.
A protected Web Server located on a private network needs to access the Internet, such as updating software. This kind of request will be forwarded through the NAT Gateway.

How to build

    cd deployment
    chmod +x ./build-s3-dist.sh \n
    ./build-s3-dist.sh TEMPLATE_BUCKET_NAME $DIST_OUTPUT_BUCKET $SOLUTION_NAME $VERSION \n

For example: If you need to be in the S3 bucket (s3bucketname), you need to run

    ./build-s3-dist.sh s3bucketname s3bucketname fortinet-fortiweb-waf-for-aws v1.0.0 

After running, two directories of /global-s3-assets and /region-s3-assets will be generated in the directory of /deployment

    │  ├── global-s3-assets
    │  │  ├── fwb-ha-vpc-alb-main.template
    │  │  ├── ha-instance-create-alb.template
    │  │  └── ha-vpc-create-alb.template
    │  │  ├── fwb-ha-vpc-nlb-main.template
    │  │  ├── ha-instance-create-nlb.template
    │  │  └── ha-vpc-create-nlb.template
    │  ├── regional-s3-assets
    │  │  ├── lambda
            └── lambda-alb.zip
            └── lambda-nlb.zip

**File structure**

    ├── deployment
    │   ├── build-s3-dist.sh[Shell -  ]
    │   ├── fwb-ha-vpc-alb-main.template[Cloudformation - ALB]
    │   ├── ha-instance-create-alb.template [Cloudformation -  Fortiweb- ALB]
    │   ├── ha-vpc-create-alb.template  [Cloudformation -  VPC- ALB]
    │   ├── fwb-ha-vpc-nlb-main.template [Cloudformation -  NLB]
    │   ├── ha-instance-create-nlb.template [Cloudformation - EC2  Fortiweb NLB]
    │   ├── ha-vpc-create-nlb.template  [Cloudformation -  VPC NLB]
    │
    ├── source
    │   └── lambda
    │   ├── cfnresponse.py  [Python - Lambda ]
    │   └── index.py[Python - Lambda  EC2, Userdata Fortiweb active-active ]
    │   └── policy                                          [FortiWeb ]
    │       ├── AntiCrawler                             []
    │       │   ├── AntiCrawler.txt                     []
    │       │   └── ReadMe.txt                          []
    │       ├── GeneralWebSiteProtect                   []
    │       │   ├── GeneralWebSiteProtect.txt           []
    │       │   └── ReadMe.txt                          []
    │       ├── ProtectDDoS                             [DDoS]
    │       │   ├── ProtectDDoS.txt                     [DDoS]
    │       │   └── ReadMe.txt                          [DDoS]
    │       ├── ProtectDatabaseBruteForce               []
    │       │   ├── ProtectDatabaseBruteForce.txt       []
    │       │   └── ReadMe.txt                          []
    │       ├── ProtectECommerce                        []
    │       │   ├── ProtectECommerce.txt                []
    │       │   └── Readme.txt                          []
    │       └── ProtectHotlinking                       []
    │           ├── ProtectHotlinking.txt               []
    │           └── ReadMe.txt                          []

**How to import scene rules**

To import scenario rules, you need to use SSH to remotely log in to the FortiWeb server and import them. Please download the files in the /source/policy/ directory, and select scenarios according to your needs. 

For example, to select the anti-crawler scenario and enter the /source/policy/AntiCrawler directory, the IP address of the FortiWeb server is 178.xx, and the key pair of the EC2 is johnw.pem, please execute the following command.

    ssh -i ~/johnw.pem admin@178.x.x.x > /dev/null 2>&1 < AntiCrawler.txt


Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge , publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR BE COPYRIGHT FOR HOLDERS CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
