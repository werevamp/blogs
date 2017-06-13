---
ID: 648
post_title: >
  A case for Docker-in-Docker on
  Kubernetes (Part 2)
author: Abhinav Das
post_date: 2017-03-21 17:51:13
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/case-docker-docker-kubernetes-part-2/
published: true
---
<p>In the <a href="https://applatix.staging.wpengine.com/case-docker-docker-kubernetes-part/">previous post</a>, we examined some of the issues of using the host Docker daemon socket, Docker-outside-of-Docker (DooD), inside containers running on a CI/CD system. In this post, we will talk about how running the Docker daemon inside a Docker container, Docker-inside-Docker (DinD), works better with <a class="external-link" href="https://kubernetes.io/docs/user-guide/pods/" rel="nofollow">Kubernetes Pods</a> as compared to DooD (If these terms are unfamiliar please check out the <a href="https://applatix.staging.wpengine.com/case-docker-docker-kubernetes-part/">previous post</a>). Our infrastructure runs on top of Kubernetes, and we also allow our users to create workflows that are composed of multiple steps. Each step in the workflow is a Kubernetes Pod.</p>
<h2>Kubernetes Pods</h2>
<p>From the Kubernetes website, a Pod is described in the following words:</p>
<blockquote>
<p>A <em>pod</em> (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers), the shared storage for those containers, and options about how to run the containers. Pods are always co-located and co-scheduled, and run in a shared context. A pod models an application-specific “logical host” - it contains one or more application containers which are relatively tightly coupled — in a pre-container world, they would have executed on the same physical or virtual machine.</p>
</blockquote>
<p>Kubernetes Pods have some very useful properties:</p>
<ul>
	<li>Each Kubernetes Pod gets a Pod IP that is reachable from all the nodes that form the Kubernetes cluster.</li>
	<li>Containers running in Kubernetes Pods share the same network namespace, i.e. Two containers say apache and mysql running on the same Pod, can talk to each other using "localhost". Also, both apache and mysql can be reached using the pod IP.</li>
	<li>Kubernetes garbage collects pods after they are terminated, preventing nodes from running out of space.</li>
</ul>
<p>In the context of a Pod, if the user wants to have access to Docker, we have a choice between DooD and DinD. Let's see how DooD and DinD approaches differ when used in the context of a Pod.</p>
<h2>Case 1: Pods and DooD</h2>
<p>When Docker commands are sent to the host Docker daemon, Kubernetes does not know anything about this newly created container and rightfully does nothing to manage it. If a named container is created using Docker commands, container creation might fail if the named container already exists. Since the new container does not know anything about Pod networking, any open ports will not be reachable using the Pod's IP. As described in the previous post, the port mapping specification on the Docker command (e.g. <span style="font-family: monospace;">docker run -v 8080:80 httpd:latest</span>) will open ports on the host, a limited and contended resource. When the container terminates, the layers of graph storage will not be deleted by Kubernetes and logs will not be cleaned up by Kubernetes. In the diagram below, Pod A is a multi-container pod running Apache <span style="font-family: monospace;">(httpd)</span> and MySQL containers. Apache is listening on port 80 and MySQL is listening on port 3306. Both these ports can be accessed on the Pod's IP <span style="font-family: monospace;">(192.168.2.100:80 and 192.168.2.100:3306)</span>. Pod B, on the other hand, uses the <span style="font-family: monospace;">docker run</span> command to start Apache. With DooD, the Apache container will run outside the Pod network and will not be available on Pod IP.</p>
<p><img class="wp-image-652 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/03/dood-1-1.png" alt="" width="538" height="471" /></p>
<p><span style="font-size: 14px;"><em>Figure 1: Docker outside of Docker on Kubernetes Pods</em></span></p>
<p>Below is a kubernetes specification for a Pod that uses DooD to create a container. You can run this on your Kubernetes cluster and see how this works. Copy the following yaml to a file, for example, say <span style="font-family: monospace;">dood.yaml</span> and type <code>kubectl create -f dood.yaml</code> on your shell.</p>
<p>&nbsp;</p>
<div class="code panel pdl conf-macro output-block">
<div class="codeContent panelContent pdl">
<div>
<div id="highlighter_263058" class="syntaxhighlighter sh-confluence  xml">
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td class="gutter">
<pre class="line number1 index0 alt2">1 
2 
3 
4 
5 
6 
7 
8 
9 
10 
11 
12 
13 
14 
15 
16 
17 
18 
19 
20</pre>
</td>
<td class="code">
<div title="Hint: double-click to select code">
<pre>apiVersion: v1 
kind: Pod 
metadata: 
    name: dood 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        volumeMounts: 
          - mountPath: /var/run 
            name: docker-sock 
    volumes: 
      - name: docker-sock 
        hostPath: 
            path: /var/run </pre>
</div>
</td>
</tr>
</tbody>
</table>
</div>
</div>
</div>
</div>
<p>The Pod will create a container that will run outside of the Pod. By running the container using DooD, you lose out on the following for the spawned container:</p>
<ul>
	<li><strong>Pod Networking</strong> - Cannot access the container using Pod IP.</li>
	<li><strong>Pod Lifecycle</strong> - On Pod termination, this container will keep running especially if the container was started with <code>-d</code> flag.</li>
	<li><strong>Pod Cleanup</strong> - Graph storage will not be cleanup after pod terminates.</li>
	<li><strong>Scheduling and Resource Utilization</strong> - Cpu and Memory requested by Pod, will only be for the Pod and not the container spawned from the Pod. Also, limits on CPU and memory set for the Pod will not be inherited by the spawned container.</li>
