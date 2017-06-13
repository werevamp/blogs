---
ID: 441
post_title: >
  Public vs. Private Cloud:  Silos vs.
  DevOps
author: Ed Lee
post_date: 2017-03-07 17:17:15
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/public-vs-private-cloud-silos-vs-devops/
published: true
---
<p>In our last two posts, we discussed how the public cloud differs from the private cloud in ways that will impact your approach to managing infrastructure and implementing DevOps.  </p>
<p>First, we discussed how the public cloud is <a href="https://applatix.staging.wpengine.com/aws-public-cloud-not-equal-private-cloud/">API-driven vs. server driven</a>.  Then we discussed the impact of how you prioritize workloads based on how <a href="https://applatix.staging.wpengine.com/aws-instance-not-vm/">public cloud instances behave differently from on-prem VMs</a>.</p>
<p>This time, we bring up another big difference between the public and private cloud – in the on-prem world you manage infrastructure and organizational structure in silos, but in the public cloud, you drive infrastructure with code.</p>
<p>That is, in the on-prem or private cloud world, there are usually three different teams, each which drive different processes in three different ‘layers.’  Specifically:</p>
<ul>
	<li>Development teams write code and test the apps</li>
	<li>Operational teams deploy and monitor the apps</li>
	<li>Infrastructure teams provision servers, networking, and storage to host the apps </li>
</ul>
<p>Fundamentally, in the on-prem world, you are are dealing with a fixed set of resources, and the teams are coordinating the provisioning and allocation of these fixed resources via trouble tickets.</p>
<p>If you need a resource, you ask the next layer, and they may or may not have the needed sources on hand.  If they don’t have it, they have to reprioritize other projects or procure and provision new resources.</p>
<p>This means that if development needs 100 VMs to do a large-scale test, it could be a month or more before they get what they need.</p>
<p>Now, this structure does provide a lot of value.   For example, this makes it possible to impose certain governance policies very reliably.  Each group becomes an expert in enforcing its own part of the overall policy.</p>
<p>On the downside, however, it does promote bureaucracy and has a big negative effect on agility.</p>
<p><strong>In contrast, you drive the public cloud with code</strong></p>
<p>The public cloud is completely driven by APIs.  Using these, you can quickly scale up to effectively infinite on-demand resources.</p>
<p>So the focus is how can I get my job done as quickly as possible, rather than how do I do get the resources to do my job.</p>
<p>Instead of driving your infrastructure with trouble tickets, you are now driving the public cloud with code.  </p>
<p><img class="alignleft  wp-image-334" src="http://applatix.staging.wpengine.com/wp-content/uploads/2016/10/DevOps-private-cloud-vs.-public-cloud-300x95.png" alt="" width="401" height="127" /></p>
<p><strong>Driving your public cloud with code and software makes you much more agile!</strong></p>
<p>Driving your public cloud with code also gives you much better utilization with capabilities like self-service and auto-scaling.   Throw in the ability to buy the right kind of resources for your job, such as spot instances (80% discount) and that is the icing on the cake!</p>
<p>Whereas in the private cloud you spend a lot of your time managing resource constraints, in the public cloud you are focussed on optimizing time constraints. How do I get this work done as fast as possible, given essentially unlimited computing resources?</p>
<p><strong>Now that you are time-constrained, you ask if the job is worth the time and the cost of running it.</strong></p>
<p>This gets to the core of the change in thinking from private to public cloud. </p>
<p>So much in private cloud is about the allocation of resources, managing queues, who gets access, identifying bottlenecks, etc.</p>
<p>But once you go to the public cloud, you have as many resources as you need on demand, so if you are operating properly, you don’t have to wait for resources. <strong>No more queues!</strong></p>
<p>And with self-service, you don’t have to wait for other people either.</p>
<p>All the thinking becomes focused on how do I ship my product the fastest. </p>
<p>But it's not entirely that simple. If you are going to operate in the public cloud, the key to success is embracing DevOps to enable self-service and the optimization of time constraints.</p>
<p>More on that next time, or see our recorded webinar “AWS is different!  How can containers help?” below or on our <a href="https://www.youtube.com/watch?v=mQjMuzSrtGs">cloud best practices YouTube</a> channel.</p>
<p><iframe src="https://www.youtube.com/embed/mQjMuzSrtGs" width="400" height="250" frameborder="0" allowfullscreen="allowfullscreen"></iframe></p>
<p>&nbsp;</p>
<p>&nbsp;</p>