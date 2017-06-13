---
ID: 689
post_title: >
  Making Kubernetes Production Ready –
  Part 2
author: Harry Zhang
post_date: 2017-05-24 01:11:23
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-2/
published: true
---
<div data-canvas-width="758.7463799999991">In <a href="https://applatix.staging.wpengine.com/making-kubernetes-production-ready/">Part 1</a>, I described the production workloads we run at Applatix and the many reasons why Kubernetes default configurations are ill-suited for running these workloads. In this post, I'm going to go over exactly how we configured Kubernetes to handle these production workloads. After much research and pouring over the source code, we found more than 30 performance and workload related knobs provided by Kubernetes components: some knobs are documented and some are not. We carefully tuned and tested many combinations of Kubernetes knob settings to identify the small set of configurations that are reliable and provide good performance for our workloads.</div>
<div data-canvas-width="758.7463799999991"> </div>
<div data-canvas-width="758.7463799999991"><span style="font-size: 20px;"><strong>Kubernetes Overview</strong></span></div>
<div data-canvas-width="758.7463799999991"> </div>
<div data-canvas-width="758.7463799999991"><img class="alignnone wp-image-690 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/05/harry-image.jpg" alt="" width="673" height="763" /></div>
<div data-canvas-width="758.7463799999991"> </div>
<div data-canvas-width="758.7463799999991">
<p class="p1">As a quick overview, consider the above block diagram of Kubernetes components and how they interact with each other.</p>
<ul>
	<li class="p1"><strong>kube-apiserver</strong>: Kubernete's REST API entry point that processes operations on Kubernetes objects, i.e. Pods, Deployments, Stateful Sets, Persistent Volume Claims, Secrets, etc. An operation mutates (create / update / delete) or reads a spec describing the REST API object(s)</li>
	<li class="p1"><strong>Etcd</strong>: A highly available key-value store for <strong>kube-apiserver</strong></li>
	<li class="p1"><strong>kube-controller-manager</strong>: Runs control loops that manage objects from <strong>kube-apiserver</strong> and perform actions to make sure these objects maintain the states described by their specs</li>
	<li class="p1"><strong>kube-scheduler</strong>: Gets pending Pods from <strong>kube-apiserver</strong>, assigns a minion to the Pod on which it should run, and writes the assignments back to API server. <strong>kube-scheduler</strong> assigns minions based on available resources, QoS, data locality and other policies described in its driving algorithm</li>
	<li class="p1"><strong>kubelet</strong>: A Kubernetes worker that runs on each minion. It watches Pods via <strong>kube-apiserver</strong> and looks for Pods that are assigned to itself. It then syncs these Pods if possible. The procedure of Syncing Pods requires resource provisioning (i.e. mount volume), talking with container runtime to manage Pod life cycle (i.e. pull images, run containers, check container health, delete containers and garbage collect containers)</li>
	<li class="p1"><strong>kube-proxy</strong>: A network proxy that reflects Service (defined in Kubernetes REST API) that runs on each node. Watches Service and Endpoint objects from <strong>kube-apiserver</strong> and modifies the underlying kernel iptable for routing and redirection.</li>
