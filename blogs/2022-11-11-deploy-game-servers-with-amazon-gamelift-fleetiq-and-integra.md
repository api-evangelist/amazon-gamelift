---
title: "Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator"
url: "https://aws.amazon.com/blogs/gametech/deploy-game-servers-with-amazon-gamelift-fleetiq-and-integrate-with-custom-routing-aws-global-accelerator/"
date: "Fri, 11 Nov 2022 15:21:50 +0000"
author: "Tulip Gupta"
feed_url: "https://aws.amazon.com/blogs/gametech/tag/amazon-gamelift/feed/"
---
<p>In a session based game, users from all over the world try to access the session over the internet. In these scenarios, building a game is always challenging.</p> 
<p>For developers who want a managed solution,&nbsp;<a href="https://aws.amazon.com/gamelift/">Amazon GameLift</a> is a dedicated game server hosting solution that deploys, operates, and scales cloud servers for multiplayer games. GameLift hosting allows you to focus on your game, while GameLift manages the deployment and scaling of your game servers, as well as other tasks such as game session placement and matchmaking.</p> 
<p><a href="https://aws.amazon.com/global-accelerator/?blogs-global-accelerator.sort-by=item.additionalFields.createdDate&amp;blogs-global-accelerator.sort-order=desc&amp;aws-global-accelerator-wn.sort-by=item.additionalFields.postDateTime&amp;aws-global-accelerator-wn.sort-order=desc">AWS Global Accelerator</a> is a networking service that improves your internet user performance by up to 60% leveraging the <a href="https://aws.amazon.com/about-aws/global-infrastructure/global_network/">AWS global network</a> infrastructure. Using AWS Global Accelerator enhances your players’ online experience by routing player traffic along the private AWS global network, reducing in-game latency, jitter, and packet loss. Amazon GameLift does not provide direct integration to Global Accelerator currently.</p> 
<p>To be able to use Global Accelerator benefits, GameLift offers GameLift FleetIQ, a solution that provides you with more control and responsibility over the game server fleet. <a href="https://docs.aws.amazon.com/gamelift/latest/fleetiqguide/gsg-intro.html">GameLift FleetIQ</a> optimizes the use of low-cost EC2 Spot Instances for cloud-based game hosting with Amazon EC2. With GameLift FleetIQ, you can work directly with your hosting resources in Amazon EC2 and Auto Scaling while taking advantage of GameLift optimizations to deliver inexpensive, resilient game hosting for your players.&nbsp;GameLift FleetIQ is designed to work together with your your existing game backend, including any player geo-IP routing, matchmaking, or lobby services that you might already have in place.</p> 
<p>By leveraging AWS Global Accelerator <a href="https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-aws-global-accelerator-custom-routing-accelerators/">custom routing accelerators</a> to accelerate the traffic for your FleetIQ game server groups, you can reduce in-game lag and jitter for players connecting to your game servers. Doing this puts game session traffic onto the fast fiber of the Amazon network at a network edge point of presence (PoP).</p> 
<p>In this blog post, we will look at deploying a fleet of game servers on <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html">Elastic Container Service</a>, matching players to game sessions using Amazon GameLift FleetIQ, and custom routing the traffic from Global Accelerator to FleetIQ servers.</p> 
<h2>High Level Architecture Overview</h2> 
<p>The client/server Unity project is a simple “game” where 2 players join the same session and move around with their 3D characters. The server receives input from the clients, simulation is run on the server side and world state is sent to the players. The characters are created and removed as the players join and leave.</p> 
<ol> 
 <li>Player requests an identity from Cognito Identity Pool</li> 
 <li>Once the credentials for the identity are received from Cognito, it signs a request to API Gateway to request a game session</li> 
 <li>API Gateway invokes the Request Game Session Lambda function to request a new game session.</li> 
 <li>The function will check if there is a game session in the SQS waiting for another player. If not, it will claim a game server through the GameLift FleetIQ API and put a message to the queue with session information for the next player. The function returns the game server IP and port to connect to. Note: Elasticache for Redis can also be used in place of SQS to store the game session. SQS was chosen for this design, as it is more cost effective solution than elasticache for a 2 player game.</li> 
 <li>The Request Game Session Lambda function retrieves the Global accelerator IP and port associated with the game server IP and port, by calling Global Accelerator. The Global Accelerator port and IP is then passed to the client.</li> 
 <li>Game Client connects to the Global Accelerator port and IP, and are routed to the game server IP and port by the Global Accelerator.</li> 
 <li>The Game Server Group is scaled based on the availability of game servers using Target Tracking. In this example, the default value is 75% utilized game servers as a target. Once new EC2 instances are launched, they will be populated by game server Tasks by the Scaler Lambda function.</li> 
