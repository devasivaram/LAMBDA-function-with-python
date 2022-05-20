# lambda-function-with-python

In this article am creating a LAMBDA function stop a development instance. Generally the devlepers after their work, they will set the development server as ON, which needs to be minimised, so am writing this article to stop an development servers using LAMBDA function with help of python after working hours.

## Pre-Requestes

We will be using 1 main EC2 instance for working our codes and there will be extra 3 servers (1 for Production and other 2 for development). Here am using jupyter workspace for my code check and we will be needing of 2 modules:

  1. import boto3 - for running our codes
  2. import pprint - for printing and fetch instance details

We need to create 1 IAM user with EC2 full access privilege with programmatic access or you can create IAM role with same privilege but use case EC2, we also need to create 1 IAM role with same privilege but with use case of LAMBDA

![image](https://user-images.githubusercontent.com/100773863/169547046-9fbec3d0-506f-40ed-9aa9-89ae42169bda.png)

# Steps followed

## Step 1: Create servers

Create production (1) and development (2) servers for our project. Add tags for development servers as:
  1. Name: webserver-dev
  2. env: dev
  3. project: anyname (I used Zomato)
 
 ![image](https://user-images.githubusercontent.com/100773863/169548569-e6ed4ea4-5a5e-4796-a5d6-2bc6937a771b.png)
 
 ## Step 2: Fetch server details
 
 Once those are created and added, we need to get those 2 development servs ID and public IP using python code:

~~~sh
import boto3
import pprint

ACCESS_KEY = "your access key"
SECRET_KEY = "your secret key"
REGION = "ap-south-1"

ec2 = boto3.client("ec2", 
             aws_access_key_id=ACCESS_KEY,
             aws_secret_access_key=SECRET_KEY,
             region_name=REGION)

instances = ec2.describe_instances(Filters=[ {'Name':'tag:project', 'Values':["zomato"]},
                                             {'Name':'tag:env', 'Values':["dev"]},
                                             {'Name':'instance-state-name', 'Values':['running']}
                                           ])
                                            
for item in instances['Reservations']:
                                            
  print(item['Instances'][0]['InstanceId'] ,item['Instances'][0]['PublicIpAddress'] )

  print()
~~~

Output:
![image](https://user-images.githubusercontent.com/100773863/169561867-f16a64fd-22c7-4aeb-a201-0906cf374b94.png)

## Step 3: Code for stopping instance

We need to write a code for stopping the developer instance:

~~~sh
import boto3

ACCESS_KEY = "your access key"
SECRET_KEY = "your secret key"
REGION = "ap-south-1"


ec2 = boto3.client('ec2',
             aws_access_key_id=ACCESS_KEY,
             aws_secret_access_key=SECRET_KEY,
             region_name=REGION)

instances = ec2.describe_instances(Filters=[ {'Name':'tag:project', 'Values':["zomato"]},
                                             {'Name':'tag:env', 'Values':["dev"]},
                                             {'Name':'instance-state-name', 'Values':['running']}
                                           ])

                                            
for instance in instances['Reservations']:
                                            
  instance_id = instance['Instances'][0]['InstanceId']
  
  print("Stopping Instance : {}".format(instance_id))

  ec2.stop_instances(InstanceIds=[ instance_id ])
~~~

Output:
![image](https://user-images.githubusercontent.com/100773863/169562077-bd9016bb-f0d6-4e69-bbd5-b1a0d3c65aca.png)


## Step 4: Create LAMBDA function

We use code and platform(python)for starting and stopping servers, now we need to trigger the code for stoping and starting instances automatically/at a specific time. 
This coding is called LAMBDA function. To trigger the LAMBDA function we can use cloudwatch service ->-> events section. We can use cron expression for setting time specific time to trigger.

> Goto LAMBDA service in AWS >> Create function >> fill the following and create. Now we get a interface where we can add code and deploy.

Code:
~~~sh
import boto3

def lambda_handler(event, context):
   
  ec2 = boto3.client('ec2',region_name="ap-south-1")
  
  instances = ec2.describe_instances(Filters=[  
                                              {'Name':'tag:project', 'Values':["zomato"]},
                                              {'Name':'tag:env', 'Values':["dev"]},
                                              {'Name':'instance-state-name', 'Values':['running']}
                                             ])

  for item in instances['Reservations']:
    
    instance = item['Instances'][0]
    instance_id = instance['InstanceId']
    print("Stopping Instance : {}".format(instance_id))
    ec2.stop_instances(InstanceIds=[ instance_id ])
~~~

***Here there are 2 parameters in lambda_handler fuction: Events and Context:***

> event is to know what is trigger is running in lambda, like wat event caused the trigger
> context is to know the memory utilisation of trigger/function


## Step 5: Create trigger

Now we need to set the trigger for our LAMBDA function to work, So we use Cloudwatch for setting trigger.

> Goto cloudwatch service in AWS >> events >> rules >> cloudwatch events >> create rule >> schedule >> select cron expression >> add target >> function name >> 
config details >> add name >> create rule.

![image](https://user-images.githubusercontent.com/100773863/169553243-a7c446b2-4264-44fa-8f9a-ffc2ada0f046.png)

You can set any cron as per your need. Once this is added we can see that our development servers has been stopped automatically.

![image](https://user-images.githubusercontent.com/100773863/169553682-b3ff0dfc-71b4-4439-8583-2c7616defc70.png)


## Conclusion

In this article, we created a LAMBDA function for stopping development instances automatically using cloudwatch trigger so that servers will remain OFF once the devlopers job is finished. Please contact me if you have any questions in this section. Thank you!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
