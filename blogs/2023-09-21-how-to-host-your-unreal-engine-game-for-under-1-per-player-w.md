---
title: "How to host your Unreal Engine game for under $1 per player with Amazon GameLift"
url: "https://aws.amazon.com/blogs/gametech/how-to-host-your-unreal-engine-game-for-under-1-per-player-with-amazon-gamelift/"
date: "Thu, 21 Sep 2023 16:49:09 +0000"
author: "Ryan Greene"
feed_url: "https://aws.amazon.com/blogs/gametech/tag/amazon-gamelift/feed/"
---
<h2>Introduction</h2> 
<p>Game developers constantly seek innovative ways to create more interactive multiplayer experiences.&nbsp;However, with increasing player expectations, rising game production costs, and growing competition, studios face the challenge of balancing game performance and profitability. Game developers from studios of various sizes tell us that reducing operational costs without compromising player experiences is a top priority for 2023 and beyond.</p> 
<p>A good place to start is by evaluating your game tech operation expenditures. <a href="https://pages.awscloud.com/GLOBAL-brand-awareness-content-download-omdia-ardm-cloud-platforms-for-games-report-learn.html?trk=4ae48b23-d281-4470-a85b-5aef2e66d445&amp;sc_channel=el">According to Omdia</a>, an independent game tech analyst firm, operations will account for approximately 19% of a studio’s total tech expenditures in 2023 and rise to nearly 25% by 2027.&nbsp;Game servers, along with compute and data storage, make up the vast majority, nearly 60%, of these operational expenditures.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/09/19/figure-003.jpg"><img alt="Figure 3: Operations as a share of games tech spending and total games market revenue." class="aligncenter wp-image-5197" height="425" src="https://d2908q01vomqb2.cloudfront.net/91032ad7bbcb6cf72875e8e8207dcfba80173f7c/2023/09/19/figure-003.jpg" width="673" /></a></p> 
<p><a href="https://aws.amazon.com/gamelift/">Amazon GameLift</a>&nbsp;from Amazon Web Service (AWS) is a fully managed service that helps developers manage and scale dedicated game servers for multiplayer games while <a href="https://aws.amazon.com/gamelift/faq/#costsavings">decreasing server compute costs by up to 70%</a> compared to existing on-premises deployments. However, customers can find even more savings by leveraging best-practice configuration strategies when setting up their GameLift servers instances. In this blog post, we share three cost-saving techniques we recommend to our customers and explain how these can result in an estimated $1 per-player per-month hosting cost, without compromising game performance or player experience.</p> 
<h2>Pricing overview</h2> 
<p>First, let’s define what type of game we use as our example and what inputs make up the per-player cost calculation. For this blog post, we use a multiplayer game built on <a href="https://www.unrealengine.com/en-US">Unreal Engine</a> as our example. Multiplayer games built on Unreal are an accurate representation of the types of games hosted on Amazon GameLift and provide historical data to reference for our cost estimates.</p> 
<p>Next, let’s cover how we estimate Amazon GameLift hosting costs. We leverage the <a href="https://calculator.aws/#/addService/GameLift">AWS Pricing Calculator for Amazon GameLift</a> to dynamically calculate estimated monthly hosting costs based on game-specific configurations. For this exercise, we estimate GameLift hosting costs in the calculator, which includes the fully managed server hosting with player matchmaking.</p> 
<p>Finally, it’s worth noting that the GameLift pricing calculator, and the price per-player estimates used in this blog, do not take into account any additional cost savings that may be available to customers through AWS discounts or credits.</p> 
<p>Now, on to optimizing our instance costs.</p> 
<h2>GameLift instance costs</h2> 
<p>GameLift instance costs are the costs associated with the compute resources you need to host your game, specifically the number of concurrent users (CCU) your game servers need to support.&nbsp;Amazon GameLift dynamically scales capacity in response to game server activity. As players arrive and start game sessions, GameLift auto scaling adds more instances; as player demand wanes, auto scaling terminates unneeded instances. Auto scaling and per-second billing allows GameLift customers to focus on building games instead of trying to predict game server capacity needed to support fluctuating player demand.</p> 
<p>There are many decisions to make when configuring your GameLift instances, but three in particular have an outsized influence on hosting costs—operating system choice, use of <a href="https://aws.amazon.com/ec2/spot/">AWS Spot Instances</a>, and selection of instance type.</p> 
<h2>Operating system</h2> 
<p>Amazon GameLift supports game servers that run on Windows Server 2016, Amazon Linux 2, and most recently,&nbsp;<a href="https://aws.amazon.com/about-aws/whats-new/2023/06/amazon-gamelift-support-linux-2023/">Amazon Linux 2023</a>. Windows operating systems include an additional licensing cost, which can increase your instance costs by almost 2x over comparable Linux servers. These cost savings make the Linux server operating system a good choice for most GameLift customers.</p> 
<table align="center" border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none;"> 
 <tbody> 
  <tr> 
   <td style="width: 161.75pt; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">On-Demand Instances</p> </td> 
   <td style="width: 94.5pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">Linux</p> </td> 
   <td style="width: 105.45pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">Windows</p> </td> 
   <td style="width: 105.8pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">Savings</p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.large</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.127</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.237</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">46%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.246</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.466</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">47%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.2xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.484</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.926</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">48%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.4xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.962</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$1.845</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">48%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.8xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$1.916</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$3.716</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">48%</span></p> </td> 
  </tr> 
 </tbody> 