</ol> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="aligncenter wp-image-4630 size-large" height="557" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image001-2-1024x557.png" width="1024" /></p> 
<h2>Custom Routing Global Accelerator</h2> 
<p>By using a custom routing accelerator in AWS Global Accelerator, you can use application logic to directly map one or more users to a specific destination among many destinations while still gaining the performance benefits of Global Accelerator. A custom routing accelerator maps listener port ranges to EC2 instance destinations in virtual private cloud (VPC) subnets. This allows Global Accelerator to deterministically route traffic to a specific Amazon EC2 private IP address and port destination in your subnet.</p> 
<p>In this example, we have specified a listener port range of 10001–30000 and a destination port range of 49151-49170. There are 2 VPC subnets in us-east-1: subnet-1 and subnet-2 both of which host the game servers.</p> 
<p>Each VPC subnet has a block size of /24 so it can support 251 Amazon EC2 instances. (Five addresses are reserved and unavailable from each subnet, and these addresses are not mapped.) Each game server process running on an EC2 instance listens on one of the 20 ports that was specified for the destination ports in the endpoint group: 49151-49170. This means that there are 5020 ports (20 x 251) associated with each subnet. Each port is associated with a session.</p> 
<p>Because we’ve specified 20 destination ports on each EC2 instance in our subnet, Global Accelerator internally associates them with 20 listener ports that you can use to access EC2 instances. To illustrate this simply, we’ll say that there’s a block of listener ports that starts with the first IP address of the endpoint subnet for the first set of 20, and then moves to the next IP address for the next set of 20 listener ports.</p> 
<p>In the example, the first listener port is 10001. That port is associated with the first subnet IP address, 10.0.0.4, and the first EC2 port, 49151. The next listener port, 10002, is associated with the first subnet IP address, 10.0.0.4, and the second EC2 port, 49152. The following table illustrates how this example mapping continues through the last IP address of the first VPC subnet.</p> 
<p><img alt="Custom Routing Global Accelerator" class="size-full wp-image-4636 aligncenter" height="703" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/Spreadsheet.png" width="686" /></p> 
<h2>Deployment Steps</h2> 
<p>The deployment will be done in 2 steps. Part 1 showing how to host game servers with FleetIQ, and part 2 showing the integration of Global Accelerator with FleetIQ</p> 
<h2>Part 1- Game Server Hosting on Amazon Elastic Container Service with Amazon GameLift FleetIQ</h2> 
<p>This <a href="https://github.com/aws-samples/amazon-gamelift-fleetiq-with-amazon-ecs">github repository</a> contains an example solution on how to scale a fleet of game servers on Elastic Container Service and match players to game sessions using a Serverless backend. Game Sessions are managed by Amazon GameLift FleetIQ. All resources are deployed with Infrastructure as Code using <a href="https://aws.amazon.com/cloudformation/">CloudFormation</a>, <a href="https://aws.amazon.com/serverless/sam/">Serverless Application Model</a>, <a href="https://aws.amazon.com/docker/">Docker</a> and bash scripts.</p> 
<h3>Key Features</h3> 
<ul> 
 <li>GameLift FleetIQ used to provision a Game Server Group. EC2 instances in the group will register to an ECS Cluster as workers</li> 
 <li>Game Server Tasks deployed to the ECS Cluster by a serverless scaler service to always keep all instances fully utilized</li> 
 <li>A Serverless API used for claiming game sessions from GameLift FleetIQ. Deployed with Serverless Application Model</li> 
 <li>CloudFormation used to deploy all infrastructure resources</li> 
 <li>Cognito used for player identities. Requests against the API are signed with Cognito credentials</li> 
 <li>Unity used on both server and client side. Unity server build is deployed as a Linux Docker container</li> 
