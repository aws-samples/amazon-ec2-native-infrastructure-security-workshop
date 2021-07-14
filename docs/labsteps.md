# Mitigate Risks Using Cloud-Native Infrastructure Security

## Lab Sections

1.    [Enable granular logging](#enable-granular-logging-to-see-everything-in-your-aws-environment)
2.    [Improve granular control of communication](#granular-provable-control-of-communications)
3.    [Improve granular network-based controls](#when-security-includes-explicitly-denying-network-access)
4.    [Evaluate detailed logging capabilities](#logging-actions-in-your-environment-and-making-it-easy-to-see-whats-changed)
5.    [Evaluate network-based protections](#logging-and-monitoring-of-the-network-for-bad-behavior-is-important-too)
6.    [Minimize admin access risk](#reducing-the-risk-of-admin-access-and-administrative-ports)

## Enable granular logging to see everything in your AWS environment
We want to enable detailed, holistic logging and network-based security monitoring.

1.    Go to the **CloudTrail** service in the console.  
2.    If it appears, click on Getting Started.
3.    We want to **Create trail**. 
4.    Let’s set a **Trail Name** of “**All-API-Commands-across-all-Regions**”. 
6.    We should save these for further evaluation, so you would want to **Create a new S3 bucket** and call it “**cloudsecurity-demo-bucket-{myname}**”. (Don’t forget, S3 buckets must have unique names, so make sure to add your name at the end. They can also only be lower case letters, numbers, “-“, and “.”)
1.    Log file encryption will be enabled by default. Give it the name of a new KMS key to use for the encryption by specifiying "**cloudsecurity-demo-kms-key**".
7.    After we **Create** that trail, we'll get the option to pick what **Events** we want to record.
1.    Make sure to select **Management Events**, with **Read** / **Write** activity, at minimum.  These options will show us every API call made to our AWS environment moving forward.
     * What are your other options? When would you want to enable those? 
1. Click *Create* to finalize the trail.  We can come back to look at the results later.
     * Monitoring what API calls are made is great, but it’s difficult to convert that into something like Change Management for all infrastructure in the cloud. Is there a service to help there?
8.    Let us also look into one of the **Services** called **Config**.
9.    After **Getting Started** we can start tracking All resources, Including Global Resources
11.    AWS does a good job of clearly defining roles, so let’s allow AWS to **Create AWS Config service-linked role**
10.    We would want to store this data in a central bucket as well. Let’s **Create a new bucket** and use the default to ensure it's unique. 
    * Note: We could use the bucket we just created, but you would have to add permissions, which would take more time.
12.    **Next**, we can choose rules we want to test against, but we can do that later too if we skip it for now.
13.    **Confirm** these choices to enable Config to monitor all changes to our environment. And we’ll see that in a bit.  
Now that we’ve got good logging of the Control Plane (API commands and Changes to the environment), let’s turn on logging of the Data Plane.
14.    Then, let us use a **Service** like **GuardDuty** to monitor logs in near real time for security anomalies.
15.    After **Getting Started** we can quickly enable this service with just one click. It’s that easy. We'll check out what GuardDuty monitoring is later.

When we looked at our on-premises environment we identified that disjointed security tooling, lack of insight into what’s going on in the environment, and difficulty managing change control and permissions in the environment all led to risks becoming problems pretty fast. With the services we just enabled, we’ll see how we now have complete insight into who’s doing what in the environment, what changes are being made, and if and when problems start to arise.

## Granular, Provable Control of Communications
Let's review and improve upon granular control of communication between workloads in the cloud.

1. Looking at the granular control of system-to-system communication used to be difficult. Now, looking at your **EC2** Service **Security Groups** allows you to quickly see who can talk to whom.
2.    Picking a Security Group like the **Services Server Security Group** we can see the more traditional way of doing things.
3.    Checking the **Outbound** rules, we see the servers can talk to a range of IP’s, 65,536 to be precise. But there are only maybe 6-8 servers that they actually need to talk to.
4.    Well, if we copy the **Security Group ID** of the **PoC Web Server Security Group** we can start to reduce that number
5.    **Edit** the **Outbound** set of rules for the **Services Server Security Group**. Delete the 10.0.0.0/16 rule and make a new rule with the Security Group name you copied.
6.    **Save** this and you’ll see the **Destination** of the rule now shows the Security Group you listed.
7.    You can **repeat** this by **Edit**ing and **Adding Rule**s for each security group you want to allow access to.

In doing this, you’ve reduced the scope of internal traffic communication from 65,636 host down to 8\. Additionally, if you ever need to stand up more servers in these groups, they would be automatically accessible without intervention, as long as you put them in the same Security Group. On premise, you would either need to have Firewalls between all internal VLAN’s, Routers, and sites or complex Network ACL’s on every switch in your environment. This reduces the risk of threats, the risk of misconfiguration, and the operational burden all at once.

## When Security includes explicitly denying network access
Let's improve on our network-based controls by using Network ACLs to prevent side-to-side movement in a granular way.

1.    Security Groups are awesome at allowing access, but in **VPC** Services, **Network ACLs** are great at explicitly blocking them.
2.    For instance, if you wanted to make sure you explicitly blocked the Load Balancer in my WebApp from talking to my Database servers, you could **Create a network ACL**.
3.    I would name it “**LoadBalancerIsolation**” and put it in the **Web Application VPC**.
4.    After selecting the new network ACL, I would add an **Outbound Rule** by **Editing Outbound Rules**
5.    **Adding Rules** like these would block whatever Subnet you apply this to from talking to the Database Subnets but still allow access to the rest of the network, including the Web and Services VPC. Note that NACLs are evaluated in the specific order of their Rule #. 
      * Rule #: **50**  
      Of type **All Traffic**  
      To the Destination **10.0.2.0/24**  
      And a **Deny** Behavior  
           and
      * Rule #: **60**  
      Of type **All Traffic**  
      To the Destination **10.0.130.0/24**  
      And a **Deny** Behavior  
           and
      * Rule #: **100**  
      Of type **All Traffic**  
      To the Destination **0.0.0.0/0**  
      And an **Allow** Behavior  
6.    After **Saving** you need to allow access to that subnet from the internet, so recreating the **All Traffic Allow** rule for **Inbound Rules** is necessary. **Add a Rule**
      * Rule #: **100**  
      Of type **All Traffic**  
      To the Destination **0.0.0.0/0**  
      And a **Allow** Behavior
2.    After **Saving** you would then use **Subnet Associations** to **Edit Subnet Associations**.
3.    Here you would Associate with the **Web App Public Subnet in AZ1** and **Web App Public Subnet in AZ2** subnets by selecting the subnets and clicking **Save changes**.

      Now, you’ve effectively ensured that if the Load Balancers in your environment misbehave, they can’t communicate with or compromise the Database servers directly. But there was no additional hardware, firewall, or complex routing required to make this simple change in the simple network topology.

     But let’s say you want to go further. The Proof of Concept servers you built aren’t exchanging state so they don’t need to communicate to each other. This way, if one gets compromised it can’t hurt the other. Let’s build that protection.

1.    You would **Create a network ACL**.
2.    Name it “**PoCProtectionAZ1**” and put it in the **Proof of Concept VPC**.
3.    Select the new ACL, and add an **Outbound Rule** by **Editing Outbound Rules**
4.    **Add Rules**
      * Rule #: **50**  
      Of type **All Traffic**  
      To the Destination **10.250.128.0/24**  
      And a **Deny** Behavior  
        and
      * Rule #: **100**  
      Of type **All Traffic**  
      To the Destination **0.0.0.0/0**  
      And an **Allow** Behavior  

1.    **Save** the rules and allow access to that subnet from the internet. **Edit inbound Rules** and **Add Rule**
      * Rule #: **100**  
      Of type **All Traffic**  
      To the Destination **0.0.0.0/0**  
      And an **Allow** Behavior  
1.    After **Saving** you would then use **Subnet Associations** to **Edit Subnet Associations**.
2.    Here you would Associate with the **Proof of Concept Public Subnet in AZ1** subnets by selecting it and clicking **Save Changes**.
3.    You can choose to duplicate those steps to block the other direction by setting
      * Name to “**PoCProtectionAZ2**”
      * Blocking traffic to the Destination **10.250.0.0/24**
      * And Associating with **Proof of Concept Public Subnet in AZ2**

Now, even within the same workload or application you are protecting servers from each other where putting Firewalls in the past would have been impossible.

## Logging actions in your environment and making it easy to see what’s changed
1.    We just made a bunch of changes, and on-premises it may be difficult to track them. Let’s check the **CloudTrail** in **Services** to see what it noticed.
2.    The Dashboard shows us some recent events, but we want to see the complete **Event History**.
3.    Here we can see your **User Name** performing actions tracked by **Event name** against different **Resource Names**. API commands not related to Data (because we chose that earlier) are being captured by CloudTrail.
4.    You can remove the system calls by using a **Filter** on **User Name** and putting your **User Name** in the text box and hitting enter. Now **Scroll down**. Do you see all the ACL’s you changed?
        But again, seeing these API calls doesn’t give you a good visual of the changes occurring.
5.    Let's go to **Config**.
6.    I can see all of my resources, including some called **EC2 NetworkAcl**.
3.    Clicking there gives you a list of ACL’s, and you can **click** on the first one.
4.    Seeing details on that ACL, you can also see a visual **Resource Timeline**
5.    In the configuration timeline, you can see the changes that occurred over the past few minutes.  
      * If you don’t see any changes go back and choose a different ACL
6.    If you expand the **Configuration change** item, you can see exactly what changes you made to the resource, including what you applied it to as a "Relationship Change".

How would you do this on-premises?

## Logging and monitoring of the network for bad behavior is important too
1.    Let’s go back to **GuardDuty** and see what findings we may have.  If this is a new or infrequently used account, you may have no Findings. If you do have Findings and this is not a new account, we can walk through those separately.
2.    Since this is a good design and relatively new, let’s create some demonstration findings in **Settings**
3.    After we **Generate sample findings** we can go back to the **Findings**
4.    You'll see some High Severity, Medium Severity,  and Informational Findings. Let’s investigate the High Severity finding called **[SAMPLE] 
[SAMPLE] Backdoor:EC2/DenialOfService.Dns**
5.    You can see the (fake) instance that caused this Finding, what the instance did wrong, when it occurred, and more information. The “.Dns” at the end means something, do you know what? Does the **Action Type** help?  
      * What are the 3 data sources GuardDuty uses?
6.    **Scrolling down** the Findings list again you see another high severity **[SAMPLE] Backdoor:EC2/DenialOfService.UdpOnTcpPorts**.
7.    Here you see a lot of the same type of information. But why is this **Action Type** different?
      * If we didn’t turn on those logs how did it see the traffic?
8.    **Scrolling down** the Findings list a bit more you find **[SAMPLE]UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration**
9.    This one not only has a different **Action Type** but also starts with **IAMUser** instead of **EC2**. Why does that matter?  
      *  Is this a different data source?

Now you’ve seen that GuardDuty is monitoring logs on your behalf, and without you having to pay for storage, the AI/ML or Threat feeds, and the man hours to do the analysis. This is all happening at Cloud scale too, no longer do you need to have terabytes of logs that are never touched.

## Reducing the risk of Admin access and administrative ports
Finally, let's further reduce administrative risks by reducing access and improving logging. With our current setup, there is still a risk of open administrative ports, right? It's a bigger risk if those ports are open to the internet and a smaller risk if open internally for malware to find. Can we find a way around this requirement?

1.    Let’s go back to **VPC** and **Security Groups**.
2.    Open the **Services Server Security Group** and the **Inbound Rules**.
3.    Now, despite the fact that those are made up IPs, you are going to **Edit Rules** and delete all the rules (Click the x on the right). Then **Save** and **Close**.
4.    But with no access, how can we monitor or log into the box if we need to? Our **Service** called **Systems Manager** can help there.
5.    Systems Manager has a feature called **Session Manager** worth checking out.
6.    At **Sessions**, you can **Start a session** with any server with the SSM agent and access to the SSM Service.
7.    We disabled all access to the **Services Server for AZ1**, yet there it is. Let’s select it and **Start session**.
8.    Is this a console? For the AWS server? Let’s find out.  
       Type: 
  
        >  TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`  
        >  curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/instance-id  
   
       Does that instance ID look familiar?

        >  curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/security-groups

       That looks like the Security Group we modified doesn’t it?

       	>  ping 8.8.8.8

       Should it work?

        >  curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/iam/security-credentials/SharedServerConnectivityRole


       Sure looks like an AWS server.

Congratulations!  You have successfully set up this AWS environment for strong logging with Cloudtrail and Config, granular communication with Security Groups and NACLs, intelligent threat detection with AWS GuardDuty, and configured your machines to have safe administrative access without requiring access from the public internet. 

If you have reached the bottom of this page with time to spare, there is [Extra Credit](./extracredit.md) available.

Finally, make sure to [Clean up](./cleanup.md) your environment to ensure you don't have any continuing charges.