</table> 
<p>&nbsp;</p> 
<p style="text-align: center;">On-Demand instance pricing taken from<a href="https://aws.amazon.com/gamelift/pricing/"> Amazon GameLift Pricing page</a> as of August 31, 2023</p> 
<h2>AWS Spot usage</h2> 
<p>Defining a target percentage of AWS Spot usage across your fleet is a key way to save costs. <a href="https://aws.amazon.com/ec2/spot/">Spot instances</a> offer customers access to spare AWS computing capacity at savings of between 50% to 85% compared to on-demand prices. With Spot Instances, you pay the Spot price that’s in effect for the time period your instances are running. Spot Instance prices are set by <a href="https://aws.amazon.com/ec2/">Amazon EC2</a> and adjust gradually based on long-term trends in supply and demand for Spot Instance capacity. Amazon GameLift uses a proprietary algorithm to place new sessions on game servers based on player latencies, instance prices, and Spot interruption rates to help increase cost savings while maintaining high game server availability. Spot instances are not guaranteed and can be interrupted, but by using game session queues and instance diversification, Amazon GameLift customers, including <a href="https://aws.amazon.com/solutions/case-studies/behaviour-interactive-case-study/">Behaviour Interactive</a>, find Spot instances both cost effective and reliable. This makes targeting 60% usage a reasonably conservative approach to leveraging Spot instances.</p> 
<table align="center" border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none;"> 
 <tbody> 
  <tr> 
   <td style="width: 161.75pt; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal"><strong>Linux Instances</strong></p> </td> 
   <td style="width: 94.5pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Spot</strong></p> </td> 
   <td style="width: 105.45pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>On-Demand</strong></p> </td> 
   <td style="width: 105.8pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Savings</strong></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.large</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.04296</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.127</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">66%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.0882</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.246</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">64%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.2xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.18108</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.484</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">63%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.4xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.3642</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.962</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">62%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 161.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="216"> <p align="center" class="MsoNormal">c4.8xlarge</p> </td> 
   <td style="width: 94.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="126"> <p align="center" class="MsoNormal" style="text-align: center;">$0.7272</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$1.916</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">62%</span></p> </td> 
  </tr> 
 </tbody> 
