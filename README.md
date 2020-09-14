# Ansible-Molecule for Network Automation

This project aims at showing the capabilties of the [Molecule tool](https://molecule.readthedocs.io/en/latest/index.html) for Network Automation Ansible projects.

## Overview

The Molecule project is designed to aid the development and testing of Ansible playbooks, roles and collections. And for this it provides support for testing with multiple instances, operating systems and distributions, virtualization providers, test frameworks and scenarios.

In the network infrastructure/automation domain testing can be particularly hard, connectivity and data collection mechanism, network device virtualization, difference between NOSes, among other issues. And here is where the flexibility that molecule gives, alongside the capabilities that ansible already provides, helps building test scenarios wher you can test your ansible projects.

To follow along with the example of the demos you need to clone it:

```shell
git clone https://github.com/davidban77/ansible-molecule-demo.git
cd ansible-molecule-demo
```

When molecule is run, it is normally in charge of:

- Creating the instances to test against.
- Preparing them, like installing pre-requisites or setting up the environemnt.
- Converging the playbooks, roles or collections you want to test.
- Performing idempotency check by running again your ansible project.
- Verifying of the instance with ansible playbook tasks that validate your desired state.
- Destroying the instances.

## Requirements

In a python virtual environment in the dependencies stated in `requirement.txt`

```shell
pip install -r requirements.txt
```

## How it works

This repository has 2 playbooks which follow the following workflow:

```shell
Collec Interface Raw Data > Parse data into structured format > Save to DB
```

There are 2 molecule scenarios built that test these playbooks.

## Test: Static Data Scenario

This scenarios aims to show how molecule can be used with docker plugin. It instantiates a `centos:7` docker container where a interface data `txt` file has the information and the playbook is in charge of reading it, parsing the data and save it to an Elasticsearch DB container.

To run all scneario's phases:

```shell
molecule test -s static
```

## Test: Mocked Data Scenario

In this scenario I am leveraging the awesome project [cisgo-ios](https://github.com/tbotnz/cisgo-ios), which give the ability to mock an SSH connection to a network device and has some predefined data which you can use to parse and test.

As a pre-requisite for this test scenario you need to build a docker image locally so it can be used:

```shell
git clone https://github.com/tbotnz/cisgo-ios.git
cd cisgo-ios
docker build -t cisgo-ios:latest .
docker tag cisgo-ios:latest molecule_local/cisgo-ios:latest
```

The tag is needed when molecule uses it.

To run the test scenario:

```shell
molecule test -s mock
```
