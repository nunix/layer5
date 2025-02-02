---
layout: post
title:  "One CNAB Bundle to Install Them All"
date:   2019-07-09 12:00:00 +0000
image: /assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/cnab-logo.jpg
author: Nuno de Carmo
permalink: /blog/2019-07-09-one-cnab-bundle-to-install-them-all
---
# One CNAB Bundle to Install Them All
by Nuno de Carmo

During KubeCon EU 2019, I had the chance to discover two new softwares that simply amazed me:

1. [Meshery](https://layer5.io/meshery), which is a multi-service mesh management plane.
1. [K3d](https://github.com/rancher/k3d), which is used to create a dockerized [K3s](https://k3s.io) server.

And, what really appealed to me about both of them is that everything from the installation to the usage was just *simple!*

And on cream on the top, both softwares are used with or inside containers, making each ideal for a create/try/delete workflow.

## The base configuration
Before we start having *real* fun with Meshery, I will quickly list the different components I used for this blog post and ensure I define what could be optional for your own setup:

1. [Meshery](https://layer5.io/meshery)
1. [Docker](https://docs.docker.com/install/) 
 - Docker is of course mandatory and as Meshery is based on a Compose file, which means that [docker-compose](https://docs.docker.com/compose/install/) is also mandatory.
1. [K3d](https://github.com/rancher/k3d) 
 - K3d or any K3s/K8s cluster that you might have already configured.
 1. [WSL2](https://devblogs.microsoft.com/commandline/wsl-2-is-now-available-in-windows-insiders/) *(optional)*
 - For the (few) ones who know me, my "OS base" is WSL2, hhich means that without much/any change, it should run fine for any Linux/MacOS setup.
1. [Graphana](https://grafana.com/) *(optional)*
 - Graphana is not mandatory however is strongly recommend. We will setup a dockerized instance, but feel free to plug Meshery with your existing instance.

### Nothing is granted 
For the sake of making the blog post around Meshery, I won't explain how to install each component and will focus only on getting K3d and Meshery working.

That said, I do not take anything for granted and as Scott Hanselman once taught me: there is no "just have to ..." or "by simply doing ...".
If you face any issue with your setup (hopefully WSL2), just let me know on [Twitter](https://twitter.com/nunixtech) or on the [Meshery Slack channel](http://slack.layer5.io).

## Meshery Installation
For the following steps, I will use the Ubuntu 18.04 WSL2 distro:

1. Start docker and confirm it's running:
<div class="highlight highlight-source-shell">
    <pre><code>
$ sudo service docker start
$ docker version
    </code></pre>
</div> 
1. Using Docker, install Meshery on your local machine by running the following:

<div class="highlight highlight-source-shell">
    <pre><code>
 $ sudo curl -L https://git.io/meshery -o /usr/local/bin/meshery
 $ sudo chmod a+x /usr/local/bin/meshery
 $ meshery start
    </code></pre>
</div>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-docker-start.png"><img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-docker-start.png" class="center-50" /></a>

<p>Create a new K3d cluster with the <code> WSL2 IP </code></p>

<div class="highlight highlight-source-shell">
    <pre><code>
$ export mainIP=`hostname -I | awk '{ print $1 }'`
$ k3d list
$ k3d create --workers 3 --api-port ${mainIP}:6443
$ export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
$ kubectl cluster-info
    </code></pre>
</div>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-k3d-start.png"><img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-k3d-start.png" class="center-50" /></a>

3. Start Meshery on the newly created cluster
<div class="highlight highlight-source-shell">
    <pre><code>
$ meshery start
    </code></pre>
</div>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-start.png"><img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-start.png" class="center-50" /></a>

1. Once Meshery is fully started, login in your preferred browser using the <code>WSL2 IP</code> instead of `<code>ocalhost</code>
<div class="highlight highlight-source-shell">
    <pre><code>
$ export BROWSER=/mnt/c/Firefox/firefox.exe
$ $BROWSER $mainIP:9081 &
    </code></pre>
</div>
<p align="center">
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-meshery-login.png"><img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-meshery-login.png" class="center-50"  /></a>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-login-success.png"><img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-login-success.png" class="center-50" /></a>
</p>

### [Optional] More analytics with Graphana
As stated above, Meshery can leverage the analytics provided by Graphana.

For this blog post, as everything is built from scratch, here is the setup for a new Graphana dockerized instance:

- Start a new Grafana on docker instance
<div class="highlight highlight-source-shell">
    <pre><code>
$ docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_SERVER_ROOT_URL=http://$mainIP" \
  -e "GF_SECURITY_ADMIN_PASSWORD=MesheryInstance" \
  grafana/grafana
    </code></pre>
</div>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-grafana-start.png">
    <img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all/wsl-grafana-start.png" class="center-50">
</a>

- Access the new instance with the admin password that you set in the docker environment variable
<div class="highlight highlight-source-shell">
    <pre><code>
$ $BROWSER $mainIP:3000 &
    </code></pre>
</div>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-grafana-login.png">
    <img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-grafana-login.png" class="center-50" />
</a>
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-grafana-login-success.png">
    <img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-grafana-login-success.png" class="center-50" />
</a>

### An inside look
While everything should run fine, it's always good to have a look at what has been deployed.

In this case, we are almost exclusively working with Docker and the "inside look" should look something like this:
<a href="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-complete.png">
    <img src="/assets/images/posts/2019-07-09-one-cnab-bundle-to-install-them-all//wsl-meshery-complete.png" class="center-50" />
</a>

## Conclusion
As [Lee Calcote](https://twitter.com/lcalcote) put it: this is a lot of buzz words: 
Meshery > K3s (deployed via K3d) > Docker > WSL2 > Windows 10

And he's totally right, still the "beauty" here, is that it "simply works".

Since the begin of the Docker era, new tooling has appeard for simplifying complex workflows.
Even Kubernetes (K8s) as a much lighter version with K3s by [Rancher](https://rancher.com).

And of course, Meshery which integrates and simplifies the installation and benchmarking of different Service Meshes.

Hope you had fun assembling all these pieces and stay tunned for the "Bonuses", more fun to come!

\> > > *Nunix out*

---

### Bonus 1: Meshery configuration

### Bonus 2: Adding a Service Mesh to the mix

### Bonus 3: A CNAB Bundle to install them all