</table> 
<p style="text-align: center;">Linux Spot and Linux On-Demand instance pricing taken from <a href="https://aws.amazon.com/gamelift/pricing/">Amazon GameLift Pricing page</a> as of August 31, 2023</p> 
<h2>Instance types</h2> 
<p>Amazon GameLift supports 68 instance families across 23 regions. For our hypothetical Unreal game, we’ve been comparing c4 instances, where the “c” stands for compute-optimized instances that are designed for compute-intensive workloads, or for our purposes, dedicated game servers. c4 is the 4th generation of this instance family.</p> 
<p>Amazon GameLift now supports instances powered by <a href="https://aws.amazon.com/about-aws/whats-new/2023/08/amazon-gamelift-instances-aws-graviton-3-processors/">AWS Graviton3 processors</a>, including c7g, where the “g” stands for <a href="https://aws.amazon.com/ec2/graviton/">AWS Graviton</a>, which enables optimal price-performance for compute-intensive gaming workloads. AWS Graviton processors are custom designed by AWS using 64-bit Arm Neoverse cores. The 3rd generation, Graviton3 processors, deliver up to 25 percent higher performance, up to 2x higher floating-point performance, and 50 percent faster memory access based on leading-edge <a href="https://en.wikipedia.org/wiki/DDR5_SDRAM">DDR5 memory</a> technology compared with Graviton2 processors. Further, c7g instances are ideal for all Linux-based workloads written in popular programming languages, such C++, C#, and C, and for Unreal Engine-based game servers.</p> 
<p>Graviton support is one of the top requests from GameLift customers looking to further optimize their game hosting costs. AWS customers already using Graviton-based instances, such as <a href="https://www.wired.com/sponsored/story/changing-the-game/">Epic Games</a>, have seen the benefits firsthand. “As we look to the future and building increasingly immersive and compelling experiences for players, we are excited to use AWS Graviton3-based EC2 c7g instances. Our testing has shown they are suitable for even the most demanding latency sensitive workloads while providing significant price-performance benefits and expanding what is possible within Fortnite and any Unreal Engine created experience,” noted <a href="https://aws.amazon.com/solutions/case-studies/innovators/epic-games/">Mark Imbriaco, Epic Games Senior Director of Engineering</a>.</p> 
<table align="center" border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none;"> 
 <tbody> 
  <tr> 
   <td style="width: 175.25pt; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal"><strong>Linux On-Demand Instances</strong></p> </td> 
   <td style="width: 81.0pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>c7g</strong></p> </td> 
   <td style="width: 105.45pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>c4</strong></p> </td> 
   <td style="width: 105.8pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Savings</strong></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 175.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal">Large</p> </td> 
   <td style="width: 81.0pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;">$0.094</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.127</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">26%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 175.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal">xlarge</p> </td> 
   <td style="width: 81.0pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;">$0.180</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.246</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">27%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 175.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal">2xlarge</p> </td> 
   <td style="width: 81.0pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;">$0.354</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.484</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">27%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 175.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal">4xlarge</p> </td> 
   <td style="width: 81.0pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;">$0.701</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.962</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">27%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 175.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="234"> <p align="center" class="MsoNormal">8xlarge</p> </td> 
   <td style="width: 81.0pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="108"> <p align="center" class="MsoNormal" style="text-align: center;">$1.394</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$1.916</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">27%</span></p> </td> 
  </tr> 
 </tbody> 