</ul> 
<h3>Contents</h3> 
<p>The project contains:</p> 
<ul> 
 <li><strong>A Unity Project</strong> that will be used for both Client and Server builds (<code>UnityProject</code>). Server build will be further built into a Docker container image and deployed to Elastic Container Registry as part of the deployment automation</li> 
 <li><strong>A Backend Project</strong> created with Serverless Application Model (SAM) to create a backend for game session requests using API’s, and a continuously running scaler function for creating game server Tasks (<code>BackendServices</code>)</li> 
 <li><strong>Infrastructure deployment automation</strong> leveraging AWS CloudFormation to deploy all infrastructure resources (<code>CloudFormationResources</code>)</li> 
 <li><strong>A build folder for the server build</strong> which includes a Dockerfile for building the docker image (<code>LinuxServerBuild</code>)</li> 
</ul> 
<h3>Implementation Overview</h3> 
<p>The AWS Infrastructure for the solution consists of</p> 
<ul> 
 <li>a <strong>VPC</strong> with public subnets across two Availability Zones to host the ECS Container Instances on top of which we will deploy the ECS game server Tasks (<code>CloudFormationResources/ecs-resources.yaml</code>)</li> 
 <li>an <strong>ECS Cluster</strong> that is used to host the ECS Tasks (<code>CloudFormationResources/ecs-resources.yaml</code>)</li> 
 <li>a <strong>GameLift FleetIQ Game Server Group</strong> that maps to an Auto Scaling Group of instances that register to the ECS Cluster and use the ECS AMI. This Game Server Group will scale based on the percentage of available game servers using Target Tracking. (<code>CloudFormationResources/ecs-resources.yaml</code>)</li> 
 <li>a <strong>Cognito Identity Pool</strong> that is used to store the player identities (<code>CloudFormationResources/cognito.yaml</code>)</li> 
 <li>an <strong>ECS Task Definition</strong> that defines the Task to be run in public subnets of the VPC using the game server image uploaded to ECR. The Tasks use bridge-networking and HostPort 0 to get a dynamic port at launch. The Security Group of the instances allow access to the dynamic port range from the internet. (<code>CloudFormationResources/game-server-task-definition.yaml</code>)</li> 
 <li>A<strong> custom routing Global Accelerator</strong> that maps the Global Accelerator IP and port to the backend game server IP and port</li> 
</ul> 
<h2>Part 2 – Integration of AWS Global Accelerator with Amazon GameLift FleetIQ to Improve In-Game Performance</h2> 
<h3>Creating a custom routing accelerator</h3> 
<p>In order to allow game traffic to be sent from the game client to the accelerator, first you create a custom routing accelerator and add each subnet, that will contain your servers in the fleet, as endpoints to the custom routing accelerator.&nbsp;In this section you can see how the custom routing Global Accelerator can be deployed through AWS Management Console. Global Accelerator custom routing is not supported by CloudFormation at the time this blog post was published.</p> 
<ul> 
 <li>Open the Global Accelerator console at <a href="https://console.aws.amazon.com/globalaccelerator/home">https://console.aws.amazon.com/globalaccelerator/home</a></li> 
 <li>Choose <strong>Create accelerator</strong>. 
  <ul> 
   <li>Provide a name for your accelerator. In the example “testagademo” has been used.</li> 
   <li>For <strong>Accelerator type</strong>, select <strong>Custom routing</strong>.</li> 
   <li>For each static IP address, choose the IP address pool to use.</li> 
  </ul> </li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4631 aligncenter" height="648" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image003-3.jpg" width="823" /></p> 
<ul> 
 <li><strong>Add Listeners</strong> 
  <ul> 
   <li>Enter <code>Ports</code> range as <code>10001-30000</code></li> 
  </ul> </li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4632 aligncenter" height="479" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image005-2.jpg" width="1552" /></p> 
<ul> 
 <li><strong>Add endpoint groups</strong> 
  <ul> 
   <li>Select the region you have deployed your game servers</li> 
   <li>Enter <code>49151</code> as <code>Port range start</code> and <code>49170</code> as <code>Port range end</code>. Choose <code>Protocols</code> as <code>TCP</code>.</li> 
  </ul> </li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4633 aligncenter" height="396" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image007-1.jpg" width="971" /></p> 
