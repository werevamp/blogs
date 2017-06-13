---
ID: 704
post_title: 'Making Kubernetes Production Ready &#8211; Part 3'
author: Harry Zhang
post_date: 2017-06-06 00:54:07
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-3/
published: true
---
<div data-canvas-width="237.1792499999999">
<p class="p1"><span style="font-size: 20px;"><strong>Choreographing Micro-services<br />
 </strong></span>In <a href="https://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-2/">Part 2</a>, I discussed how to tune workload and performance related knobs in Kubernetes for production. However, this is not the end of story. The way you manage your micro-services should also be carefully choreographed to work smoothly with Kubernetes and ensure a stable system. At Applatix, we run large Kubernetes clusters with a mix of DevOps workloads. Some workloads are short-lived while other run for long periods of time. Some use very little resources while other use lots of CPU, memory and disk. Meanwhile, Applatix micro-services need to frequently talk to Kubernetes to perform DevOps tasks as well. As a result, we need to cater to the stability needs of the most demanding workload. In other words, we favor stability over efficiency. Here are some things we do to ensure the stability of our Kubernetes clusters.</p>
<p class="p1"> </p>
</div>
<div data-canvas-width="752.0947049999996">
<div data-canvas-width="237.1792499999999">
<p class="p1"><span style="font-size: 20px;"><strong>Protect Nodes<br />
 </strong></span>Not only the master node but also every minion should be protected from overload and resource exhaustion. Minions can be protected by properly configuring its <strong>kubelet</strong>. We set the "<em>--max-pods</em>" flag in <strong>kubelet</strong> to limit the number of pods admitted for scheduling. It is not uncommon for large batch jobs or runaway applications to spawn very large numbers of Pods in a short period of time. Without limiting Pod admission, <strong>Kubelet</strong> and other system Pods can be overwhelmed, resulting in unresponsive and unstable nodes. In combination with limiting Pod admissions, you should reserve the resources needed by Kubernetes and other high priority services by using the "<em>--</em><em>kube-reserved</em><strong>"</strong> and "<em>--system-reserved</em>" flags in <strong>kubelet</strong>. <strong>Kubelet</strong> will reserve and deduct these resources from the node's total allocable resource when registering node with the Kubernetes master. These limits will be followed by <strong>kube-scheduler</strong> when it schedules Pods. Finally, setting appropriate resource request and limit values for all Pods is equally important to protect nodes, as this ensures no potential runaway Pod can monopolize resources and starve other Pods or even introduce host level OOM (out of memory) kills.</p>
<p class="p1"> </p>
<p class="p1"><span style="font-size: 20px;"><strong>Throttle Object Creations<br />
 </strong></span>Originally, we allowed large numbers of DevOps tasks to be submitted to Kubernetes even if the cluster did not have sufficient spare resources to run them. We believed that Kubernetes can store the backlog of work and schedule them as existing tasks finished. This strategy turned out to introduce instability in Kubernetes because as pending tasks accumulated, Kubernetes master would become overloaded trying to schedule too many Pods at once. These attempts to schedule massive backlogs of tasks dramatically increased CPU and memory consumption of <strong>kube-scheduler</strong>, <strong>kube-apiserver</strong> and <strong>etcd</strong>, frequently causing OOMs and made Kubernetes unresponsive. To improve this situation, we implemented high-level admission control algorithms to control both the number and rate of tasks submitted to Kubernetes and avoid resource deadlocks when running workflows that require sequencing the execution of multiple Pods. This made our Kubernetes clusters much more stable under high loads.</p>
<p class="p1"> </p>
<p class="p1"><span style="font-size: 20px;"><strong>Back-off and Retry<br />
 </strong></span>In an Applatix cluster, some system-level micro-services need to communicate regularly with Kubernetes. These services include the cluster auto-scaler, workflow and application management engines, and various daemons for managing user Pods. As these micro-services are distributed, they can generate a burst of activities cluster-wide, and may of these activities involve communicating with Kubernetes. Since we use Kubernetes API limits to avoid overloading the <strong>kube-apiserver</strong>, the excess API calls will be rejected and clients will receive error code <strong>429</strong>. In this case, exponential back-off and retry are used to cool down the cluster while ensuring progress. It is also important to add jitter in the back-off to prevent synchronized API request storms after a <strong>kube-apiserver</strong> crash. Kubernetes has clearly defined its API conventions, please refer to their documentation <span class="s1"><a href="https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md">[1]</a> </span>for more information.</p>
<p class="p1"> </p>
</div>
<div data-canvas-width="81.246315">
<p class="p1"><span style="font-size: 20px;"><strong>Avoid Talking to Master if Possible<br />
 </strong></span>API Calls to Kubernetes master is relatively expensive, especially when you can get the same information from local services. For example, <strong>kubelet</strong> can be accessed via the host network and can open a read-only port, using the "<em>--read-only-port</em>" flag, to serve a limited but useful set of read-only APIs. These APIs are not documented, but can be found by reading the source code <a href="https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/server/server.go"><span class="s1">[2]</span></a>:</p>
<ul>
	<li class="p1">
<p class="p1">"<strong>/pods</strong>" path can get all Pod information on that host:</p>
</li>
</ul>
<p><img class="alignnone wp-image-708 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/06/Pasted-image-at-2017_06_05-04_17-PM.png" alt="" width="706" height="791" /></p>
<ul>
	<li class="p1">"<strong>/spec/</strong>" path can get host information</li>
