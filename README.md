ELK (Elasticsearch, Logstash and Kibana)
===

All about my ELK configurations + nxlog as a log forwarder

The Elasticsearch, Logstash, Kibana (ELK) stack has become very popular recently for cheap and easy centralized logging. We will go over the installation of Elasticsearch, Logstash and Kibana, and how to configure them to gather and visualize the CFEngine outputs of my systems in a centralized location.
* Logstash is an open source tool for collecting, parsing, and storing logs for future use. 
* Kibana is a web interface that can be used to search and view the logs that Logstash has indexed. 
* Both of these tools are based on Elasticsearch. Elasticsearch, Logstash, and Kibana, when used together is known as an ELK stack.

It is possible to use Logstash to gather logs of all types, but we will limit the scope of this tutorial to CFEngine related log gathering. For a log forwarder, we are going to use NXlog instead of Logstash Forwarder. NXlog is an open source log management tool available at no cost. Why? you better to follow this page. http://nxlog.org/products/nxlog-community-edition/features

I'm setting ELK up on a fresh install of Ubuntu Server 14.04 LTS. virtual machine with 2GB memory and 2 CPUs.

## Let's get started!

### Install Logstash and Elasticsearch
**Note**: Logstash 1.4.2 recommends Elasticsearch 1.1.1

First, I'm adding the repos of Elasticsearch and Logstash. Run the following command to import Elasticsearch public GPG key into apt:
```
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
```
Edit `/etc/apt/sources.list` and add the Elasticsearch and Logstash repositories to the end of the file:
```
deb http://packages.elasticsearch.org/elasticsearch/1.1/debian stable main  
deb http://packages.elasticsearch.org/logstash/1.4/debian stable main
```
Update your apt package database:
```
sudo apt-get update
```
Install Logstash this command:
```
sudo apt-get -y install logstash
```
*(Logstash and Elasticsearch require Java. It would take some times depending on how fast your internet is.)*

Then install Elasticsearch 1.1.1
```
sudo apt-get -y install elasticsearch=1.1.1
```
Elasticsearch is now installed. Let's edit the configuration:
```
sudo vim /etc/elasticsearch/elasticsearch.yml
```
Add the following line somewhere in the file, to disable dynamic scripts:
```
script.disable_dynamic: true
```
**Optional**: *If you want to restrict outside access to your Elasticsearch instance (port 9200), so outsiders cannot read your data or shutdown your Elasticseach cluster through the HTTP API. Find the line that specifies network.host and uncomment it so it looks like this:*
```
network.host: localhost
```
Save and exit `elasticsearch.yml`. Now start Elasticsearch:
```
sudo service elasticsearch restart
```
Then run the following command to start Elasticsearch on boot up:
```
sudo update-rc.d elasticsearch defaults 95 10
```
Now that Elasticsearch is up and running. A quick test to make sure that Elasticsearch was installed correctly and works. To do that, run `curl http://localhost:9200`. Curl should output something like:
```
{
  "status" : 200,
  "name" : "Rl'nnd",
  "version" : {
    "number" : "1.1.1",
    "build_hash" : "f1585f096d3f3985e73456debdc1a0745f512bbc",
    "build_timestamp" : "2014-04-16T14:27:12Z",
    "build_snapshot" : false,
    "lucene_version" : "4.7"
  },
  "tagline" : "You Know, for Search"
}
```
Let's install Kibana

### Install Kibana (+ Nginx as a web server)

**Note**: Logstash 1.4.2 recommends Kibana 3.1.2

Download Kibana to /tmp directory with the following command:
```
cd /tmp; wget https://download.elasticsearch.org/kibana/kibana/kibana-3.1.2.tar.gz
```
Extract Kiban tarball with tar:
```
tar xvfz kibana-3.1.2.tar.gz
```
We will be using Nginx to serve our Kibana installation, so let's make it more standadisation to where web-related stuff should be located:
```
sudo mkdir -p /var/www/kibana3
```
Now copy the Kibana files into the directory:
```
sudo cp -R /tmp/kibana-3.1.2/* /var/www/kibana3/
```
Last, but definitely not least, before we can use the Kibana web interface, we have to install Nginx. So:
```
sudo apt-get -y install nginx
```
We better should ensure ownership of Kibana to `www-data` as well:
```
sudo chown -R www-data:www-data /var/www
```

## Quick overview of important locations for files
### Elasticsearch:
* Binaries and stuff: `/usr/share/elasticsearch`
* Plugin manager: `/usr/share/elasticsearch/bin/plugin`
* Configuration: `/etc/elasticsearch/elasticsearch.yml`
* Data: `/var/lib/elasticsearch/<cluster-name>`

### Logstash:
* Binaries and stuff: `/opt/logstash`
* Configuration: `/etc/logstash/conf.d`
* Logs: `/var/log/logstash`

### Nginx
* Binaries, configuration and stuff: `/etc/nginx`
* Logs: `/var/log/nginx`
