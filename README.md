### Setup Instructions
#### 1. Install Virtualbox 
### Port forwarding rules
	Elasticsearch	TCP	127.0.0.1	9200	-	9200
	Kibana		TCP	127.0.0.1	5601	-	5601
	SSH		TCP	127.0.0.1	22	-	22

#### 2. Download Ubuntu server ISO
***
In VirtualBox, select Machine / New and set up a Linux, Ubuntu 64-bit system. Set however much RAM you can spare on your system (I use 8GB on my 16GB system – at least 2GB to be safe.) Create a virtual drive wherever you can spare the space; you’ll need at least 20GB for this course. Start it, and select the ISO file you downloaded for Ubuntu to start the Ubuntu installation process. Go with the defaults unless you have reason to do otherwise, and enter whatever username and password you want for your root account. When prompted to write changes to disk, select “yes”. You can change selections using the TAB key, and confirm selections with ENTER.

In VirtualBox, bring up the settings for your Ubuntu virtual machine, and select Network, then advanced, and hit the port forwarding button.  Add an entry for “Elasticsearch” for TCP on host IP 127.0.0.1, and enter 9200 for the host port and guest port. Leave the Guest IP blank. Also add a “Kibana” entry in the same way, with port 5601, and an “SSH” entry, with port 22. On MacOS, you may have a conflict with port 22 – so try setting the host port for SSH to 2222 instead, and connect to your virtual server using port 2222 instead of 22 as shown in the course videos.
VirtualBox Troubleshooting

If you are running the Avast anti-virus program, it will conflict with VirtualBox. There is a registry hack that gets around the problem, but you might consider switching to Microsoft’s free Windows Defender instead while using this course.

Don’t forget to check your BIOS settings if you’re having trouble. Virtualization needs to be enabled, and I’ve seen reports of “Hyper-V” virtualization causing problems if it’s on.
Installing Elasticsearch
***
#### 3.Start your Ubuntu virtual machine and log in.

#### 4.First we need a Java environment:
```bash
sudo apt-get install openjdk-8-jre-headless -y
sudo apt-get install openjdk-8-jdk-headless -y
```
#### 5.Now we can install Elasticsearch itself:

#### For the Elasticsearch 5:
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo
tee -a /etc/apt/sources.list.d/elastic-5.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```
#### For the Elasticsearch 6:
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo
tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```
#### 6. Edit the elasticsearch.yml file:
```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```
#### 7. Change network.host to 0.0.0.0 
(in vi, use the arrow keys to where you want to edit, then hit “i” to enter “insert mode” and make your edits. When done, hit ESC to exit “insert mode”, then type :wq to write your changes and quit vi.)
```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
sudo /bin/systemctl start elasticsearch.service
```
#### Elasticsearch is now up and running!

#### 8. Hit 127.0.0.1:9200 from a browser to test it.
***
Troubleshooting

If Elasticsearch doesn’t seem to be running, give it a few minutes and try again. But you may be encountering an issue specific to Ubuntu 16.04 that has a workaround described at https://lxadm.com/Problems_starting_elasticsearch_in_Ubuntu_16.04

Install the Shakespeare Search Index in Elasticsearch 5:
```bash
wget http://media.sundog-soft.com/es/shakes-mapping.json
curl -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
wget http://media.sundog-soft.com/es/shakespeare.json
curl -XPOST '127.0.0.1:9200/shakespeare/_bulk' --data-binary @shakespeare.json   
curl -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
"query" : {
"match_phrase" : {
"text_entry" : "to be or not to be"
								}
					}
}'
```
#### Install the Shakespeare Search Index in Elasticsearch 6:
```bash
wget http://media.sundog-soft.com/es6/shakes-mapping.json
curl -H 'Content-Type: application/json' -XPUT 127.0.0.1:9200/shakespeare --data-binary @shakes-mapping.json
wget http://media.sundog-soft.com/es6/shakespeare_6.0.json
curl -H 'Content-Type: application/json' -X POST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary  @shakespeare_6.0.json   
curl -H 'Content-Type: application/json' -XGET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
"query" : {
"match_phrase" : {
"text_entry" : "to be or not to be"
								}
					}
}'
```
***
#### Install openssh server
`sudo apt-get install openssh-server`