</ul>
<p><img class="alignnone wp-image-709 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/06/Pasted-image-at-2017_06_05-04_21-PM.png" alt="" width="705" height="529" /></p>
<ul>
	<li class="p1">"<strong>/stats/</strong>" path shows cadvisor <a href="https://github.com/google/cadvisor"><span class="s1">[3] </span></a>stats of the host, etc.</li>
</ul>
<p><img class="alignnone wp-image-710 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/06/Pasted-image-at-2017_06_05-04_23-PM.png" alt="" width="705" height="849" /></p>
<p class="p1">Finally, in some cases, it may be better to get information from the cloud provider to retrieve cloud level metadata such as IP addresses, region info, etc. instead of from Kubernetes master. For AWS, <a href="http://169.254.169.254"><span class="s1">http://169.254.169.254 </span></a>is the "magic" URL you can use <a href="http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-instance-metadata.html#instancedata-data-retrieval"><span class="s2">[4]</span></a>, and for GCP/GKE, <a href="http://metadata.google.internal/computeMetadata/v1/"><span class="s1">http://meta</span><span class="s1">data.google.internal/computeMetadata/v1/</span></a> is the URL to access the metadata server <a href="https://cloud.google.com/compute/docs/storing-retrieving-metadata"><span class="s2">[5]</span></a>.</p>
<p class="p1"> </p>
<p class="p1"><span style="font-size: 20px;"><strong>Cross Monitoring Nodes</strong></span><br />
 Like any virtual machine, Kubernetes nodes can suffer from many software related problems. For example, Linux kernel bugs are often trigger by interactions with container managers such as Docker that can cause kernel crashes <a href="https://github.com/kubernetes/kops/issues/874#issuecomment-278824037"><span class="s1">[6]</span></a>, Docker can hang for long periods of time and cause Kubernetes to end up with inconsistent Pod/container status, critical Kubernetes components can get OOM killed and lose connection with each other. Such situations are much more frequent on nodes that have high levels of churn (frequent Pod starts and stops). Kubernetes relies highly on synchronizing states of objects between minion and master. As a result, such software related problems will initially affect only a single Kubernetes node, but can quickly escalate and affect the entire cluster if you do not react in a timely manner to fix the unhealthy node.</p>
<p class="p1">At Applatix, we run special micro-services and system daemons for monitoring critical cluster components such as Docker, <strong>Kubelet</strong>, and Kubernetes master. In the event, any of these system components go into non-recoverable inconsistent states (e.g., fail to restart a hanging docker daemon), our monitoring service will terminate the node, and an identical replacement node will be automatically launched. Although terminating an unhealthy node can introduce additional churns inside the cluster by killing and restarting all Pods running on it, and will take time for the replacement node to come up, it is much better than waiting conservatively for that node to (possibly) recover, as having unhealthy nodes hanging around will accumulate and spread local errors and finally make the entire cluster unusable.</p>
<p class="p1"> </p>
<p class="p1"><span style="font-size: 20px;"><strong>Conclusion<br />
 </strong></span>At Applatix, we took huge efforts to learn about how Kubernetes actually behaves under DevOps production workloads. After configuring Kubernetes and architecting our micro-services properly, Kubernetes has become very stable and responsive. This concludes my 3-part blog series about sharing our experiences of making Kubernetes production ready. Kubernetes plays a critical role in modern containerized cloud platforms. I hope it encourages you to try using Kubernetes for your production workloads, and that it saves you much pain in making your Kubernetes cluster production ready. If you find this useful or have questions, just drop us a note at <a href="mailto:info@applatix.com"><span class="s1">info@applatix.com</span></a>!</p>
<p class="p1"> </p>
<p class="p1"> </p>
<p class="p1"><strong>References<br />
 </strong><span style="font-size: 12px;">Ref [1]: <a href="https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md">https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md<br />
 </a></span><span style="font-size: 12px;">Ref [2]: <a href="https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/server/server.go">https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/server/server.go<br />
 </a></span><span style="font-size: 12px;">Ref [3]: <a href="https://github.com/google/cadvisor">https://github.com/google/cadvisor<br />
 </a></span><span style="font-size: 12px;">Ref [4]: <a href="http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-instance-metadata.html#instancedata-data-retrieval">http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-instance-metadata.html#instancedata-data-retrieval<br />
 </a></span><span style="font-size: 12px;">Ref [5]: <a href="https://cloud.google.com/compute/docs/storing-retrieving-metadata">https://cloud.google.com/compute/docs/storing-retrieving-metadata<br />
 </a></span><span style="font-size: 12px;">Ref [6]: <a href="https://github.com/kubernetes/kops/issues/874#issuecomment-278824037">https://github.com/kubernetes/kops/issues/874#issuecomment-278824037</a></span></p>
<p class="p1"> </p>
<p class="p2"><em>Harry Zhang is a Member of Technical Staff at Applatix. He works with platform team focusing on building containerized micro-services with Kubernetes and AWS. Reach him on <a href="https://www.linkedin.com/in/hao-h-zhang-74b39a41/">LinkedIn</a> for questions, comments, or information about Applatix.</em></p>
</div>
</div>