</ul>
<p class="p2"><span style="font-size: 20px;"><strong>Knobs, knobs, and more knobs</strong></span></p>
<p class="p1">A rule of thumb is that Kubernetes workload and resource consumption are directly related to the number of Pods and rate of Pod churn (starts &amp; stops) cluster wide. Based on benchmarking information about Kubernetes <a href="http://blog.kubernetes.io/2016/07/kubernetes-updates-to-performance-and-scalability-in-1.3.html">[1]</a> <a href="http://blog.kubernetes.io/2016/03/1000-nodes-and-beyond-updates-to-Kubernetes-performance-and-scalability-in-12.html">[2]</a> and perusing Kubernetes source code <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/server.go#L505">[3]</a>, Kubernetes officially recommends 32-cores and 120GB-memory for a 2,000 node, 60,000 Pod cluster. Although a certain fraction of mutating workloads is added in such benchmarks <a href="http://blog.kubernetes.io/2016/03/1000-nodes-and-beyond-updates-to-Kubernetes-performance-and-scalability-in-12.html">[2]</a>, the benchmark workloads are relatively static compared to a typical DevOps workload, where there can be hundreds of Pods spinning up and down every minute. With the Applatix production workload, the Kubernetes master components' memory usage is very sensitive to Pod churn, and much more memory is needed than compared with the official recommendation. In general, careful consideration of your particular workload is needed to properly configure your Kubernetes cluster for the desired level of stability and performance.</p>
<p class="p1">The following is a summary of the knobs that we adjust for Applatix's production clusters. As a reminder, our usage of these flags was tested specifically for high churn workloads. Your configurations may vary:</p>
<p class="p1"><strong>kube-apiserver </strong><a href="https://kubernetes.io/docs/admin/kube-apiserver/">[4]</a> <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/options/options.go">[5]</a> <a href="https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go">[6]</a></p>
<table class="t1" style="height: 594px; border-color: #000000; width: 816px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 87px; border-color: #595959;" valign="top">
<p class="p1"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 163px;" valign="top">
<p class="p1"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 244px;" valign="top">
<p class="p1"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 314px;" valign="top">
<p class="p1"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 87px;" valign="top">
<p class="p2" style="text-align: left;"><span style="font-size: 12px;"> Throttle API requests</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 163px;" valign="top">
<p class="p3" style="text-align: left;"><span style="font-size: 12px;"><b>--max-requests-inflight</b></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 244px;" valign="top">
<p class="p2" style="text-align: left;"><span style="font-size: 12px;">This flag will limit the number of API calls that will be processed in parallel, which is a great control point of kube<strong>-</strong>apiserver memory consumption. The API server can be very CPU intensive when processing a lot of requests in parallel.</span></p>
<p class="p5" style="text-align: left;"><span style="font-size: 12px;">With the latest Kubernetes release, they provide more fine-grained API throttling mechanisms with "<b>--max-requests-inflight</b>" and "<b>--max-mutating-requests-inflight</b>"</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 314px;" valign="top">
<p class="p2" style="text-align: left;"><span style="font-size: 12px;">Adjust this value from the default (400) until you find a good balance. If it is too low, you will see too many request-limit-exceed errors. If it is too high, you will see</span></p>
<p class="p6" style="text-align: left;"><span style="font-size: 12px;">kube-apiserver getting OOM (Out Of Memory) killed because it is trying to process too many requests in parallel.</span></p>
<p class="p6" style="text-align: left;"><span style="font-size: 12px;">Generally speaking, 15 parallel requests per 25~30 Pods is sufficient.</span></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 87px;" valign="top">
<p class="p2" style="text-align: left;"><span style="font-size: 12px;">Control memory consumption</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 163px;" valign="top">
<p class="p3" style="text-align: left;"><span style="font-size: 12px;"><b>--target-ram-mb</b></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 244px;" valign="top">
<p class="p2"><span style="font-size: 12px;">This value is used for kube-apiserver to guess the size of the cluster and to configure the deserialize cache size and watch cache <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/apiserver-watch.md">[9]</a> sizes inside the API server.</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 314px;" valign="top">
<p class="p2"><span style="font-size: 12px;">The kube-apiserver uses the same assumption as the above mentioned Kubernetes benchmark: 120GB for ~ 60,000 Pods, 2000 nodes, which is equivalent to 60MB / Node, and 30 Pods on each node.</span></p>
<p class="p6"><span style="font-size: 12px;">Generally speaking, 60MB per 20~30 Pods is a good assumption to make.</span></p>
<p class="p6"><span style="font-size: 12px;"><b>Container memory request can be set to equal to or greater than this value.</b></span></p>
</td>
</tr>
</tbody>
</table>
<p><strong>kube-controller-manager </strong><a href="https://kubernetes.io/docs/admin/kube-controller-manager/">[7]</a> <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-controller-manager/app/options/options.go">[8]</a></p>
<table class="t1" style="width: 812px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 97px;" valign="top">
<p class="p1"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 282px;" valign="top">
<p class="p1"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 289px;" valign="top">
<p class="p1"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 248px;" valign="top">
<p class="p1"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 97px;" valign="top">
<p class="p1"><span style="font-size: 12px;">Control level of parallelism</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 282px;" valign="top">
<p class="p1" style="text-align: left;"><span style="font-size: 12px;"><strong> --concurrent-deployment-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-endpoint-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-gc-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-namespace-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-replicaset-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-resource-quota-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-service-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-serviceaccount-token-syncs<br />
 </strong></span><span style="font-size: 12px;"><strong> --concurrent-rc-syncs</strong></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 289px;" valign="top">