</ul>
<h2>Case 2: Pods and DinD</h2>
<p>Docker-in-Docker works by running a Docker daemon inside a Docker container. <strong>The main requirement for DinD daemon is that it must not share the graph storage of the host's Docker daemon.</strong> Containers created with the DinD daemon are not visible to the host Docker daemon. We use the concept of <a class="external-link" href="https://www.usenix.org/conference/hotcloud16/workshop-program/presentation/burns" rel="nofollow">Sidecar containers</a> and create a Kubernetes Pod that contains a <a class="external-link" href="https://hub.docker.com/_/docker/" rel="nofollow">dind container</a>. This container starts Docker daemon on <span style="font-family: monospace;">/var/lib/docker</span>. We can use <span style="font-family: monospace;"><a class="external-link" href="https://kubernetes.io/docs/user-guide/volumes/#emptydir" rel="nofollow">EmptyDir</a></span> and mount it as <span style="font-family: monospace;">/var/lib/docker</span> inside the dind container. The yaml file below describes such a Pod. The <span style="font-family: monospace;">docker-cmds</span> container issues Docker commands to start the Apache container. The sidecar container, <span style="font-family: monospace;">dind-daemon</span>, starts the Docker REST service on port 2375. Setting the <span style="font-family: monospace;">DOCKER_HOST</span> to <span style="font-family: monospace;">tcp://localhost:2375</span> ensures that the Docker binary in the main container points to this Docker daemon using <span style="font-family: monospace;">DOCKER_HOST</span> environment variable.</p>
<div class="code panel pdl conf-macro output-block">
<div class="codeContent panelContent pdl">
<div>
<div id="highlighter_288803" class="syntaxhighlighter sh-confluence  xml">
<table border="0" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>
<pre>1 
2 
3 
4 
5 
6 
7 
8 
9 
10 
11 
12 
13 
14 
15 
16 
17 
18 
19 
20 
21 
22 
23 
24 
25 
26 
27 
28 
29 
30</pre>
</td>
<td>
<pre>apiVersion: v1 
kind: Pod 
metadata: 
    name: dind 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        env: 
          - name: DOCKER_HOST 
            value: tcp://localhost:2375 
      - name: dind-daemon 
        image: docker:1.12.6-dind 
        resources: 
            requests: 
                cpu: 20m 
                memory: 512Mi 
        securityContext: 
            privileged: true 
        volumeMounts: 
          - name: docker-graph-storage 
            mountPath: /var/lib/docker 
    volumes: 
      - name: docker-graph-storage 
        emptyDir: {}</pre>
</td>
</tr>
</tbody>
</table>
</div>
</div>
</div>
</div>
<p>To check if Pod networking is running you can create a new Pod that has curl and specify the Pod IP on the curl command.</p>
<table style="height: 281px;" width="739">
<tbody>
<tr>
<td style="width: 729px;">
<pre>$&gt; kubectl get pods -o wide  
NAME   READY   STATUS    RESTARTS   AGE  IP               NODE  
dind   2/2     Running   1          6m   192.168.142.19   ip-10-128-6-95  
  
$&gt; kubectl run curl-pod --restart=Never \ 
--image=hiromasaono/curl -- curl 192.168.142.19 
pod "curl-pod" created  
  
$&gt; kubectl logs curl-pod  
&lt;html&gt;&lt;body&gt;&lt;h1&gt;It works!&lt;/h1&gt;&lt;/body&gt;&lt;/html&gt; 
</pre>
</td>
</tr>
</tbody>
</table>
<p>Now this container is running as a child process of the dind daemon process and inherits the CPU and memory constraints. When you delete the Pod, this container is killed and never shows up on the host. The EmptyDir used for DinD graph storage is also reclaimed by Kubernetes when Pod is deleted. Figure 2, shows this above behavior pictorially. When the ubuntu container runs the command "docker run httpd:latest", a new container is created inside the dind container. Name conflicts for containers created using <code>docker run --name "something"</code> are only possible within the pod, so we can safely execute this workflow step in parallel.</p>
<p><img class="wp-image-469 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/03/dind.png" alt="" width="561" height="489" /></p>
<p><span style="font-size: 14px;"><em>Figure 2: Docker inside Docker on Kubernetes Pods</em></span></p>
<h2>Summary</h2>
<p>At Applatix, we manage the Graph Storage for the DinD container so that there is reuse between workflows. This allows workflows to run faster by not having to always fetch layers in the Graph Storage. We also make it easy for users run Docker commands from inside containers, so that the user does not have to worry about storage provisioning, reuse and configuration. A unique feature of our system is that it tracks the dollar cost of running containers and allows our users to understand the cost of running their applications in the public cloud. Docker in Docker keeps the container from "escaping" the Pod and allows us to manage its resource utilization and cost. We find that using DinD allows us to use Docker consistently and reliably in our CI/CD system. If you have a question about this blog, feel free to drop us an email at info@applatix.com or @applatix on twitter.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>