---
ID: 26
post_title: 'Public vs. Private cloud:  Avoid a naive deployment'
author: Ed Lee
post_date: 2016-03-21 20:20:24
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/avoid-naive-public-cloud-or-aws-migration/
published: true
---
<p>Ever since Amazon started reporting AWS revenue separately from the rest of their business, we've been surprised at how large it is and how quickly it is growing. It's not just startups and SaaS companies using public cloud anymore.</p>
<p>Companies suddenly realize that the race to the cloud is on, and those who quickly figure out how to use the public cloud could gain a competitive advantage while the rest are left behind.</p>
<p>The public cloud, however, is a very different beast from on-prem private clouds. Effectively using the public cloud involves far more than lifting and shifting your VMs into the cloud.</p>
<p>The public cloud offers nearly infinite resources on demand. In a private cloud, fixed resources are allocated by prioritizing one project over another.</p>
<p>While in a public cloud, any needed resources can be dynamically allocated and the question becomes, "Is this project worth doing at this time?"</p>
<p>In the private cloud, you must evaluate the relative value of one project vs. another.   In the public cloud, you must understand the business value of the work to be performed.</p>
<p><strong>This seemingly simple shift in focus has profound consequences for how we value and schedule work.</strong></p>
<p>In the public cloud, the ease with which resources can be allocated and consumed brings with it new fears and responsibilities.</p>
<p>While the self-service and programmable capabilities of the public cloud can empower teams and individuals, without appropriate policies and controls, a single individual or runaway program could easily consume an entire month of your cloud budget in a single day! Not a pleasant thought.</p>
<p>Even something as basic as the VM is different in the private and public clouds.</p>
<p>A VM running on on-prem virtualization infrastructure is an elastic slice of a server. That is, any CPU or memory resources not used by a VM is available for use by other VMs.</p>
<p>In contrast, a VM, or instance, in the public cloud is more like a fixed partition of a server with dedicated resources. Any CPU or other resources not used by an instance is wasted. That is, dedicated resources provide good performance isolation between instances, which is particularly important in multi-tenant environments, but also generates more waste.</p>
<p>The result of these and many other differences between public and private clouds is that a naive public cloud implementation will be plagued by "cloud sprawl," inefficient use of cloud resources, and cloud sticker shock.</p>
<p>Such negative experiences could delay the adoption of public cloud and even retrenchment back to private clouds while the handful of companies that tames the public cloud will have the opportunity to race ahead.</p>