---
ID: 343
post_title: >
  Why DevOps in the public cloud must be
  purpose-built
author: Saradhi Sreegiriraju
post_date: 2016-11-01 07:47:00
post_excerpt: ""
layout: post
permalink: >
  http://applatix2.wp/design-devops-automation-public-cloud-aws-azure/
published: true
---
<p>Last time we discussed how <a href="http://applatix2.wp/devops-key-success-public-cloud-aws-azure/">DevOps is the key to success in the public cloud,</a> based on differences between the behavior of the public and private cloud. </p>
<p>In short, the public cloud offers tools, services, technologies and consumption models that are simply not available in on-premises infrastructure. For reasons discussed earlier, lifting and shifting of on-premises tools, technologies, and processes to public cloud will not work.  You must operate the public cloud differently and <a href="http://applatix2.wp/devops-key-success-public-cloud-aws-azure/">DevOps is mandatory in services like AWS, Azure, and Google Cloud</a>. </p>
<p>A purpose-built DevOps system for public cloud is also a must. To use an analogy, a purpose built DevOps system for the public cloud is like an autonomous car, a fundamentally different system built from the ground up with intelligent software.  Using your on-premises DevOps system with plug-ins for the public cloud is like adding a bolted-on lane change warning system to your existing car. Let us examine why. </p>
<p><strong>Public cloud instance !=</strong><strong> On-premises VM</strong></p>
<p>One of the fundamental mistakes is to treat an instance in public cloud the same as a VM on on-premises infrastructure. A VM running on on-premises is an elastic slice of a server. That is, any CPU or other resources not used by a VM are available for use by other VMs on that fixed capacity server. In contrast, an instance in the public cloud has dedicated resources. <em>Any resources not used by an instance are wasted. Given that the meter is always running in public cloud, it is tantamount to wasting spend</em><strong>.</strong></p>
<p>The following picture can be helpful in illustrating the differences.</p>
<p><img class="wp-image-345 aligncenter" src="http://applatix2.wp/wp-content/uploads/2016/10/VM-vs.-public-cloud-instance-300x87.png" alt="VM vs. AWS instance in public cloud " width="569" height="165" /></p>
<p>&nbsp;</p>
<p>Without recognizing this, a lift-and-shift of VMs, tools, and applications from on-premises infrastructure to public cloud results in 'sticker shock' and utilization levels that are actually lower.    </p>
<p><strong>How do you address this? Containers!</strong></p>
<p>Containers provide a light-weight mechanism to run multiple workloads on the same instance to achieve higher utilization.  </p>
<p>Most successful large scale providers such as Google and Facebook have been leveraging containers for more than a decade to achieve rapid ship cycles and resource utilization levels near 80%. In addition to enabling multiple, different workloads on an instance, containers enable easy, consistent packaging of applications for easy distribution and deployment. </p>
<p>Of course, using container technology comes with it's own challenges as the technology is relatively early and moving quickly.</p>
<p><strong>The Public cloud APIs offer new capabilities and consumption models</strong></p>
<p>Public cloud services like AWS offer many capabilities that are simply not available on on-premises infrastructure. The following picture can be helpful in illustrating the distinctive features of public cloud (using AWS as an example).</p>
<p><img class=" wp-image-349 aligncenter" src="http://applatix2.wp/wp-content/uploads/2016/10/public-vs.-private-cloud-API-benefits-1-300x236.png" alt="compare AWS to private cloud APIs" width="441" height="347" /></p>
<p>Let us look at few of these distinctive features.</p>
<p><em>Programmability</em>: Public cloud resources – compute, storage, networking, and application services – are all programmable via APIs. Manage code to manage public cloud resources.</p>
<p><em>Instant global scale</em>: Leveraging resources distributed in multiple geographies is as simple as making an API call.</p>
<p><em>Cost efficiencies</em>: Market place based resources (such as spot instances for compute in AWS) provide cost effective on-demand scaling of resources without the need for long-term commitment to meet bursting workload demands.</p>
<p>What is the impact of these new services, capabilities, deployment options and marketplace models on application development and deployment? These capabilities offer a wealth of new opportunities to improve the economics of DevOps but it also imperative that you design your DevOps system to take advantage of the new capabilities offered by public cloud infrastructures.</p>
<p>Next time we'll examine the attributes of a DevOps system built for the public cloud.  </p>