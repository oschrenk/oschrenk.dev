---
layout: post
title:  "Connecting local services to AWS MSK"
date:   2019-03-02 14:01:52 +0100
categories: aws msk ssh
---
Connecting local services to AWS MSK

{% highlight shell %}
#!/bin/bash

# set host
export BASTION_HOST=ec2-user@255.255.255.255
export BASTION_SSH=~/.ssh/id_rsa_bastion

# filter msk by name of project
export NAME_FILTER=project

# set cluster arn
export KAFKA_CLUSTER=$(aws kafka list-clusters --cluster-name-filter $NAME_FILTER | jq -r .ClusterInfoList[].ClusterArn | head)

export KAFKA_BROKER_STRING=$(aws kafka get-bootstrap-brokers --cluster-arn $KAFKA_CLUSTER | jq -r .BootstrapBrokerString)

# create localhost aliases
echo ""
echo "Asking for password for each host"
echo $KAFKA_BROKER_STRING | tr ',' '\n' | cut -d ':' -f 1 | sort | xargs -I {} sudo ifconfig lo0 alias add {}

echo ""
echo "lo0 aliases"
ifconfig lo0 inet| grep inet | grep -v 127.0.0.1 | awk '{$1=$1};1' | cut -d ' ' -f 2

# create ssh tunnels
echo $KAFKA_BROKER_STRING | tr ',' '\\n' | cut -d : -f 1 | sort | xargs -I {} ssh -i $BASTION_SSH -fN -L {}:9092:{}:9092 $BASTION_HOST

# list tunnels
echo ""
echo "tunnels"
ps -ef | grep "ssh -i" | grep -v grep
{% endhighlight %}

{% highlight shell %}
#!/bin/bash

# filter msk by name of project
export NAME_FILTER=project

# Kafka installation
export KAFKA_BIN=~/Downloads/kafka_2.12-2.1.0

# topic to consume
export KAFKA_TOPIC=topic-name


# set cluster arn
export KAFKA_CLUSTER=$(aws kafka list-clusters --cluster-name-filter $NAME_FILTER | jq -r .ClusterInfoList[].ClusterArn | head)

export KAFKA_BROKER_STRING=$(aws kafka get-bootstrap-brokers --cluster-arn $KAFKA_CLUSTER | jq -r .BootstrapBrokerString)

$KAFKA_BIN/bin/kafka-console-consumer.sh --bootstrap-server (echo $KAFKA_BROKER_STRING | tr ',' '\\n' | head) --topic $KAFKA_TOPIC
{% endhighlight %}