<ul> 
 <li><strong>Add VPC subnet endpoints.</strong> 
  <ul> 
   <li>Select Subnet as Endpoint type.</li> 
   <li>Select the subnets created earlier as a part of Part 1 deployment in Endpoint.</li> 
   <li>Select Allow all traffic for Subnet traffic.</li> 
  </ul> </li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4634 aligncenter" height="871" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image009-1.jpg" width="1553" /></p> 
<ul> 
 <li><strong>Create the accelerator.</strong> 
  <ul> 
   <li>The accelerator gets created with two external IP addresses associated with it.</li> 
  </ul> </li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4635 aligncenter" height="301" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image011-1.jpg" width="1553" /></p> 
<p><strong>Registering FleetIQ servers</strong></p> 
<p>Once the custom routing global accelerator has been created, update the&nbsp;Request Game Session lambda function to send back the IP and port of the global accelerator.</p> 
<ul> 
 <li>Select the IAM role associated with the Lambda function and provide permissions to access Global accelerator.</li> 
</ul> 
<p><img alt="Deploy Game Servers with Amazon GameLift FleetIQ and Integrate with Custom Routing AWS Global Accelerator" class="size-full wp-image-4629 aligncenter" height="585" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2022/11/10/image013-1.jpg" width="1629" /></p> 
<ul> 
 <li>Update requestgamesession.py to call the Global Accelerator and get the IP address and port.</li> 
</ul> 
<pre><code class="lang-json"># Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: MIT-0

import json
import os
import logging
import boto3
logger = logging.getLogger()
logger.setLevel(logging.DEBUG)

aga_client = boto3.client('globalaccelerator', region_name='us-west-2')


# Retrieves port and IP from custom routing Global Accelerator for a server IP and port
def list_custom_routing(private_ip_address, port, subnet_id):
    agaport = []
    port1=int(port)

    try:
        response = aga_client.list_custom_routing_port_mappings_by_destination(
            EndpointId = subnet_id,
            DestinationAddress=private_ip_address,
            MaxResults=30
    )
    except Exception as e:
        logger.error("Could not find IP and port in AGA: %s", str(e))
        
    try:
        for prop in response['DestinationPortMappings']:
            print (prop['DestinationSocketAddress'])

            if prop['DestinationSocketAddress']['Port'] == port1:
                agaport = prop['AcceleratorSocketAddresses'][0]['Port']
                agaip = prop['AcceleratorSocketAddresses'][0]['IpAddress']

                break
            else:
                print("no match")
 
          
    except Exception as e:
        logger.error("Could not find port and IP in AGA: %s", str(e))

    return agaip, agaport


# Tries to find an existing or free game session and return the IP and Port to the client
def lambda_handler(event, context):
    sqs_client = boto3.client('sqs')

    # 1. Check SQS Queue if there are sessions available
    # Try to receive message from SQS queue
    try:
        response = sqs_client.receive_message(
            QueueUrl=os.environ['SQS_QUEUE_URL'],
            MaxNumberOfMessages=1,
            VisibilityTimeout=15,
            WaitTimeSeconds=1
        )

        if len(response['Messages']) &gt; 0:
            message = response['Messages'][0]
            print(message)
            print("received from queue")
            receipt_handle = message['ReceiptHandle']
            connection_info = message['Body']
            print(receipt_handle)
            print("got session: " + connection_info)

            connection_splitted = connection_info.split(":")
            agaip = connection_splitted[0]
            agaport = connection_splitted[1]

            print("IP: " + agaip + " PORT: " + agaport)
    
            # Delete received message from queue
            sqs_client.delete_message(
                QueueUrl=os.environ['SQS_QUEUE_URL'],
                ReceiptHandle=receipt_handle
            )

            # Return result to client
            return {
                "statusCode": 200,
                "body": json.dumps({ 'publicIP': agaip, 'port': agaport })
            }
        else:
            print("No session received from the SQS queue, will try claiming a new one")
    except:
        print("Failed getting a session from the SQS queue, will try claiming a new one")

    # 2. If not, try to claim a new session through FleetIQ
    try:
        client = boto3.client('gamelift')
        response = client.claim_game_server(
            GameServerGroupName='ExampleGameServerGroup',
        )
        print("got game server")
        print(response)
        connection_info = response["GameServer"]["ConnectionInfo"]
        instance_id = response["GameServer"]["InstanceId"]

        connection_splitted = connection_info.split(":")
        ip = connection_splitted[0]
        port = connection_splitted[1]

        print("IP: " + ip + " PORT: " + port)
        
        # Retrieve private IP address for the server to call AGA    
        ec2_client = boto3.client('ec2')
        response1 = ec2_client.describe_instances(
            Filters=[
                {
                    'Name': 'instance-id',
                    'Values': [instance_id]
                }
            ]
        )
            
        private_ip_address = response1['Reservations'][0]['Instances'][0]['NetworkInterfaces'][0]['PrivateIpAddress']
        subnet_id = response1['Reservations'][0]['Instances'][0]['NetworkInterfaces'][0]['SubnetId']
        
        # Call AGA to get the AGA port and IP
        agaip, agaport = list_custom_routing(private_ip_address,port,subnet_id)

        # Put a ticket in to the SQS queue for the next player (we match 1-v-1 sessions)
        agaportst=str(agaport)
        concat_connection_info = agaip + ":" + agaportst

        response = sqs_client.send_message(
            QueueUrl=os.environ['SQS_QUEUE_URL'],
            MessageBody=(
                concat_connection_info
            )
        )

        print(response['MessageId'])

        return {
            "statusCode": 200,
              "body": json.dumps({ 'publicIP': agaip, 'port': agaport })
        }
    except:
        print("Failed getting a new session")

    # 3. Failed to find or claim a server
    return {
            "statusCode": 500,
            "body": json.dumps({ 'failed': 'couldn\'t find a free server spot'})
    }

