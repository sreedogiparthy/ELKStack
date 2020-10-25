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

<p><span class="note"><strong>Note:</strong> Elasticsearch’s configuration file is in YAML format, which means that we need to maintain the indentation format. Be sure that you do not add any extra spaces as you edit this file.<br></span></p>

<p>The <code>elasticsearch.yml</code> file provides configuration options for your cluster, node, paths, memory, network, discovery, and gateway. Most of these options are preconfigured in the file but you can change them according to your needs. For the purposes of our demonstration of a single-server configuration, we will only adjust the settings for the network host. </p>

<p>Elasticsearch listens for traffic from everywhere on port <code>9200</code>. You will want to restrict outside access to your Elasticsearch instance to prevent outsiders from reading your data or shutting down your Elasticsearch cluster through its <a href="https://en.wikipedia.org/wiki/Representational_state_transfer">REST API</a>. To restrict access and therefore increase security, find the line that specifies <code>network.host</code>, uncomment it, and replace its value with <code>localhost</code> like this:</p>
<div class="code-label " title="/etc/elasticsearch/elasticsearch.yml">/etc/elasticsearch/elasticsearch.yml</div><pre class="code-pre "><code>. . .
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: <span class="highlight">localhost</span>
. . .
</code></pre>
<p>We have specified <code>localhost</code> so that Elasticsearch listens on all interfaces and bound IPs. If you want it to listen only on a specific interface, you can specify its IP in place of <code>localhost</code>. Save and close <code>elasticsearch.yml</code>. If you’re using <code>nano</code>, you can do so by pressing <code>CTRL+X</code>, followed by <code>Y</code> and then <code>ENTER</code> . </p>

<p>These are the minimum settings you can start with in order to use Elasticsearch. Now you can start Elasticsearch for the first time.</p>

<p>Start the Elasticsearch service with <code>systemctl</code>. Give Elasticsearch a few moments to start up. Otherwise, you may get errors about not being able to connect.</p>
<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo systemctl start elasticsearch
</li></ul></code></pre>
<p>Next, run the following command to enable Elasticsearch to start up every time your server boots:</p>
<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo systemctl enable elasticsearch
</li></ul></code></pre>
<p>You can test whether your Elasticsearch service is running by sending an HTTP request:</p>
<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">curl -X GET "localhost:9200"
</li></ul></code></pre>
<p>You will see a response showing some basic information about your local node, similar to this:</p>
<pre class="code-pre "><code><div class="secondary-code-label " title="Output">Output</div>{
  "name" : "Elasticsearch",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "qqhFHPigQ9e2lk-a7AvLNQ",
  "version" : {
    "number" : "7.7.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

</code></pre>
<p>Now that Elasticsearch is up and running, let’s install Kibana, the next component of the Elastic Stack.</p>

<a name="step-2-—-installing-and-configuring-the-kibana-dashboard" data-unique="step-2-—-installing-and-configuring-the-kibana-dashboard"></a><a name="step-2-—-installing-and-configuring-the-kibana-dashboard" data-unique="step-2-—-installing-and-configuring-the-kibana-dashboard"></a><h2 id="step-2-—-installing-and-configuring-the-kibana-dashboard">Step 2 — Installing and Configuring the Kibana Dashboard</h2>

<p>According to the <a href="https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html">official documentation</a>, you should install Kibana only after installing Elasticsearch. Installing in this order ensures that the components each product depends on are correctly in place.</p>

<p>Because you’ve already added the Elastic package source in the previous step, you can just install the remaining components of the Elastic Stack using <code>apt</code>:</p>
<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo apt install kibana
</li></ul></code></pre>
<p>Then enable and start the Kibana service:</p>
<pre class="code-pre command prefixed"><code><ul class="prefixed"><li class="line" data-prefix="$">sudo systemctl enable kibana
</li><li class="line" data-prefix="$">sudo systemctl start kibana
</li></ul></code></pre>




