# confluent-kafka-vagrant-ansible

* Installs a 3 node Vagrant cluster using CentOS 7
* Deploys a Confluent Kafka (Platform) cluster supported by multi-node Zookeeper
* Enables SSL for authentication and broker traffic encryption

# Preamble

Securing a Kafka or Confluent Platform cluster is pretty straightforward, either do it with SSL or SASL.  This example covers
SSL. As a good security practicioner, you won't want to rely solely on this, Kafka also has access control lists (ACLs) for fine
grained control. SSL is used for Client<->Broker and Broker<->Broker authentication, with the SSL certs also enabling TLS encryption
of network traffic.

Here's the flow:

1. Generate an SSL certificate and key for each Broker and one for a consumer and producer client
2. Create a certifcate authority (CA)
3. Sign the server certificate
4. Sign the server certificate with the CA
5. Configure the Brokers to enable SSL and point at the correct assets
6. Test authenticated communication

Each Broker's configured with two unique assets, a truststore holding all the certs it needs to know about, to allow it to say
yes you can communicate with me, as well as vice versa with other components, and a keystore which holds the certificate keys.

The Java tool `keytool` is used to create the stores in the .jks format

The verbose openssl and keytool commands are very handily wrapped up into a shell script which takes a list of hosts to loop
through and generate the required assets.  In this example they'll all live in the secrets directory, which hosts have been
configured to pick up from /vagrant/secrets.

By default, Kafka Brokers listen on TCP:9092/Plaintext. With SSL enabled in this example, all clients need to authenticate
accordingly over the same port. You can also configure plaintext and SSL on different ports to run in parallel.

# Prerequisites

## On OSX
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew cask install virtualbox
brew cask install vagrant
brew install ansible
vagrant plugin install vagrant-hostsupdater
vagrant plugin install vagrant-vbguest
```

## Usage

Clone the repo

```
git clone git@github.com:aggress/confluent-kafka-vagrant-ansible
cd confluent-kafka-vagrant-ansible
```

Generate the SSL certs and stores (requires OpenSSL and keytool)
```
cd secrets
./create-certs.sh
```

Run Vagrant

`vagrant up`

This'll take between 5-10 minutes to complete due to package downloads and installs. This can be greatly reduced
by configuring a custom Vagrant box with those steps already completed, I have it down to just under 3 minutes.

You'll want to let the cluster coordinate itself before creating the test topic otherwise you may find, not all
Brokers are in sync.  Logs can be found in `/var/log/kafka/kafkaServer.out`

## SSL Testing

Create a test topic
```
/usr/bin/kafka-topics --create --zookeeper 192.168.33.10:2181 --replication-factor 3 --partitions 1 --topic test
/usr/bin/kafka-topics --describe --zookeeper localhost:2181 --topic test
```

Test Kafka's talking SSL on TCP:9092

`openssl s_client -debug -connect localhost:9092 -tls1`

Test talking SSL through the console producer on Broker1

`/usr/bin/kafka-console-producer --broker-list localhost:9092 --topic test --producer.config /vagrant/secrets/host.producer.ssl.config`

And, in parallel listen on Broker2

`/usr/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic test --consumer.config /vagrant/secrets/host.consumer.ssl.config`

Write some messages on Broker1 and see them appear on Broker2. You can test again by removing the `--producer.config` and `--consumer.config`
options. This should not now work, as the console clients can't authenticate.


## References

In my opinion, the Apache Kafka docs are clearer than Confluent's

https://kafka.apache.org/documentation/#security

http://docs.confluent.io/current/kafka/ssl.html

## Credits

create-certs.sh is Confluent's and updated to not prompt and output for the hosts in this example.

https://github.com/confluentinc/cp-docker-images/blob/v3.2.1/examples/kafka-cluster-ssl/secrets/create-certs.sh