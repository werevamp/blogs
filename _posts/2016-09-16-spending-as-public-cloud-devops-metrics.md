---
ID: 116
post_title: 'Follow the money:  Spending as a DevOps metric'
author: Ed Lee
post_date: 2016-09-16 16:10:36
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/spending-as-public-cloud-devops-metrics/
published: true
---
<p class="p1"><span class="s1">Metrics are critically important in DevOps. This is particularly true of the public cloud, which has different management challenges than on-prem infrastructure.</span></p><p class="p1"><span class="s1">With on-prem infrastructure, the pool of shared resources is fixed and allocated based on priority.  That is, a</span><span class="s1">t a given time, is project A or B more important? </span></p><p class="p1"><span class="s1">With the public cloud, resources are on-demand, so the question becomes -  is this project worth this much money at this time?</span></p><p class="p1">The latter requires more sophisticated metrics.  But spending metrics viewed through the lens of<span class="s1"> resource allocation, security, and troubleshooting can be a big help.   </span></p><p class="p1"><span class="s1" style="font-size: 20px;"><b>Spending metrics and resource allocation</b></span></p><p class="p1"><span class="s1">Overspending is a top concern in the public cloud.  This is challenging given that resources are provisioned on demand by automated applications using APIs by users with varying degrees of fiscal visibility or responsibility :)</span></p><p class="p1">Spending metrics track how much of a resource was consumed by a project, department or user over a  time period.  DevOps should be able to generate warnings and enforce budgetary constraints when certain spending thresholds are crossed. </p><p class="p1"><span class="s1">Public cloud usage/spending is tracked in dollars, where on-prem infrastructure usage is based on utilization.   </span><span class="s1">Public cloud usage includes the cost of all operational and facility costs.  It is almost impossible to get accurate estimates of these costs for on-prem infrastructure.</span></p><p class="p1"><span class="s1">Better spending metrics lead to better resource allocation decisions. This is a key advantage of the public cloud. </span></p><p class="p1"><span class="s1" style="font-size: 20px;"><b>Spending metrics and security</b></span></p><p class="p1"><span class="s1">It may seem odd to list security as a metric. Isn't it a policy that you configure and forget? This thinking can get you in trouble, especially in the public cloud! Security is not a static problem. New threats and defenses are constantly evolving.</span></p><p class="p1"><span class="s1">All things being equal, the public cloud is as secure or more so than on-prem. The challenge is that the self-service nature of the public cloud gives more freedom to developers and operations to deploy new or change existing applications and security policies.</span></p><p class="p1"><span class="s1">It is no fun to come to work to discover that your public cloud servers have been used to launch DDoS attacks, racking up a huge networking bill!  The Internet data transfer bills for a public cloud server can be much higher than the cost of the server itself.  Even worse, someone who breaks into your system could trash or delete your production servers - requiring hours or even days to restore service.  Look for this situation by monitoring spending trends.</span></p><p class="p1"><span class="s1">Other security metrics to monitor in the public cloud include inbound connections from unfamiliar machines, particularly from the Internet (possible break in) and large increases in outbound messages to new machines on the Internet (possible DDoS attack).</span></p><p class="p1"><span class="s1" style="font-size: 20px;"><b>Spending metrics and troubleshooting</b></span></p><p class="p1"><span class="s1">Troubleshooting or tuning metrics are critical for quickly identifying failed services, performance bottlenecks and flaky machines and software. End-to-end latency metrics are particularly useful for identifying performance bottlenecks. A linearized timeline of where requests spend most of their time being serviced makes it easier to identify problem services.    V</span><span class="s1">ariance is also useful to analyze outliers that take much longer to service.</span></p><p class="p1"><span class="s1">Adding a spending breakdown to a latency analysis can focus attention on expensive component services and improve the cost efficiency of the system.</span></p><p class="p1"><span class="s1">The public cloud presents new as well as old challenges.   </span><span class="s1">Spend the time to get the right metrics and avoid out of control spending, security break-ins, or poor user experience.</span></p>