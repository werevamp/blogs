---
ID: 450
post_title: >
  A Case for Docker-in-Docker on
  Kubernetes (Part I)
author: Abhinav Das
post_date: 2017-03-14 17:44:54
post_excerpt: ""
layout: post
permalink: >
  https://applatix.staging.wpengine.com/case-docker-docker-kubernetes-part/
published: true
---
<p>If you are reading this blog, you are at least curious about running Docker commands from within a Docker container.</p>
<p>Jérôme Petazzoni’s <a href="https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/">excellent blog article</a> describes the pitfalls of using Docker-in-Docker (DinD) for Continuous Integration (CI) and testing. He advocates bind mounting the host Docker daemon socket inside the Docker container that needs to issue Docker commands. This approach has been called <a href="http://container-solutions.com/running-docker-in-jenkins-in-docker/">Docker-outside-of-Docker</a> (DooD).</p>
<p>I am going to make a case for why using DinD is a better alternative to DooD when running Docker from a Kubernetes Pod and describe how we do it at Applatix.</p>
<p>But first, why would you want to run a Docker container from within a Docker container? One reason is that your Continuous Integration (CI) app (e.g. Jenkins) may be containerized and you want to provide a build/test container for each CI job you want Jenkins to run.</p>
<p>Another reason is that you may want to build a Docker container image from inside your containerized CI job. Docker is a very useful tool for running other tools, so it is quite natural to invoke Docker as just another tool to get things done.</p>
<p>Let’s see what happens when you bind mount and use the host Docker daemon socket from inside a container. Note that this will simply cause all Docker commands issued from the container to be passed to the host Docker daemon.</p>
<p>We first create a container named “run_docker_cmds” that has the host Docker socket mounted in the container.</p>
<div>
<table width="850">
<tbody>
<tr>
<td style="width: 821px;">
<pre><span style="font-size: 12px;">$&gt; docker run -it --name run_docker_cmds -v /var/run/:/var/run docker:latest sh
$&gt; docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS        NAMES
001ec7961f3f   docker:latest  "docker-entrypoint.sh"   12 hours ago    Up 3 seconds                run_docker_cmds</span></pre>
</td>
</tr>
</tbody>
</table>
</div>
<p>Next, we run a Docker command from inside this container that starts a Docker container named “sleep_60”. Running “docker ps” on the host shows that this new container shows up as just another container on the host.</p>
<table style="width: 849px; height: 244px;">
<tbody>
<tr>
<td style="width: 839px;">
<pre><span style="font-size: 12px;"># from inside run_docker_cmds container:</span>
<span style="font-size: 12px;"># docker run --rm --name sleep_60 ubuntu:latest sleep 60</span>
<span style="font-size: 12px;">$&gt; docker ps</span>
<span style="font-size: 12px;">CONTAINER ID    IMAGE          COMMAND                CREATED        STATUS          PORTS      NAMES</span>
<span style="font-size: 12px;">e8348bbb0914    ubuntu:latest  "sleep 60"             12 hours ago   Up 2 seconds               sleep_60</span>
<span style="font-size: 12px;">001ec7961f3f    docker:latest  "docker-entrypoint.sh" 12 hours ago   Up 48 seconds              run_docker_cmds</span></pre>
</td>
</tr>
</tbody>
</table>
<p>That was easy! Note, however, that this container is a “sibling” of the container from which we started this container. If you try to create this container with the same name as the parent, it will fail due to a name conflict.</p>
<p>This highlights a general problem of using the host’s Docker daemon from inside a container: poor isolation resulting In containers interfering with each other.</p>
<table style="width: 845px; height: 126px;">
<tbody>
<tr>
<td style="width: 1791px;">
<pre><span style="font-size: 12px;">$&gt; docker run --rm --name run_docker_cmds ubuntu:latest sleep 60</span>
<span style="font-size: 12px;">docker: Error response from daemon: Conflict. The name "/run_docker_cmds" is already in use by container 
001ec7961f3fd25f6c3e4ad23674993d78ff8d085c94d83b7daf634b17ec0216. You have to remove (or rename) that container 
to be able to reuse that name..</span></pre>
</td>
</tr>
</tbody>
</table>
<p>At Applatix, we use Kubernetes as our platform for orchestrating containers in the public cloud. Our users specify jobs as workflows where each step of the workflow is a Docker container (see Figure 1 below). To make it easy for our users, we allow them to use Docker commands from their containers.Is this really a problem in practice? Can’t we just pick names that don’t conflict? It turns out that if you want a general platform for running jobs, it requires a lot of work to get the authors of these jobs to coordinate and avoid name collisions.</p>
<p><img class="alignnone wp-image-662 size-full" src="http://applatix.staging.wpengine.com/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-2.38.39-PM-2.png" alt="" width="540" height="470" /></p>
<p><span style="font-size: 14px;"><em>Figure1: A workflow running on Applatix</em></span></p>
<p>This allows users to pull, build or run Docker images and to use their existing container-based scripts as steps in the workflow. Each step in the workflow is then converted to a Kubernetes Pod, where a Kubernetes Pod can have one or more containers. Users can even run Docker compose applications as steps in our workflow.</p>
<p>To continue our discussion, let’s say a user wants to run a named Docker container that runs apache from within a step of her workflow.</p>
<table width="850">
<tbody>
<tr>
<td>
<pre><span style="font-size: 12px;">$&gt; docker run --rm --name my_webserver httpd:latest</span></pre>
</td>
</tr>
</tbody>
</table>
<p>If the user is running multiple instances of this workflow and this step happens to overlap in time with the same step from another workflow, then one of the workflows would fail to create this container due to a name conflict.</p>
<p>There are many other problems that can be caused by poor isolation. For example, by allowing users to issue commands directly to the host Docker daemon, they can create Docker containers that are privileged or bind mount host paths, potentially creating havoc on the host.</p>
<p>To summarize, exposing the host’s Docker daemon allows user containers to create “sibling” containers on the host. Since these containers acquire named resources from the host, these resources need to be managed to avoid conflicts.</p>
<p>Any CI/CD system that allows users to create arbitrary Docker containers, will quickly run into manageability, security and stability problems caused by poor isolation.</p>
<p>Each approach has its advantages and disadvantages, but when it comes to running Docker containers from Kubernetes Pods, we think the DinD approach is better. To find out why, stay tuned for the next post.</p>
<p>(UPDATE:  See part 2 of this post <a href="https://applatix.staging.wpengine.com/case-docker-docker-kubernetes-part-2/">here</a> )</p>