</table> 
<p style="text-align: center;">Linux On-Demand instance pricing taken from <a href="https://aws.amazon.com/gamelift/pricing/">Amazon GameLift Pricing page</a> as of August 31, 2023</p> 
<h2>Total instance costs</h2> 
<p>Changes to any of these three GameLift configurations can have a meaningful impact on your game server hosting costs. When taken together, these three recommendations can lead to significant cost savings, up to 80% when compared to the alternative. And perhaps even better news, choosing Linux as your operating system and Graviton as your instance type can actually increase the performance and stability of your game servers.</p> 
<table align="center" border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none;"> 
 <tbody> 
  <tr> 
   <td style="width: 125.75pt; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal"><strong>Instances</strong></p> </td> 
   <td style="width: 130.5pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Linux, Spot, c7g</strong></p> </td> 
   <td style="width: 105.45pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Windows, OD, c4</strong></p> </td> 
   <td style="width: 105.8pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>Total Savings</strong></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 125.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal">Large</p> </td> 
   <td style="width: 130.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;">$0.0462</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.237</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">81%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 125.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal">xlarge</p> </td> 
   <td style="width: 130.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;">$0.0918</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.466</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">80%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 125.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal">2xlarge</p> </td> 
   <td style="width: 130.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;">$0.17784</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.926</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">81%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 125.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal">4xlarge</p> </td> 
   <td style="width: 130.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;">$0.3606</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$1.845</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">80%</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 125.75pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="168"> <p align="center" class="MsoNormal">8xlarge</p> </td> 
   <td style="width: 130.5pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="174"> <p align="center" class="MsoNormal" style="text-align: center;">$0.73464</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$3.716</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span style="color: black;">80%</span></p> </td> 
  </tr> 
 </tbody> 
</table> 
<p style="text-align: center;">All instance pricing taken from <a href="https://aws.amazon.com/gamelift/pricing/">Amazon GameLift Pricing page</a> as of August 31, 2023</p> 
<h2>GameLift bandwidth costs</h2> 
<p>Bandwidth costs are captured under data transfer out (DTO), or the costs associated with transferring data from Amazon GameLift to the internet (<a href="https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/">learn more about AWS DTO charges</a>). DTO charges can be an area of uncertainty for game developers and are often overlooked during configuration.</p> 
<p>Because DTO requirements are unique to each game, we can’t give specific recommendations as to how to configure your game’s bandwidth needs, but we can look at historical usage across GameLift customers using Unreal Engine. For these customers, average DTO usage as a percentage of total GameLift monthly hosting costs varied, but just to be on the safe side, we’ll assume DTO represents 20% of the total GameLift costs for our Unreal game.</p> 
<h2>Bringing it together</h2> 
<p>We can now plug in the prior recommendations as inputs into the <a href="https://calculator.aws/#/addService/GameLift">AWS Pricing Calculator for Amazon GameLift</a> and estimate a total monthly hosting cost, which can then be divided by our expected number of peak monthly players (peak CCU) to get to a monthly per player hosting cost. We assume 100,000 as our peak CCU for our example Unreal game, but any peak CCU you choose will give you a similar per-player estimate.</p> 
<p>Other inputs we used in our estimate are as follows: average CCU equals 50% of peak, 16 game sessions per instance, 8 players per game session, 10% idle buffer, and DTO representing 20% of total costs. This comes to a total monthly Amazon GameLift cost of $81,396.67, or $0.81 per player per month. There are some benefits as your peak CCU rises, as the same prior inputs with 10,000 peak CCU comes to $0.87 per player, or with 1 million peak CCU comes to $0.78. But either way, you are well under the $1 per player threshold.</p> 
<table align="center" border="1" cellpadding="0" cellspacing="0" class="MsoTableGrid" style="border-collapse: collapse; border: none;"> 
 <tbody> 
  <tr> 
   <td style="width: 148.25pt; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal"><strong>Cost Details (Monthly)</strong></p> </td> 
   <td style="width: 1.5in; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>10K CCU</strong></p> </td> 
   <td style="width: 105.45pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>100K CCU</strong></p> </td> 
   <td style="width: 105.8pt; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>1M CCU</strong></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 148.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal">Upfront</p> </td> 
   <td style="width: 1.5in; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;">$0.00</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.00</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$0.00</p> </td> 
  </tr> 
  <tr> 
   <td style="width: 148.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal">Instance (c7g.2xlarge)</p> </td> 
   <td style="width: 1.5in; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;">$6,436.55</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$64,365.67</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$643,654.69</p> </td> 
  </tr> 
  <tr> 
   <td style="width: 148.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal">Bandwidth (DTO)</p> </td> 
   <td style="width: 1.5in; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;">$2,285.00</p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$17,031.20</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$135,291.20</p> </td> 
  </tr> 
  <tr> 
   <td style="width: 148.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal">Total Hosting Costs</p> </td> 
   <td style="width: 1.5in; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;">$<span class="awsuiroot18wu0aaqum93">8,721.55</span></p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;">$81,396.67­­­­­­­</p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span class="awsuiroot18wu0aaqum93">$778,945.89</span></p> </td> 
  </tr> 
  <tr> 
   <td style="width: 148.25pt; border-top: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="198"> <p align="center" class="MsoNormal"><strong>Per Player Hosting Costs </strong></p> </td> 
   <td style="width: 1.5in; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="144"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>$0.87</strong></p> </td> 
   <td style="width: 105.45pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="top" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><strong>$0.81</strong></p> </td> 
   <td style="width: 105.8pt; border-top: none; border-left: none; padding: 0in 5.4pt 0in 5.4pt;" valign="bottom" width="141"> <p align="center" class="MsoNormal" style="text-align: center;"><span class="awsuiroot18wu0aaqum93"><strong>$0.78</strong></span></p> </td> 
  </tr> 
 </tbody> 
