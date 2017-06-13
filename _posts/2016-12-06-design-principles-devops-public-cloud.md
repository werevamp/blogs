---
ID: 375
post_title: >
  How to Design DevOps for the public
  cloud
author: Saradhi Sreegiriraju
post_date: 2016-12-06 19:14:59
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/design-principles-devops-public-cloud/
published: true
---
<p>Earlier we wrote that <a href="http://applatix.staging.wpengine.com/devops-key-success-public-cloud-aws-azure/">DevOps is the key to success in the public cloud</a> and <a href="http://applatix.staging.wpengine.com/design-devops-automation-public-cloud-aws-azure/">DevOps must be purpose-built</a> for AWS, Microsoft Azure and Google Cloud. </p>
<p>We continue with implications for DevOps organizations and design principles DevOps systems in the public cloud.</p>
<p>Previously we pointed out key differences in public vs. private cloud - that the public cloud is API driven with new capabilities for DevOps tools to support and public cloud instances are fixed partitions vs. elastic slices.</p>
<p>How does this impact DevOps in the public cloud?</p>
<p>&nbsp;</p>
<p><strong>On-premises, DevOps is driven by CMDs and trouble tickets</strong></p>
<p>In the diagram below, we depict a typical on-prem organizational stack on the left.</p>
<p>Developers write code and submit workflows using tools like Jenkins.  Operations teams configure capacity with scripts of commands through tools like Chef.  Infrastructure teams build out new, fixed capacity as VMs.  </p>
<p>Trouble tickets are the unit of communication between teams.  A new test farm can take days or weeks to go through the process.</p>
<p><img class="alignnone  wp-image-337" src="http://applatix.staging.wpengine.com/wp-content/uploads/2016/10/DevOps-private-vs.-public-cloud-service-like-AWS-or-Azure-300x116.png" alt="devops-private-vs-public-cloud-service-like-aws-or-azure" width="582" height="225" /></p>
<p><strong>Public cloud DevOps is based on code and APIs</strong></p>
<p>Public cloud services are infinite, on-demand resources.  Self-service dramatically increases developer productivity - all you need is a credit card.   </p>
<p>Dev and Ops meld together because developers are now directly driving infrastructure with software (instead of trouble tickets).  Operations must write code (instead of CMDs and scripts). Operators must now become developers – hence the difficulty in developing skills and hiring appropriate staff</p>
<p>Governance and auditability are still critical, but not at the expense of the speed and agility that comes with the public cloud.   Without governance and control, ‘sticker shock’ results.</p>
<p>The key metric is no longer utilization, but cycle time and spending.  Auto-scaling up and down is a key success factor in both cycle time and spending, but this requires software that directly drives cloud APIs.</p>
<p>So with all these issues, what does a DevOps system for the public cloud need?   </p>
<p>&nbsp;</p>
<p><strong>DevOps systems in the public cloud should be Service driven (vs. command-driven)</strong></p>
<p>A public cloud DevOps system directly drives the public cloud infrastructure APIs.   Micro-services abstract the constantly evolving services, features, deployment and buying choices.  DevOps is API and service driven instead of VM and command oriented.     </p>
<p>This is only natural – the power of the public cloud is that it has always been ‘just an API’ – DevOps is following this progression by offering the process and the tools another layer of API abstraction.     </p>
<p>This creates consistency, scale, and reduces maintenance as DevOps integrates workflows, resources, and cloud APIs together.</p>
<p>&nbsp;</p>
<p><strong>Use containers to drive high utilization in the public cloud</strong></p>
<p>You must embrace containers to get high utilization out of DevOps in the public cloud.   Companies like Google and Facebook have been using containers for a decade to get upwards of 80% utilization and cycle times that enable hundreds of releases per day.   Why?  <a href="http://applatix.staging.wpengine.com/avoid-naive-public-cloud-or-aws-migration/">Containers enable true auto-scale to efficiently use fixed capacity</a> public cloud instances. </p>
<p>Containers and auto-scaling of resources help you avoid the ‘sticker shock’ that comes with a typical ‘lift and shift’ of on-prem VMs to the public cloud.</p>
<p>&nbsp;</p>
<p><strong>Enforce spending ‘guardrails” </strong></p>
<p>Along with self-service comes over-consumption.  At large public cloud shops, it’s not uncommon to see managers badgering developers to find and turn off unused instances.</p>
<p>We’ve written before that <a href="http://applatix.staging.wpengine.com/spending-as-public-cloud-devops-metrics/">spending is an important metric in public cloud DevOps</a> – not just for measuring overages but also for troubleshooting and security. Real-time information with policy controls is critical.    </p>
<p>Ideally, DevOps software takes a systems approach, where the system itself provides negative feedback to bring spending in line with policy in a self-correcting manner.  The system should give behavioral ‘nudges’ to developers as they consume capacity. </p>
<p>&nbsp;</p>
<p><strong>To summarize</strong>, a purpose built end-to-end DevOps system is a must for success in the public cloud.  DevOps systems must be able to natively leverage the capabilities and consumption models of public cloud efficiently. It also must preserve self-service while providing governance, auditability, and security. Lastly, a DevOps system built for public cloud must be application and service-oriented rather than server and VM oriented.</p>
<p>Imagine a large-scale test simulating hundreds of thousands of users. A DevOps system should enable developers to directly drive this process with services that control public cloud APIs for underlying infrastructure, run the tests economically (say, leveraging spot instances), be smart enough to rerun only failed tests, and then tear down the capacity afterward.</p>
<p>Now, imagine what it takes to execute this use case, along with the economic and agility benefits of doing it well, several times a day, and you can appreciate why DevOps is the key to the public cloud and why a purpose built DevOps system that natively leverages public cloud capabilities is mandatory to success in public cloud.</p>