<p class="p1"><span style="font-size: 12px;">Kube-controller-manager has a set of flags that can provide fine-grained controls of parallelism. Increasing parallelism means Kubernetes will be more agile when updating specs, but also allows the controller manager to consume more CPU and memory.</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 248px;" valign="top">
<p class="p1"><span style="font-size: 12px;">In general, increase settings for components you use more intensively. For larger clusters, feel free to increase default values as long as you are OK with its memory usage. For smaller clusters, if you are tight on memory, you can lower the settings. Kubernetes default values can be found in the kube-controller-manager documentation.</span></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 97px;" valign="top">
<p class="p1"><span style="font-size: 12px;">Control memory consumption</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 282px;" valign="top"><span style="font-size: 12px;"> <b>--replication-controller-lookup-cache-size<br />
 </b></span><b style="font-size: 12px; font-family: inherit;"> --replicaset-lookup-cache-size<br />
 </b><span style="font-size: 12px;"><b> --daemonset-lookup-cache-size</b></span>
<p class="p7"> </p>
<p class="p7"> </p>
<p class="p7"> </p>
</td>
<td class="td5" style="background-color: #ffffff; width: 289px;" valign="top"><span style="font-size: 12px;"> This set of flags is not documented but still available for use. Increasing look up cache size can increase sync speed of corresponding controllers, but will increase memory consumption for controller manager. In practice, we set memory request for controller manager container to be greater than the sum of these 3 values. Note that after Kubernetes 1.6, replication controller, replica set and daemonset no longer require a lookup cache so these flags are not needed.</span></td>
<td class="td6" style="background-color: #ffffff; width: 248px;" valign="top"><span style="font-size: 12px;"> The default values (4GB for Replication Controller, 4GB for Replica Set and 1GB for Daemon Set) works fine for even large workloads. You can tune these values down for smaller clusters to save memory.</span>
<p class="p7"><span style="font-size: 12px;"><b>Set the container's memory request to slightly greater than the sum of these values.</b></span></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 97px;" valign="top">
<p class="p1"><span style="font-size: 12px;">Throttle API query rate</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 282px;" valign="top"><span style="font-size: 12px;"> <b>--kube-api-burst<br />
 </b><b> --kube-api-qps</b></span></td>
<td class="td5" style="background-color: #ffffff; width: 289px;" valign="top"><span style="font-size: 12px;"> These 2 flags set normal and burst rate that controller manager can talk to kube-apiserver. We increase these values with larger Applatix cluster configs.</span></td>
<td class="td6" style="background-color: #ffffff; width: 248px;" valign="top"><span style="font-size: 12px;"> Default values work pretty well (20 for QPS and 30 for burst). Increase these values for large, production clusters. </span></td>
</tr>
</tbody>
</table>
<p class="p1"><strong>kube-scheduler</strong> <a href="https://kubernetes.io/docs/admin/kube-scheduler/">[10]</a></p>
<table class="t1" style="width: 808px; height: 209px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 98px;" valign="top">
<p class="p2"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 164px;" valign="top">
<p class="p2"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 377px;" valign="top">
<p class="p2"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 245px;" valign="top">
<p class="p2"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 98px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Throttle API query rate</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 164px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b> --kube-api-burst<br />
 </b></span><span style="font-size: 12px;"><b> --kube-api-qps</b></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 377px;" valign="top">
<p class="p3"><span style="font-size: 12px;">These 2 flags sets normal and burst rate that kube-scheduler can talk to kube-apiserver, as kube-scheduler polls "need-to-schedule" Pods from api-server, and writes scheduling decisions. <a href="https://coreos.com/blog/improving-kubernetes-scheduler-performance.html">[11]</a> <a href="https://docs.google.com/presentation/d/1HYGDFTWyKjJveAk_t10L6uxoZOWTiRVLLCZj5Zxw5ok/edit#slide=id.gd6d8abb5d_0_2805">[12]</a> kube-scheduler's memory consumption can increase noticeably when there is a burst of pod creation happening inside the cluster</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 245px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Set QPS and Burst to roughly 20% and 30% of max inflight requests for API server is a good assumption to make.</span></p>
</td>
</tr>
</tbody>
</table>
<p class="p1"><strong>kube-proxy </strong> <a href="https://kubernetes.io/docs/admin/kube-proxy/">[13]</a> </p>
<table class="t1" style="width: 803px; height: 162px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 102px;" valign="top">
<p class="p2"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 151px;" valign="top">
<p class="p2"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 366px;" valign="top">
<p class="p2"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 244px;" valign="top">
<p class="p2"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 102px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Throttle API query rate</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 151px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b>--kube-api-burst<br />
 </b></span><span style="font-size: 12px;"><b>--kube-api-qps</b></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 366px;" valign="top">
