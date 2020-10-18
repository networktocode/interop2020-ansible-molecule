# Molecule for Testing Network Automation Ansible Projects

[Molecule](https://molecule.readthedocs.io/en/latest/index.html) is a project designed to aid in the development of Ansible projects. This demo aims to show its testing framework capabilities in a Network Automation context.

## Overview

The Molecule project is designed to aid the development and testing of Ansible playbooks, roles and collections. And for this it provides support for testing with multiple instances, operating systems and distributions, virtualization providers, test frameworks and scenarios.

In the network infrastructure/automation domain testing can be particularly hard, connectivity and data collection mechanism, network device virtualization, difference between NOSes, among other issues. And here is where the flexibility molecule gives, alongside the capabilities that ansible already provides, help building test scenarios for your ansible projects.

To follow along with the example of the demos you need to clone the project:

```shell
git clone https://github.com/davidban77/ansible-molecule-demo.git
cd ansible-molecule-demo
```

When molecule is run, it is normally in charge of:

- Creating the instances to test your ansible playbooks, roles and/or collections.
- (Optionally) Preparing the instances, like installing pre-requisite packages or setting up the environment.
- Converging the playbooks, roles or collections you want to test.
- Performing idempotency check by running again your ansible project.
- Verify the final state of the instances.
- Destroy the instances.

## Requirements

In a python virtual environment install the dependencies stated in `requirement.txt`

```shell
pip install -r requirements.txt
```

## How it works

This repository has 2 playbooks which follow the following workflow:

```shell
Collect Interface Raw Data > Parse data into structured format > Save to DB
```

There are 2 molecule scenarios built that test these playbooks.

---

## Test: Static Data Scenario

This scenarios aims to show how molecule can be used with `docker` plugin. It instantiates a `centos:7` docker container where a interface data `txt` file has the information and the playbook is in charge of reading it, parsing the data and save it to an Elasticsearch DB container.

To run all scneario's phases:

```shell
molecule test -s static
```

---

## Test: Mocked Data Scenario

In this scenario I am leveraging the awesome project [cisshgo](https://github.com/tbotnz/cisshgo), which give the ability to mock an SSH connection to a network device and has some predefined data which you can use to parse and test.

As a pre-requisite for this test scenario you need to build a docker image locally so it can be used:

```shell
git clone https://github.com/tbotnz/cisshgo.git
cd cisshgo
docker build -t cisshgo:latest .
docker tag cisshgo:latest molecule_local/cisshgo:latest
```

The tag is needed when molecule uses it.

To run the test scenario:

```shell
molecule test -s mock
```

---

## Test: GNS3 Scenario

In this interesting scenario you can leverage a GNS3 server as a resource provider for the molecule scenario, meaning that **molecule will create, prepare and destroy GNS3 labs and routers** for testing the ansible playbooks.

### Extra requirements:

- **GNS3 server and valid router image installed**: In this case the GNS3 server is on the same network as the host running molecule and it has the IOSv image and template installed. This means as well that connectivty from the molecule machine to GNS3 server must exist to be able to connect to its API.
- [gns3fy](https://github.com/davidban77/gns3fy)

```shell
pip install gns3fy
```

- [Ansible collection for gns3](https://galaxy.ansible.com/davidban77/gns3) for the creation and destroy playbooks to be succesfully executed

```shell
ansible-galaxy collection install davidban77.gns3
```

- [netaddr](https://github.com/netaddr/netaddr) python package to successfully run some jinja2 filters in ansible

```shell
pip install netaddr
```

### Molecule procedure in GNS3 scenario

![molecule-gns3-scenario](docs/interop2020-molecule-gns3.svg)

The steps shown in the diagram are executed in the following molecule phases:

- **Create and prepare**: Step 1 and 2. Uses the GNS3 ansible modules to connect to server, create lab, create router and push initial bootstrap SSH config over console.
- **Converge**: Step 3 and 4. Where the playbook being tested is executed. It follows the same procedure as the steps above. Important note here is that the connection is now `network_cli`.
- **Verify**: Step 5. Could have been the same `verify.yml` of the other scenarios but since the interfaces were different, I just copied and replaced the interfaces to be verified.
- **Destroy**: Step 6. It deletes the GNS3 lab and router **AND** the docker networks and container (elasticsearch). So all resources being used are released.
