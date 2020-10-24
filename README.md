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


<p><span class="note"><strong>Note</strong>: When installing the Elastic Stack, you must use the same version across the entire stack. In this tutorial we will install the latest versions of the entire stack which are, at the time of this writing, Elasticsearch 7.7.1, Kibana 7.7.1, Logstash 7.7.1, and Filebeat 7.7.1.<br></span></p>

<h2 id="prerequisites">Prerequisites</h2>

<p>To complete this tutorial, you will need the following:</p>

<ul>
<li><p>An Ubuntu 20.04 server with 4GB RAM and 2 CPUs set up with a non-root sudo user. You can achieve this by following the <a href="https://github.com/dogiparthy85/ubuntu20.04-server">Initial Server Setup with Ubuntu 20.04</a>.For this tutorial, we will work with the minimum amount of CPU and RAM required to run Elasticsearch. Note that the amount of CPU, RAM, and storage that your Elasticsearch server will require depends on the volume of logs that you expect. </p></li>
<li><p>OpenJDK 11 installed. See the section <a href="https://github.com/dogiparthy85/Java-Ubuntu20.04">Installing the Default JRE/JDK</a> <a href="https://github.com/dogiparthy85/Java-Ubuntu20.04">How To Install Java with Apt on Ubuntu 20.04</a> to set this up. </p></li>
<li><p>Nginx installed on your server, which we will configure later in this guide as a reverse proxy for Kibana. Follow our guide on <a href="https://github.com/dogiparthy85/Nginx-on-Ubuntu-20.04">How to Install Nginx on Ubuntu 20.04</a> to set this up.</p></li>
</ul>
