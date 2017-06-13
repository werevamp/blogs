---
ID: 434
post_title: An AWS instance is not a VM!
author: Saradhi Sreegiriraju
post_date: 2017-02-28 19:07:24
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/aws-instance-not-vm/
published: true
---
<p>Last time we talked about how the <a href="https://applatix.staging.wpengine.com/aws-public-cloud-not-equal-private-cloud/">public cloud is different</a> - not just a larger version of your private cloud.  The public cloud is API driven which forces a different operating approach for public cloud services like AWS, Azure, and Google Cloud.   </p>
<p>This time we discuss another major difference in the public cloud vs. the private cloud - one that drives an entirely different mentality in how you think about capacity and prioritizing workloads.   This is the difference between a public cloud instance and a VM.</p>
<p>&nbsp;</p>
<p><strong>An AWS instance is not a VM!</strong></p>
<p>If you look at how private cloud infrastructure is managed – it revolves around the server.  Much of private cloud operations is how you allocate, configure, and manage VMs on these servers.  </p>
<p>The Public cloud of course also has VMs (called instances).  But these are really rather different.</p>
<p>In the private cloud, any resources not used by one VM on a server can and are used by other VMs on that server.</p>
<p>But in the public cloud, the VM or Instances is instead a <strong>fixed</strong> slice of a server.</p>
<p>This is important in the public cloud to provide isolation between you and other customers. Otherwise, things like debugging would be very hard.</p>
<p>But the implication of this difference is that everything you don’t use on your instance goes unused and is wasted.</p>
<p><img class="alignleft size-medium wp-image-440" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/02/private-cloud-vms-vs-aws-instances-1-300x157.png" alt="" width="300" height="157" /></p>
<p>So isn’t the trick simply to right size your server?    AWS offers dozens of different instance types you can use.  </p>
<p>But people find it really difficult to right size, and your workloads can change over the course of the day or over longer stretches of time.</p>
<p>So many people usually ‘right size’ their instances once, and don’t come back to it very often.  And this won’t turn your instances off at night or during other low usage periods.     With this approach, you are not really taking advantage of the capability to auto-scale. </p>
<p>So fundamentally, a Public cloud instance is more like a server than a VM and you need a lighter weight virtualization technology to take advantage of full benefits of public cloud.   </p>
<p><strong>Otherwise, you are effectively ‘de-virtualizing’ many of the workloads you took such great pains to virtualize in the private cloud.</strong></p>
<p>This is why the ‘lift and shift’ of private cloud VMs to public cloud instances as an initial strategy for migrating to the public cloud may often result in sticker shock.  You are underutilizing your fixed instances in the public cloud</p>
<p>So how do you prevent this?    If you are going all-in on the public cloud, you will need to take a page from what the largest hyper-scale Web companies have been doing for more than a decade – use containers!</p>
<p>But containers come with their own challenges.  More on that in our next post!</p>
<p>Or, catch our webinar “AWS is different!  How can containers help?” below or on our <a href="https://www.youtube.com/watch?v=mQjMuzSrtGs">cloud best practices YouTube</a> channel.</p>
<p><iframe src="https://www.youtube.com/embed/mQjMuzSrtGs" width="400" height="250" frameborder="0" allowfullscreen="allowfullscreen"></iframe></p>
<p>&nbsp;</p>