<p class="p3"><span style="font-size: 12px;">These 2 flags sets normal and burst rate that kube-scheduler can talk to kube-apiserver. kube-proxy mainly use kube-apiserver to watch for changes in Service and Endpoint objects.</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 244px;" valign="top">
<p class="p3"><span style="font-size: 12px;">The default values are fine.</span></p>
</td>
</tr>
</tbody>
</table>
<p>  <strong>etcd</strong></p>
<table class="t1" style="width: 798px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 109px;" valign="top">
<p class="p2"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 141px;" valign="top">
<p class="p2"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 355px;" valign="top">
<p class="p2"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 259px;" valign="top">
<p class="p2"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 109px;" valign="top">
<p class="p1"><span style="font-size: 12px;">Control memory consumption</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 141px;" valign="top">
<p class="p1"><span style="font-size: 12px;"><strong> --snapshot-count</strong></span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 355px;" valign="top">
<p><span style="font-size: 12px;"> Etcd memory consumption and disk usage are directly affected by `--snapshot-count`, and it is likely to have memory burst when there are a lot of Pod churns cluster-wide. See more information about etcd tuning <a href="https://coreos.com/etcd/docs/latest/tuning.html">here</a>.</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 259px;" valign="top">
<p class="p1"><span style="font-size: 12px;">We reduced the default values significantly but keep it enough for our usage. The default value is unnecessarily large for our use case, and can easily cause etcd to be OOM killed. We referred to <a href="https://coreos.com/etcd/docs/latest/op-guide/hardware.html">etcd's documentation</a> for hardware <span class="s1">provisioning</span></span></p>
</td>
</tr>
</tbody>
</table>
<p class="p1"><strong>kubelet </strong><a href="https://kubernetes.io/docs/admin/kubelet/">[14]</a></p>
<table class="t1" style="width: 795px;" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="td1" style="background-color: #ebebeb; width: 107px;" valign="top">
<p class="p2"><b>Function</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 136px;" valign="top">
<p class="p2"><b>Flags</b></p>
</td>
<td class="td2" style="background-color: #ebebeb; width: 329px;" valign="top">
<p class="p2"><b>Description</b></p>
</td>
<td class="td3" style="background-color: #ebebeb; width: 259px;" valign="top">
<p class="p2"><b>Recommendation</b></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 107px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Throttle API query rate</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 136px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b> --kube-api-burst<br />
 </b></span><span style="font-size: 12px;"><b> --kube-api-qps</b></span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 329px;" valign="top">
<p class="p3"><span style="font-size: 12px;">These 2 flags sets normal and burst rate that kubelet can talk to kube-apiserver. As each node will have a limited number of pods, default values work pretty good for us in our stress tests</span></p>
</td>
<td class="td7" style="background-color: #ffffff; width: 259px;" rowspan="2" valign="top">
<p class="p3"><span style="font-size: 12px;"> </span></p>
<p class="p3"> </p>
<p class="p3"><span style="font-size: 12px;">Based on information provided in Kubernetes' benchmark, they use 30 Pods per node, and we assume the default values are good enough for this. Scale these values based on the estimated number of Pods that you want to run for each minion. Use the default values as a minimum guideline.</span></p>
<span style="font-size: 14px;"> </span></td>
</tr>
<tr>
<td class="td9" style="background-color: #ffffff; width: 107px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Control event generation rate</span></p>
</td>
<td class="td10" style="background-color: #ffffff; width: 136px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b> --event-burst<br />
 </b></span><span style="font-size: 12px;"><b> --event-qps</b></span></p>
</td>
<td class="td10" style="background-color: #ffffff; width: 329px;" valign="top">
<p class="p3"><span style="font-size: 12px;">These 2 flags controls rate kubelet creates events. More events means more workload for master node to process, and you need to tune this value if your have some micro service analyzing (especially caching) Kubernetes event streams, as these services can possibly get OOMed when kubelets are generating too many events globally</span></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 107px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Throttle container registry query rate</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 136px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b> --registry-burst<br />
 </b></span><span style="font-size: 12px;"><b> --registry-qps</b></span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 329px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Rate control when talking to container registry. In our daily development</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 259px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Default values are fine. Increase the value if your application is sensitive to the delay of pulling container images.</span></p>
</td>
</tr>
<tr>
<td class="td4" style="background-color: #ffffff; width: 107px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Protect host</span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 136px;" valign="top">
<p class="p4"><span style="font-size: 12px;"><b> --max-open-files<br />
 </b></span><span style="font-size: 12px;"><b> --max-pods<br />
 </b></span><span style="font-size: 12px;"><b> --kube-reserved<br />
 </b></span><span style="font-size: 12px;"><b> --system-reserved</b></span></p>
