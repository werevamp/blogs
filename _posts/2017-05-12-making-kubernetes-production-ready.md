---
ID: 671
post_title: 'Making Kubernetes Production Ready &#8211; Part 1'
author: Harry Zhang
post_date: 2017-05-12 21:30:26
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/making-kubernetes-production-ready/
published: true
---
<p><span class="inline-comment-marker valid" data-ref="ba3a2b1a-9ac2-43d9-9206-ecccf2045cac">What does it take to run 7500 containers</span> supporting 300 micro-service based applications while concurrently running a continuous integration and testing (CI) job once every 10s of seconds on a Kubernetes cluster that can autoscale up to 30 minions (240 CPU cores and 1TB memory)?</p>
<p>In this blog series, I'm going to share our war stories with you, and save you the pain of learning the lessons on your own.</p>
<p>&nbsp;</p>
<p><strong>First things first.</strong> There are many ways to create a Kubernetes cluster. Whether you use kops, kubeadm, or kube-up, it's a very satisfying moment when you finally connect to the Kubernetes API and launch your first Pod! But this joy may be short lived if you expect to have a stable, scalable, production-ready cluster with no additional work.</p>
<p>The default configurations for Kubernetes components are not designed for heavy and dynamic workloads characteristic of DevOps environments and micro-service based application deployments where containers are quickly created and destroyed. As a result, you will see unstable behaviors from many core Kubernetes components. At times, it may seem like your cluster is going crazy! Welcome to the brave new world.</p>
<p>At Applatix, we <span class="inline-comment-marker valid" data-ref="4c8d5a55-d471-4f90-a872-cd823a471ad4">invested great efforts to understand fundamental Kubernetes behaviors under various loads </span>and learned how to configure clusters so that they run stably under heavy, dynamic load conditions. We have also modified our core system-level micro-services to ensure that they work with Kubernetes to maintain platform stability.</p>
<p>&nbsp;</p>
<p><strong>Our production workload.</strong> The production workloads we run are <span class="inline-comment-marker valid" data-ref="2d4c3cf7-07a6-41fd-9759-27e0a6eb95c1">challenging</span> on many fronts.</p>
<ul>
	<li><span class="inline-comment-marker valid" data-ref="4a8eb529-39b7-459d-a659-50f5ea4c4a3c">We need to handle mixed workloads with greatly varying time spans.</span> Some <span class="inline-comment-marker valid" data-ref="1f8df63f-c70d-4d4d-82ed-21a6b64eacd6">workloads</span> are short-running DevOps jobs that last up to 10s of minutes while others are long-running applications that can run from hours to months.</li>
	<li><span class="inline-comment-marker valid" data-ref="28a6f6e6-8789-4094-95c2-d13f092659b2">There can be hundreds of pods created and destroyed every minute due to automatically triggered CI jobs.</span></li>
	<li><span class="inline-comment-marker valid" data-ref="02a7ccf6-c0d5-46b4-85d0-3251de99a859">Many container workloads from users are unpredictable. </span><span class="inline-comment-marker valid" data-ref="b91a1000-e048-4a6b-aa1f-91fbb681b336">Some workloads may be misconfigured and provide inaccurate estimates of expected resource usage</span>.</li>
