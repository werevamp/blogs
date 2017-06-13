---
ID: 381
post_title: 'Creating Kubernetes Clusters: The Kube-up vs Kops choice'
author: Shrinand Javadekar
post_date: 2016-12-14 15:40:14
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/create-kubernetes-clusters-kube-up-vs-kops/
published: true
---
<p>Mark Twain said, “The secret to getting ahead is to get started”. Project teams, just like ours, are aiming to get ahead by building applications on top of Kubernetes. However, the first question about getting started is simply “how to”.</p>
<p>There are several options for creating a Kubernetes cluster on AWS. Some of these can be found on <a href="http://kubernetes.io/docs/getting-started-guides/aws/">this</a> page.</p>
<p>The prominent ones we looked at were <strong>kube-up</strong> and <strong>kops - </strong>Both of these have been developed by engineers from Google and promise getting off the ground easily.</p>
<p>The initial goal was simply to create a test cluster on AWS and play with it. While doing that, we also wanted to see the flexibility and options provided by the two methods.</p>
<p>The workload we eventually wanted to run on Kubernetes was varying in nature. Therefore, it was important that there be enough knobs to create appropriately sized and configured clusters. If the tool provides only limited options, it should at least be extensible so that we could add any missing functionality.</p>
<p>Here are some of the things that we learned along the way.</p>
<p><strong>Kube-up:</strong></p>
<ul>
	<li>Kube-up is a shell script that uses the AWS command line client to deploy a cluster. It keeps things super simple and is easy to modify.</li>
	<li>The config options are basically environment variables. There are no easy ways of discovering these options other than looking at the code. But once you find them, they work fine.</li>
	<li>Unfortunately, shell scripts are not the easiest to read and maintain. And this is made worse by the fact that the AWS command line client lends itself to long commands with many options. Modifying the script to suit one’s needs is possible, but needs a careful reading of the code. The kube-up script is run with the “-e” option which means that if any command fails, the script exits.</li>
	<li>AWS imposes several rate limits on resources and their APIs calls occasionally fail for no good reason. Therefore, calls to AWS services should be backed up by solid retry logic. Adding sophisticated retry logic into the shell script is not trivial. In kube-up, there is no retry logic. If any AWS call fails, the script exits. The idea is to rerun kube-up until it succeeds. Fortunately, there is logic to detect resources that were already setup and the script automatically skips these steps when rerun.</li>
	<li>One of the big limitations of the Kubernetes cluster deployed by kube-up is that the master node is not deployed in a highly available configuration. If the master node dies for any reason, there is no easy way of getting it back. Users have to manually spin up an instance from the appropriate AMI and perform other steps such as attaching the elastic IP, attaching the EBS volume, etc.</li>
	<li>Also, the IP address of the master is baked into the minions during cluster creation and cannot be easily changed (actually it is baked into the minion nodes’ user-data script that runs every time a minion starts). This means that there can only be one master node and, if it dies, a new one needs to be created with the exact same IP address as the one that died. The IP address is governed by the environment variable MASTER_INTERNAL_IP.</li>
	<li>There was some attempt made to support kubernetes cluster upgrades. But it was never finished and it looks like they’ve given up on it.</li>
	<li>There is a separate script called “kube-down” that is used for tearing down clusters.</li>
</ul>
<p>Currently, kube-up is *not* the recommended way of deploying Kubernetes clusters on AWS. The kube-up effort was ditched in favor of building a more sophisticated tool, ‘kops’.</p>
<p><strong>Kops:</strong></p>
<ul>
	<li>Kops is available as an executable binary. It is written in GOLANG.</li>
	<li>One of the first things to notice about kops is that it has a nice help menu. So no more reading shell scripts to find what options to set. Win!</li>
	<li>The next noticeable difference is the need to specify the STATE_STORE; an s3 bucket into which the cluster configuration is uploaded. This is good because at any point in time if one wants to know what options were used to create a given cluster, one can simply read the configuration from the S3 bucket.</li>
	<li>The other benefit of the config being in S3 is that it allows easy modification. One simply changes the config stored in s3 (or run “kops edit”) and then run “kops upgrade” to make the changes take effect.</li>
	<li>The above also applies for upgrading Kubernetes versions. Changing the kubernetes version and running “kops upgrade” does an in-place upgrade of Kubernetes.</li>
	<li>Unlike kube-up, clusters created using kops are configured to have highly available master nodes. kops basically runs one or more master nodes in an autoscaling group. So you can always have more than one master node and even if a master node goes down, AWS will automatically start another one. This also implies that the MASTER_INTERNAL_IP is not baked into the minions.</li>
	<li>Kops creates DNS entries in Route53 and uses these instead of raw IP addresses. This allows seamless failover of services and uses load-balancing for better performance.</li>
	<li>The drawback of kops though is that modifying it to suit your own needs has a steep learning curve. Modifying the “kube-up” bash script is easier.</li>
</ul>
<p>One of the other great things about Kubernetes, in general, is the community. It is very active and very supportive. The pace of developing new features, especially in kops is awesome.</p>
<p>Comparing kube-up with kops, I think there’s little doubt that kops will become, if not already, the best way to deploy Kubernetes on AWS.</p>