</td>
<td class="td6" style="background-color: #ffffff; width: 329px;" valign="top">
<p class="p3"><span style="font-size: 12px;">These 4 flags helps kubelet protect the host.</span></p>
</td>
<td class="td5" style="background-color: #ffffff; width: 259px;" valign="top">
<p class="p3"><span style="font-size: 12px;">Set `--max-pods` to prevent too many small containers from overloading the kubelet for a node. Generally, limit it to 80 or so Pods per node (roughly about 300 containers) per m3.2xlarge node.</span></p>
<p class="p6"><span style="font-size: 12px;">The `--kube-reserved` `--system-reserved` are also useful flags to reserve resources for system and kubernetes components (such as kernel, docker, kubelet, and other Kubernetes Daemon Sets). kube-scheduler will take this into consideration when scheduling pods</span></p>
</td>
</tr>
</tbody>
</table>
<p class="p1"><br />
In addition to configuring component resources, master node root device size needs to be tuned based on the cluster size to ensure log rotation happens frequently enough to satisfy the verbosity level we set for the Kubernetes master components. If log rotation is not active enough, master node could become unstable when root device gets full, making the whole cluster unstable.</p>
<p class="p1">To recap, turning these knobs properly based on your production workload is a critical step towards creating a production-ready cluster. Even so, it may still be a painful experience trying to identify the root causes and come up with remedies for the instabilities and anomalies that you will likely observe in your cluster each time you scale up your load to new levels. In the end, a stable and performant Kubernetes cluster can play a critical role in helping you migrate your apps to a scalable container-based infrastructure, particularly in the public cloud. Welcome to the new era of cloud computing!</p>
<p class="p1">In <a href="https://applatix.staging.wpengine.com/making-kubernetes-production-ready-part-3/">Part 3</a>, we will look into how we architected Applatix software and our micro-services to work with Kubernetes to ensure cluster stability and availability. Stay tuned!</p>
<p class="p2"><span style="font-size: 12px;">Ref [1]: <a href="http://blog.kubernetes.io/2016/07/kubernetes-updates-to-performance-and-scalability-in-1.3.html">http://blog.kubernetes.io/2016/07/kubernetes-updates-to-performance-and-scalability-in-1.3.html</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [2]: <a href="http://blog.kubernetes.io/2016/03/1000-nodes-and-beyond-updates-to-Kubernetes-performance-and-scalability-in-12.html">http://blog.kubernetes.io/2016/03/1000-nodes-and-beyond-updates-to-Kubernetes-performance-and-scalability-in-12.html</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [3]: <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/server.go#L505">https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/server.go#L505</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [4]: <a href="https://kubernetes.io/docs/admin/kube-apiserver/">https://kubernetes.io/docs/admin/kube-apiserver/</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [5]: <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/options/options.go">https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-apiserver/app/options/options.go</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [6]: <a href="https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go">https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [7]: <a href="https://kubernetes.io/docs/admin/kube-controller-manager/">https://kubernetes.io/docs/admin/kube-controller-manager/</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [8]: <a href="https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-controller-manager/app/options/options.go">https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-controller-manager/app/options/options.go</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [9]: <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/apiserver-watch.md">https://github.com/kubernetes/community/blob/master/contributors/design-proposals/apiserver-watch.md</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [10]: <a href="https://kubernetes.io/docs/admin/kube-scheduler/">https://kubernetes.io/docs/admin/kube-scheduler/</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [11]: <a href="https://coreos.com/blog/improving-kubernetes-scheduler-performance.html">https://coreos.com/blog/improving-kubernetes-scheduler-performance.html</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [12]: <a href="https://docs.google.com/presentation/d/1HYGDFTWyKjJveAk_t10L6uxoZOWTiRVLLCZj5Zxw5ok/edit#slide=id.gd6d8abb5d_0_2805">https://docs.google.com/presentation/d/1HYGDFTWyKjJveAk_t10L6uxoZOWTiRVLLCZj5Zxw5ok/edit#slide=id.gd6d8abb5d_0_2805</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [13]: <a href="https://kubernetes.io/docs/admin/kube-proxy/">https://kubernetes.io/docs/admin/kube-proxy/</a></span></p>
<p class="p2"><span style="font-size: 12px;">Ref [14]: <a href="https://kubernetes.io/docs/admin/kubelet/">https://kubernetes.io/docs/admin/kubelet/</a></span></p>
<p class="p3"> </p>
<p class="p1"><br />
 <em>Harry Zhang is a Member of Technical Staff at Applatix. He works with platform team focusing on building containerized micro-services with Kubernetes and AWS. Reach him on <a href="https://www.linkedin.com/in/hao-h-zhang-74b39a41/">LinkedIn</a> for questions, comments, or information about Applatix. </em><br />
 </p>
</div>