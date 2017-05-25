# confluent-kafka-vagrant-puppet

Installs a 3 node Vagrant cluster using Centos 7
Deploys a Confluent Kafka cluster supported by multi-node Zookeeper
Creates a test topic with replication = 3

# Prerequisites

## On OSX:
```
brew cask install virtualbox
brew cask install vagrant
brew install ansible
vagrant plugin install vagrant-hostsupdater
vagrant plugin install vagrant-vbguest
```

## Update base box to reduce provisioning time
```
vagrant init centos/7
vagrant ssh
sudo yum update
ctrl +d
vagrant package --output centos7_updated
vagrant box add centos7_updated.box 
```

## Usage

Takes between 5-7 mintues to complete

```git clone git@github.com:aggress/confluent-kafka-vagrant-ansible``
```cd confluent-kafka-vagrant-ansible```
```vagrant up```