</ul>
<p><span class="inline-comment-marker valid" data-ref="44639609-973e-4b97-b2f8-b5888db80722">The following is based on a single master cluster running Kubernetes 1.4.3.</span> <span class="inline-comment-marker valid" data-ref="61831d1b-cafa-43dd-a488-72c5a52f0b35">Each micro-service consists of one or more pods</span> and other infrastructure such as volumes with run times varying from hours to months. Each CI task is a multi-step workflow, that performs code checkouts, parallelized builds and tests, and creates and uses fixtures (containerized services) for running complex tests.</p>
<p>During busy times, we have about 2300 Pods (~7500 containers) running on a single cluster and the cluster will autoscale up to 30 m3.2xlarge minions (240 CPU cores and 1TB memory cluster wide). We also run larger stress tests for shorter periods of time but the production cluster has the greatest<span class="inline-comment-marker valid" data-ref="3f728284-c6cd-4b31-91a3-1c9d40afe379"> longevity</span>. </p>
<p>&nbsp;</p>
<h3 id="MakingKubernetesProductionReadyatApplatix-UsingDefaultValuesIsDangerous"><strong>Using Default Values Is Dangerous</strong></h3>
<p>Running Applatix workload with default configurations for Kubernetes configurations is dangerous. There are two broad categories of negative consequences:</p>
<ul>
	<li><span class="inline-comment-marker valid" data-ref="ae8895a1-5e52-4ed7-97c5-161f409ec144">resource related</span></li>
	<li><span class="inline-comment-marker valid" data-ref="ae8895a1-5e52-4ed7-97c5-161f409ec144">performance related</span></li>
</ul>
<p>Kubernetes components have fixed default configurations for their resource consumptions. For example, maximum inflight mutating / non-mutating calls for API server are set to 200 and 400 respectively; lookup cache size for Replication Set, Replication Controller, and Daemon Set are 4GB, 4GB and 1GB respectively for controller manager; API query per second for scheduler is 100.</p>
<p>Using limits that are too large will cause instability by significantly increasing CPU and memory consumption during peak loads while limits that are too small will result in suboptimal performance and wasted resources.</p>
<p>For example, for a large Kubernetes cluster, you may want to use an instance with 128GB of memory for the master. However, Kubernetes would not be able to effectively use such a large instance with the default limits leading to poor master node performance which can cause clients communicating with the master to suffer timeouts.</p>
<p><span class="inline-comment-marker valid" data-ref="28f5dc98-c9b1-4d6a-9b75-31ad9cd4a571">Conversely</span>, when you use an instance with 8GB memory for the master, Kubernetes controller manager can use up most of the memory making the cluster unstable. To avoid either extreme, <strong>you must tune component limits for the maximum size of the cluster and type of load you expect.</strong></p>
<p>At Applatix, we deploy Kubernetes clusters of various sizes depending on customers' needs. We provision resources according to cluster sizes. If Kubernetes limits (such as the default values) are too high compared with the amount of available resources, OOM (Out of Memory) conditions can quickly occur on a master node when workloads add up, resulting in repeated killing of important system components which destabilizes the system.</p>
<p>Although those system components can be restarted automatically, the restarted containers will just balloon up again and be repeatedly killed due to the pending load. As system components get repeatedly killed, object status, such as Pod, Service, and Persistence Volume Claims may become inconsistent cluster wide. This can quickly become catastrophic as the confusion starts to spread from one micro-service to another.</p>
<h3 id="MakingKubernetesProductionReadyatApplatix-UsingDefaultValuesIsDangerous"><strong>To Recap</strong></h3>
<p>Careful configuration is essential for running Kubernetes clusters in production. The configuration must take into account the desired maximum size of the cluster as well as the expected workloads. Without the correct configurations, a Kubernetes cluster will suffer from instability and/or waste precious resources.</p>
<p>In subsequent blogs, I will delve into the <a href="http://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-2/">Kubernetes configuration details</a> and go over how we architected our <a href="https://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-3/">micro-services for stability and availability</a>. The configuration settings and the architectural decisions allow us to reliably run large Kubernetes clusters with hundreds of micro-services and thousands of pods in production for long periods of time. Stay tuned!</p>
<p>&nbsp;</p>
<p><em>Harry Zhang is a member of technical staff at Applatix. He works with platform team focusing on building containerized micro-services with Kubernetes and AWS. Reach him on <a href="https://www.linkedin.com/in/hao-h-zhang-74b39a41/">LinkedIn</a> for questions, comments, or information about Applatix. </em></p>