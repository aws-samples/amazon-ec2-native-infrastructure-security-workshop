## Extra Credit Section
[Log all commands](#logging-all-commands)

## Logging all commands
There is a way to log all commands sent to the instance as well. First, you have to create S3 buckets and CloudWatch Logs.

1.    Go to **S3**.
2.    **Create a Bucket** called **cloudsecurity-ssmlogs-bucket-{myname}**. Before you move on, turn on **Default Encryption** using **AWS-KMS** and the **aws/s3**
3.    Then **Grant Amazon S3 Log Delivery group write access to the bucket**.
4.    Go to **CloudWatch**.
5.    In **Logs** you must **Create log group**, called **cloudsecurity-ssmlogs-logs**.
6.    Go to **IAM**.
7.    We will modify the **Role** called SharedServerConnectivityRole
8.    Expand the Inline Policy and click **Edit Policy**
9.    **Add additional permissions** including

[//]: # (Comment: To make linebreaks work here, you have to have two invisible spaces after each line.)
  	 **S3**
	 > **Write**   
	 > **Bucket**: **Any**   
	 > **Object**: **Any**

  	 **CloudWatch Logs**
	 > **Write**   
	 > **Log Group**   
	 > **Log Stream**   
	 > **logs:DescribeLogGroups**   
	 > **logs:DescribeLogStreams**   

  	 **EC2 Messages**
	 > **GetMessages**   

	 **Systems Manager**
	 > **ListInstanceAssociations**   
	 > **UpdateInstanceAssociationsStatus**   

10.    **Review the policy** and **Save Changes**
11.    If there are any errors, go to **Previous** and keep adding **Any** to the resources the policy requires defined.  
_This can be more restrictive in a production environment._
12.    Now go back to **Systems Manager**, **Session Manager**.
13.    Let’s use **Preferences** to set up logging.
14.    **Edit** the settings to **Write session output** and choose the bucket called “**cloudsecurity-ssmlogs-bucket-{myname}**”. 
15.    Let’s also send the output to **Cloudwatch logs**, we can deselect **Encrypt Log Data**, and create a log group name “**cloudsecurity-demo-bucket-{myname}**”.
         * In production I would not recommend storing unencrypted logs.
16.    **Save** that configuration.
17.    Now back at **Sessions**, you can **Start a session** with any server with the SSM agent and access to the SSM Service.

         TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
         curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/instance-id
         curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/security-groups
         curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/iam/security-credentials/SharedServerConnectivityRole

18.    **Terminate** the connection.
19.    Checking **Session History** you will see the **Output Location** of your log.
20.    Look at the **CloudWatch Logs** of your session and see what commands you typed.

Now that you've completed the extra credit, you should continue to [continue with cleanup](./cleanup.md).
