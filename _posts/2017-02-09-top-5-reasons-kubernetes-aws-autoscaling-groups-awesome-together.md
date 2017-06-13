---
ID: 421
post_title: >
  Top 5 reasons Kubernetes and AWS
  Autoscaling groups are awesome together.
author: Shrinand Javadekar
post_date: 2017-02-09 14:02:26
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/top-5-reasons-kubernetes-aws-autoscaling-groups-awesome-together/
published: true
---
<p>Helen Keller once said, “Alone we can do so little, together we can do so much!”</p>
<p>I often think of this quote when working with Kubernetes and AWS together :-).</p>
<p>AWS is feature rich, reliable and battle hardened. Kubernetes provides excellent features for running micro-services based applications.</p>
<p>Together, Kubernetes and AWS Auto Scaling Groups (ASGs) can create magic in scalability, high availability, performance, and ease of deployment!  </p>
<p>Here are 5 reasons why…</p>
<p>&nbsp;</p>
<p><strong>1. Kubernetes minions and master can run in their own ASG.</strong>  </p>
<p>When these are on-demand instances, this virtually guarantees that the desired # of instances will always be running.</p>
<p>Even if some instance goes down, AWS will create a new one.   Kubernetes minions can be configured such that they automatically join a cluster on boot-up.</p>
<p>This guarantees that the cluster will keep having resources it needs to continue to run workloads.</p>
<p>In the case of the Kubernetes master running in an ASG, the master node can restart even if it goes down making the cluster more available.  Or, the master can be scaled up when the load on the cluster is high.</p>
<p>&nbsp;</p>
<p><strong>2. AWS autoscaling groups can span across availability zones (AZs).</strong></p>
<p>This makes minions run across multiple fault domains providing better availability. So even if one AWS availability zone has problems, minions will spin up in other AZs. Once again, the cluster keeps running.</p>
<p>&nbsp;</p>
<p><strong>3. Multiple ASGs can be used if needed for separating the “resource domains”</strong></p>
<p>Separating pods from “User 1” and “User 2” could be done creating different ASGs for each of them and using Kubernetes node labels.</p>
<p>This also allows enforcing some sort of priority between these ASGs, although the logic for this will have to be implemented.</p>
<p>&nbsp;</p>
<p><strong>4. Features like “<a href="https://github.com/kubernetes/contrib/tree/master/cluster-autoscaler">cluster-autoscaler</a>” can make the Kubernetes cluster scale-up or scale-down depending on load.</strong></p>
<p>This is one of the best features of Kubernetes and AWS working together. It is possible to query the Kubernetes master and get the current load (requested amount of CPU and memory). It is also possible to query an AWS autoscaling group and compute whether the required amount of CPU and memory is present in that ASG.</p>
<ul>
	<li>If it is less, simply updating the “Desired” number of instances in the ASG will make AWS create new instances and the Kubernetes cluster can scale.</li>
	<li>If the ASG has far too many resources than needed, the “Desired” number of instances can be reduced and the cluster can be scaled down.</li>
</ul>
<p>&nbsp;</p>
<p><strong>5. Dynamically change launch-configurations for more magic!</strong></p>
<p>ASGs have “launch-configurations” which tell AWS the type of instances to spin-up, the bid price in case these are spot instances, the AZ in which to bring up the instances, and more.</p>
<p>These “launch-configurations” can be updated at run-time. This allows switching between on-demand and spot-instances.</p>
<p>Or, change the instance type (m3.large to m3.xlarge or r3.large, etc..). This makes the Kubernetes cluster very malleable. It can scale-up or scale-out.  This can help improve performance and also provide better SLAs.</p>
<p>AWS Autoscaling groups and Kubernetes rock when used together to benefit a wide variety of workloads.  Have fun!</p>
<p>&nbsp;</p>
<p><em>Shrinand Javadekar is a “cloud junkie” at Applatix where his primary focus is running Kubernetes on AWS. Prior to that, he worked on a wide variety of projects at VMware and Dell/EMC which included cloud storage systems, distributed data caching, virtual machine fault tolerance, etc. Find him on LinkedIn  or <a href="https://twitter.com/shrinandj">@shrinandj</a> </em></p>