</table> 
<p style="text-align: center;">Total and per player hosting costs recently estimated using the <a href="https://calculator.aws/#/addService/GameLift">AWS Pricing Calculator for Amazon GameLift</a></p> 
<p>We dove deep into three of the most significant Amazon GameLift cost optimization recommendations, but there are others worth noting. These include:</p> 
<ul> 
 <li><strong>Diversification</strong>: <a href="https://aws.amazon.com/blogs/compute/best-practices-to-optimize-your-amazon-ec2-spot-instances-usage/">Diversifying across regions and instance types</a> increases your pool of available instances, which increases Spot instance availability and reduces the likelihood of interruptions. More Spot usage equates to more cost savings.</li> 
 <li><strong>Queues:</strong> Leverage <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/queues-best-practices.html">best practices for Amazon GameLift game session queues</a> to fall back to on-demand when Spot is unavailable, which is another way to reduce risk.</li> 
 <li><strong>Right-sizing</strong>: Once your game launches, it’s critical to continuously <a href="https://aws.amazon.com/blogs/gametech/game-server-observability-with-amazon-gamelift-and-amazon-cloudwatch/">observe</a>, <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/monitoring-overview.html">monitor</a>, <a href="https://docs.aws.amazon.com/gamelift/latest/developerguide/fleets-manage-capacity.html">tune</a>, and <a href="https://aws.amazon.com/blogs/gametech/how-to-optimize-gamelift-fleets-using-cloudwatch/">optimize</a> Amazon GameLift configurations to increase efficiency and reduce costs.</li> 
 <li><strong>Server operations</strong>: <a href="https://aws.amazon.com/blogs/gametech/qxr-studios-accelerates-game-testing-and-development-with-amazon-gamelift-anywhere/">Reduce development and iteration costs</a> using GameLift Anywhere, a feature of Amazon GameLift that <a href="https://aws.amazon.com/blogs/aws/introducing-amazon-gamelift-anywhere-run-your-game-servers-on-your-own-infrastructure/">decouples game session management from the underlying compute resources</a>.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>Amazon GameLift gives developers a lot of control over how they customize the service to meet their games’ needs. And as we’ve described, configuration choices can have a big impact on your per-player hosting costs. Leveraging Linux servers, Spot instances, and Graviton3 processors can significantly reduce your GameLift hosting costs right out of the gate. Ultimately,&nbsp;developers make decisions about where to host their games sometimes years before actual launch, which makes flexibility and the ability to optimize costs critically important. There are no guarantees when it comes to launching a game, but following the recommendations provided here is a good place to start if you want to host your next multiplayer hit for under $1 per player.</p>
