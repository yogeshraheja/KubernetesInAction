# POC for Logging Infrastructure on Kubernetes

This ELK stack for PO has been created by Thinknyx Technologies LLP to demonstrate how to automate the elastic stack on Kubernetes.

Below are the components used in the STACK:

* Elasticsearch for storing and searching logs.
* Kibana for viewing them.
* Logstash for analysing and breaking down logs.
* Filebeat for pushing all app logs to logstash.

Elasticsearch is installed as a StatefulSet with hostPath volume. In case you need to use this code in the productin kindly consider Container Storage Options for no data loss.
