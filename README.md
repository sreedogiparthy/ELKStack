# ELKStack
<h1 class="content-title Tutorial-header">How To Install Elasticsearch, Logstash, and Kibana (Elastic Stack) on Ubuntu 20.04</h1>

  <h3 id="introduction">Introduction</h3>
  <p>The Elastic Stack — formerly known as the <em>ELK Stack</em> — is a collection of open-source software produced by <a href="https://www.elastic.co/">Elastic</a> which allows you to search, analyze, and visualize logs generated from any source in any format, a practice known as <em>centralized logging</em>. Centralized logging can be useful when attempting to identify problems with your servers or applications as it allows you to search through all of your logs in a single place. It’s also useful because it allows you to identify issues that span multiple servers by correlating their logs during a specific time frame.</p>
  
  <p>The Elastic Stack has four main components:</p>
  
  <ul>
<li><a href="https://www.elastic.co/products/elasticsearch"><strong>Elasticsearch</strong></a>: a distributed <a href="https://en.wikipedia.org/wiki/Representational_state_transfer"><em>RESTful</em></a> search engine which stores all of the collected data.</li>
<li><a href="https://www.elastic.co/products/logstash"><strong>Logstash</strong></a>: the data processing component of the Elastic Stack which sends incoming data to Elasticsearch.</li>
<li><a href="https://www.elastic.co/products/kibana"><strong>Kibana</strong></a>: a web interface for searching and visualizing logs.</li>
<li><a href="https://www.elastic.co/products/beats"><strong>Beats</strong></a>: lightweight, single-purpose data shippers that can send data from hundreds or thousands of machines to either Logstash or Elasticsearch.</li>
</ul>

<p>In this tutorial, you will install the <a href="https://www.elastic.co/elk-stack">Elastic Stack</a> on an Ubuntu 20.04 server. You will learn how to install all of the components of the Elastic Stack — including <a href="https://www.elastic.co/products/beats/filebeat">Filebeat</a>, a Beat used for forwarding and centralizing logs and files — and configure them to gather and visualize system logs. Additionally, because Kibana is normally only available on the <code>localhost</code>, we will use <a href="https://www.nginx.com/">Nginx</a> to proxy it so it will be accessible over a web browser. We will install all of these components on a single server, which we will refer to as our <em>Elastic Stack server</em>.</p>


<p><span class="note"><strong>Note</strong>: When installing the Elastic Stack, you must use the same version across the entire stack. In this tutorial we will install the latest versions of the entire stack which are, at the time of this writing, Elasticsearch 7.9.3, Kibana 7.9.3, Logstash 7.9.3, and Filebeat 7.9.3.<br></span></p>

<h2 id="prerequisites">Prerequisites</h2>

<p>To complete this tutorial, you will need the following:</p>

<ul>
<li><p>An Ubuntu 20.04 server with 4GB RAM and 2 CPUs set up with a non-root sudo user. You can achieve this by following the <a href="https://github.com/dogiparthy85/ubuntu20.04-server">Initial Server Setup with Ubuntu 20.04</a>.For this tutorial, we will work with the minimum amount of CPU and RAM required to run Elasticsearch. Note that the amount of CPU, RAM, and storage that your Elasticsearch server will require depends on the volume of logs that you expect. </p></li>
<li><p>OpenJDK 11 installed. See the section <a href="https://github.com/dogiparthy85/Java-Ubuntu20.04">Installing the Default JRE/JDK</a> <a href="https://github.com/dogiparthy85/Java-Ubuntu20.04">How To Install Java with Apt on Ubuntu 20.04</a> to set this up. </p></li>
<li><p>Nginx installed on your server, which we will configure later in this guide as a reverse proxy for Kibana. Follow our guide on <a href="https://github.com/dogiparthy85/Nginx-on-Ubuntu-20.04">How to Install Nginx on Ubuntu 20.04</a> to set this up.</p></li>
</ul>

<p>Additionally, because the Elastic Stack is used to access valuable information about your server that you would not want unauthorized users to access, it’s important that you keep your server secure by installing a TLS/SSL certificate. This is optional but <strong>strongly encouraged</strong>.</p>

<p>However, because you will ultimately make changes to your Nginx server block over the course of this guide, it would likely make more sense for you to complete the Let’s Encrypt setup.</p>

<h2 id="step-1-—-installing-and-configuring-elasticsearch">Step 1 — Installing and Configuring Elasticsearch</h2>

<p>The Elasticsearch components are not available in Ubuntu’s default package repositories. They can, however, be installed with APT after adding Elastic’s package source list.</p>

<p>All of the packages are signed with the Elasticsearch signing key in order to protect your system from package spoofing. Packages which have been authenticated using the key will be considered trusted by your package manager. In this step, you will import the Elasticsearch public GPG key and add the Elastic package source list in order to install Elasticsearch.</p>

<p>To begin, use cURL, the command line tool for transferring data with URLs, to import the Elasticsearch public GPG key into APT.  Note that we are using the arguments -fsSL to silence all progress and possible errors (except for a server failure) and to allow cURL to make a request on a new location if redirected.  Pipe the output of the cURL command into the apt-key program, which adds the public GPG key to APT. </p>

<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
</li></ul></code></pre>

<p>Next, add the Elastic source list to the <code>sources.list.d</code> directory, where APT will search for new sources:</p>

<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
</li></ul></code></pre>

<p>Next, update your package lists so APT will read the new Elastic source:</p>

<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo apt update
</li></ul></code></pre>

<p>Then install Elasticsearch with this command:</p>

<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo apt install elasticsearch
</li></ul></code></pre>

<p>Elasticsearch is now installed and ready to be configured. Use your preferred text editor to edit Elasticsearch’s main configuration file, <code>elasticsearch.yml</code>. Here, we’ll use <code>nano</code>:</p>

<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo nano /etc/elasticsearch/elasticsearch.yml
</li></ul></code></pre>




