---
ID: 545
post_title: >
  What are Containers? And what are their
  benefits in AWS and the public cloud?
author: Saradhi Sreegiriraju
post_date: 2017-05-07 10:35:02
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/what-are-containers-and-what-are-their-benefits-in-aws-and-the-public-cloud/
published: true
---
<p class="p1"><span class="s1">Last time we talked about <a href="https://applatix.staging.wpengine.com/best-definition-devops/">the definition of DevOps</a> and why <a href="https://applatix.staging.wpengine.com/aws-public-cloud-not-equal-private-cloud/">DevOps is different in Public cloud services like AWS</a>.<span class="Apple-converted-space">  </span>Next in this series “<a href="https://applatix.staging.wpengine.com/video-slides-webcast-devops-aws-different-can-containers-help/">DevOps is different!<span class="Apple-converted-space">  </span>How can containers help?</a>“ we talk about containers and why they have many benefits in the public cloud. </span></p>
<p class="p1"><span class="s1"><b>What is a container?</b></span></p>
<p class="p1"><span class="s1">Consider the diagram below.  </span><span class="s1">On the left, you have a traditional virtualization model using virtual machines (VMs) with a hypervisor running an OS on a server (Dell, HP, etc.).</span></p>
<p class="p1"><span class="s1">On the right, you have a much lightweight virtualization infrastructure.  The containers are sharing a lot of the OS in the guest.  This is a lot more resource efficient.</span></p>
<p class="p1"><img class="alignleft wp-image-506 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/04/what-are-containers.png" alt="" width="2324" height="1314" /></p>
<p class="p1"><span class="s1">As a result, the way you define a container - with a container image - is a lot smaller than a VM image. You are packing up just the necessary libraries and binaries to run the application and not the entire OS. </span></p>
<p class="p1"><span class="s1">Because each container is packaged with its own custom execution environment, it will run the same on a developer laptop as in production.</span></p>
<p class="p1"><span class="s1">This level of portability is extremely valuable!  Or at least that is the promise - but as with any fast moving technology, it can be hard to do easily and reliably in practice. </span></p>
<p class="p1"><span class="s1"><b>How do containers help with DevOps in the public cloud? </b></span></p>
<p class="p1"><span class="s1">Containers provide <em>lightweight virtualization</em>.  Containers are how leading technology companies like Google and Facebook have been squeezing agility and efficiency from their private clouds for over a decade.<span class="Apple-converted-space">  </span>Containers enable them to deploy multiple times per day and get 80% utilization of their massive clouds. </span></p>
<p class="p1"><span class="s1">Compare that with a typical on-prem cloud utilization of 25-35%, which is what you get if you lift-and-shift your VMs from on-prem to the cloud without using containers.  This is one of the causes of AWS sticker shock!</span></p>
<p class="p1"><span class="s1">When lift-and-shift VMs to the cloud, you are essentially de-virtualizing the infrastructure you spent so much time virtualizing in the first place! You are turning each elastic VM back into a fixed slice of a physical server.</span></p>
<p class="p1"><span class="s1">Containers provide very <em>high portability and efficiency</em>.   It’s ideal for DevOps in dynamic environments like the public cloud where instances can come and go on demand, and you can use marketplace based instruments like spot instances.</span></p>
<p class="p1"><span class="s1">By leveraging containers and micro-services, you can dynamically scale your application to match the load and pack a lot more work onto a single instance.</span></p>
<p class="p1"><span class="s1">You can also track metrics at a much more granular level than a server or VM. You can do monitoring and perform analytics at a job or app level, which is far more relevant to your business.</span></p>
<p class="p1"><span class="s1">And that is the goal of DevOps - not only you are delivering software and fixing bugs faster, you are solving more problems that are relevant to your business!</span></p>
<p class="p1"><span class="s1">For all these reasons above, we think containers and the public cloud go together like Peanut Butter and Jelly!</span></p>
<p class="p1"><span class="s1">Next time we'll get into how we got our own battle scars - obstacles encountered and lessons learned in working with containers in AWS.  </span></p>