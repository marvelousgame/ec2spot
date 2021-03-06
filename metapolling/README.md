# metapolling

A quick hack to enable your EC2 spot instances to send SNS notification two minutes in prior to the termination due to price too low and deregister itself from registered ELBs immediately.

Basically this will execute a shell script in the backgeound on instance launch, polling the EC2 metadata every 5 seconds, and publish SNS message to specified topic arn when receiving the termination notify signal.


Read the [official blog](https://aws.amazon.com/tw/blogs/aws/new-ec2-spot-instance-termination-notices/) about how it works. 



## HOWTO

Add the following lines in the `UserData` of your EC2 or launch configuration.

```
# polling the spot metadata URL 
curl -s https://metapolling.ec2spot.com -o /tmp/spot_terminating_notify.sh

# polling metadata in the background
ARN=<YOUR_SNS_TOPIC_ARN> bash /tmp/spot_terminating_notify.sh  &
```

please replace the **ARN** variable above with your real ARN (e.g. `arn:aws:sns:ap-northeast-2:123456789012:MY_SNS_TOPIC` )



make sure your EC2 is running with the role of proper privileges:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1471246554000",
            "Effect": "Allow",
            "Action": [
                "sns:Publish"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "Stmt1471442030000",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:DescribeLoadBalancers"
            ],
            "Resource": [
                "*"
            ]
        }        
    ]
}
```

*restrict the Resource field to the SNS topic arn if you like.*



## SECURITY

If you have security concern, you can download the shell script and save in your EC2 instance locally, rebuilding new AMI and have the **UserData** just execute it locally without using cURL on instance launch.

