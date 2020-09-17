# Clustered MQTT Broker Testing Results
 
## Introduction
This repository provides experiment data from the [ECSA 2020](https://ecsa2020.disim.univaq.it/) paper "[A Comparison of MQTT Brokers for Distributed IoT Edge Computing"](http://www.koziolek.de/docs/Koziolek2020-ECSA-preprint.pdf) written by Heiko Koziolek, Sten Grüner and Julius Rückert from ABB Research. A 10-min video presenting the paper is available on [Youtube](https://www.youtube.com/watch?v=hvjQLZfQvso).

## Overview
We investigated the performance and resilience of several clustered MQTT Brokers. There is a subfolder for each analyzed broker, which contains 

 - broker's configuration file(s) used during the experiments, e.g., in XML (config.xml) or plain text files (.conf)
 - corresponding Kubernetes/Helm configuration files (.yaml), which may for example contain MQTT configuration options to be passed to the broker containers
 - MZBench [BDL files](https://satori-com.github.io/mzbench/scenarios/tutorial/) (benchmark definition language) for each experiment. These scripts initialized the load testing tool MZBench using [vmq_mzbench workers](https://github.com/vernemq/vmq_mzbench). They contain for example the connection configuration (e.g. QoS1), the number of publishers and subscribers spawned, the message frequency, and the payload information.
 - raw monitoring data from MZBench (experimentXYZ.csv) recorded during the experiments. These files contain timestamps, metrics names, and values. 
 - raw monitoring data from the tool [Prometheus](https://prometheus.io/), a 78 MB ZIP file containing invididual CSV files for various system-wide metrics (e.g., CPU load of a Docker container)
 - plots from individual experiments (experimentXYZ_cpu_load.pdf)
 - plots aggregated from multiple experiments (workload-vs-cpu-load__all.pdf)

## Experiment Setup
Our original testbed was a [StarlingX](https://www.starlingx.io/) all-in-one duplex bare metal installation running on two identical servers in a redundant, high-available fashion. Each server had a Dual Intel Xeon CPU E5-2640 v3 running at 2.60 GHz with 8x2 physical cores (32 threads), 128 GB of RAM and Gigabit connectivity.

StarlingX  v3.0 is an open-source virtualization platform for edge clusters and runs on top of CentOS 7.6. All tested brokers run in Docker CE orchestrated by K8s. Prometheus monitoring tools measure CPU load among other metrics. For the broker installations we used helm charts (VerneMQ 1.10.2, EMQX 4.0.5) or public tutorials from broker vendors (Enterprise HiveMQ 4.3.2 evaluation). In K8s, the brokers use replication controllers (HiveMQ), stateful sets (VerneMQ, EMQX) and load balancer services (metalLB as Level 2 Load Balancer). 

A dedicated node in the same Ethernet segment as the StarlingX controllers acts as load driver (CentOS 8.1, Intel Xeon CPU E5-2660 v4 @ 2.00 GHz, 16 cores (32 threads) and 8 GB of RAM). We evaluated different load driver applications, including mqtt-stresser, paho-clients, Locust/MQTT, JMeter/MQTT and MZBench. We decided to use MZBench due to a low resource footprint, convenient Web UI allowing to monitor and export metrics, and also a possibility to define load scenarios in a simple Benchmark Definition Language (BDL). 

We utilized custom MZBench MQTT workers provided by VerneMQ.  In addition to metrics from Prometheus and MZBench, we used broker dashboards provided by brokers to validate throughput measurements. 

## What to do with the data
The data can be used for various purposes:
 - checking the validity of the experiments
 - re-processing the raw data to check other metrics (e.g., disk load)
 - re-plotting the data to extract different information
 - defining new experiments for different workloads, but using a similar setting
 - analyzing additional MQTT brokers with the same experiment settings

## How to use the data
To reproduce the experiments you first need to setup the MQTT Brokers, and there are numerous methods to do this:
 - [EMQX](https://github.com/emqx/emqx) can be installed from binaries or by building the source. If you have Kubernetes/Helm you can also use [EMQX helm charts](https://github.com/emqx/emqx-rel/tree/master/deploy/charts/emqx).
 - [HiveMQ](https://www.hivemq.com/docs/hivemq/4.4/user-guide/getting-started.html) provides various installation options.
 - [VerneMQ](https://github.com/vernemq/vernemq) is also available as source, binaries, Docker containers and [Helm charts](https://docs.vernemq.com/guides/vernemq-on-kubernetes).

To use the BDL scripts, you need to setup [MZBench](https://satori-com.github.io/mzbench/), for example as Docker container, from binaries, or building the source. You can import or copy the BDL scripts into your MZBench installation (using its dashboard or command line) to reproduce the experiments. MZBench's [quickstart guide](https://satori-com.github.io/mzbench/) explains you how to do this in detail. Starting the scenarios requires updating the included IP addresses for the brokers to your own installation. We used private IP addresses only valid in our own specific experiment setup.

We used Python scripts to create the plots for the paper, which crossed the Prometheus cpu load metrics with the throughput metrics from MZBench. 

## Related Work
- [VerneMQ: Reaching 5M message connections](https://www.slideshare.net/ConnectedMarketing/reaching-5-million-messaging-connections-our-journey-with-kubernetes-126143229)
- [HiveMQ: 10M MQTT clients](https://www.hivemq.com/benchmark-10-million/)
- [50M concurrent socket connections](https://blog.hotstar.com/building-pubsub-for-50m-concurrent-socketconnections-5506e3c3dabf)
- [ScaleAgent: Benchmark of MQTT servers](https://bit.ly/2WsTw0Z)
- [Message-oriented Middleware for Industrial Production Systems](https://doi.org/10.1109/COASE.2018.8560493)