</code></pre> 
<p>Now the client will only see the IP and port of the Global Accelerator . When it connects to that IP and port of the Global Accelerator, the traffic will be routed to the game server in your fleet.&nbsp;You can also check for errors in CloudWatch logs for the Lambda function Request Game Session.</p> 
<p>To verify that you’ve set things up correctly, you can build the example game client and then run two game clients.</p> 
<p><strong>Build and run two clients</strong></p> 
<ul> 
 <li>Set the the Scripting Define Symbol CLIENT in the Player Settings in the Unity Project (File -&gt; “Build Settings” -&gt; “Player settings” → “Player” → “Other Settings” → “Scripting Define Symbol” → Replace completely with “CLIENT”)</li> 
 <li>Open the scene “GameWorld” in Scenes/GameWorld</li> 
 <li>Open Build Settings (File -&gt; Build Settings) in Unity and set target platform to Mac OSX (or whatever the platform you are using) and uncheck the box Server Build</li> 
 <li>Build the client to any folder (Click “Build”, select your folder and click “Save”)</li> 
 <li>You can run two clients by running one in the Unity Editor and one with the created build. This way the clients will get different Cognito identities. If you run multiple copies of the build, they will have the same identity. In this solution it doesn’t matter but if your backend starts using the identity for player data, then it might.</li> 
 <li>You should see the clients connected to the same game session and see the movements synchronized between clients.</li> 
</ul> 
<h3>Cleaning Up Resources</h3> 
<p><strong>NOTE</strong>: Recommend deleting Global Accelerator prior to deleting resources in part 1</p> 
<p>To clean up resources in part 1</p> 
<ol> 
 <li>Stop all tasks in the ECS cluster</li> 
 <li>Modify configuration.sh with the correct region</li> 
 <li>Run cleanup_all_resources.sh</li> 
</ol> 
<p>To clean up resources in part 2,</p> 
<ol> 
 <li>Disable the Global Accelerator</li> 
 <li>Delete Global Accelerator</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this blog post, you saw how a fleet of game servers can be deployed using GameLift FleetIQ and an ECS cluster. You also saw how custom routing Global Accelerator can be integrated to the FleetIQ servers to provide&nbsp;TCP or UDP traffic flow from the players to the server (and back, since Global Accelereator accelerates traffic in both directions), allowing them to experience their game with the reduced latency, jitter reduction and packet loss.</p> 
<p>If you have a gaming application that requires a group of users to interact with each other on the same session running on a specific EC2 instance and port, custom routing Global Accelerator can&nbsp;reduce latency and jitter globally for such applications, to enable a better player experience.</p>
