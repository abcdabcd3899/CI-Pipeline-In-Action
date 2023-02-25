# CI Pipeline In Action

![Github stars](https://img.shields.io/github/stars/abcdabcd3899/CI-Pipeline-In-Action.svg)
![Github language](https://img.shields.io/badge/language-YAML-green.svg)
![License](https://img.shields.io/github/license/abcdabcd3899/CI-Pipeline-In-Action.svg)
![Forks](https://img.shields.io/github/forks/abcdabcd3899/CI-Pipeline-In-Action.svg)

## Outline

* [Concourse Pipeline](#cp)
* [Github Actions](#ga)
* [CircleCI](https://circleci.com/docs/)

## <span id='cp'>Concourse Pipeline</span>

### Environments

* ubuntu/centos x86_64 ✅
* linux arm64 ubuntu/centos ❌
* m1 pro apple silicon macos ❌
* windows 🤷‍♂️

### Install Docker

```shell
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli docker-compose containerd.io docker-compose-plugin
sudo service docker start
sudo service docker status
docker --version
# sudo docker run hello-world
# exit
```

### Install Concourse CI

```shell
git clone https://github.com/concourse/concourse-docker.git
cd concourse-docker
./keys/generate
docker-compose up -d
docker-compose down
```

### Install Fly CLI

<https://github.com/concourse/concourse/releases>

Download the `fly`tool and decompress. Then moving the fly to `/usr/local/bin`

### Demo Show

```shell
fly -t ci login -c http://localhost:8080 -u test -p test
fly targets
```

Open web browser `http://localhost:8080`, user_name: test password: test

<img width="1916" alt="截屏2022-09-30 16 56 23" src="https://user-images.githubusercontent.com/13810907/193233277-4ee02ae3-3d8e-45a5-bc80-4c949de790b3.png">

* pipeline.yml

```yaml
jobs:
  - name: job-hello-csapp
    public: true
    plan:
      - task: hello-csapp
        config:
          platform: linux
          image_resource:
            type: docker-image
            source: {
                repository: ubuntu,
                tag: 20.04
          }
          run:
            path: echo
            args: [hello, csapp!]
```

```shell
fly -t ci set-pipeline -p test_pipeline -c pipeline.yml
```

<img width="1912" alt="截屏2022-10-02 15 04 13" src="https://user-images.githubusercontent.com/13810907/193442261-0069fad0-02a9-4aa1-b5e7-05b02566f613.png">

### Excerises

[References](https://github.com/zhangh43/hire)

#### Setup a Concourse CI(Continuous Integration) example pipeline

1. Refer to [Quick Start](https://concourse-ci.org/quick-start.html) to deploy Concourse using docker-compose.
2. Setup the hello-world pipeline based on [the tutorial](https://concourse-ci.org/tutorial-hello-world.html).
3. Change the hello-world pipeline with two new requirements:

* Requirement I.
Concourse will run CI jobs in a docker container. Please create a user-defined docker image and upload it to docker hub.
Then Concourse can fetch and use this container. Docker Image requirement:

  1. base image: Ubuntu20.04
  2. install package libleveldb-dev
  3. generate a rsa key.
  4. [Plus] enable coredump

  The following yaml generate the coredump file. It is need for `privileged: true`.

```yaml
jobs:
    - name: coredump
      public: true
      plan:
        - task: execute-the-tasks
          privileged: true
          config:
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               user: root
               path: /bin/bash
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  ulimit -c unlimited
                  echo 'ulimit -c unlimited' >> ~/.bash_profile
                  echo '/tmp/core.%t.%e.%p' | tee /proc/sys/kernel/core_pattern
                  apt-get update
                  apt-get install git -y
                  apt-get install gcc -y
                  apt-get install vim -y
                  apt-get install libleveldb-dev -y
                  apt-get install openssh-client -y
                  ssh-keygen -q -t rsa -N '' -f /root/.ssh/id_rsa
                  source ~/.bash_profile
                  cat /root/.ssh/id_rsa.pub
                  git clone https://github.com/abcdabcd3899/Computer-Systems-Labs
                  cd Computer-Systems-Labs/gdb_test
                  gcc -g -o test test3.c
                  ./test
```

* Requirement II. Instead of just echo helloworld, you should write a simple Concourse CI pipeline to get a github repo, compile a C program and run the program. Refer to [Concourse documentation](https://concourse-ci.org/docs.html).
  The expected jobs:

  1. Fetch a [github repo](https://github.com/abcdabcd3899/Computer-Systems-Labs)
  2. Implement the [lab2](https://github.com/abcdabcd3899/Computer-Systems-Labs/tree/main/lab2/datalab-handout)

A: pipeline.yml

```yaml
jobs:
    - name: lab2
      public: true
      plan:
        - task: execute-the-tasks
          config:
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               path: /bin/sh
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  apt-get update
                  apt-get install gcc -y
                  apt-get install gcc-multilib -y
                  apt-get install make -y
                  apt-get install git -y
                  git clone https://github.com/Lily127Yang/Computer-Systems-Labs.git
                  cd Computer-Systems-Labs/lab2/datalab-handout
                  make
                  ./btest -T 50
                  result=`./btest -T 50 | grep "Total point" | cut -d " " -f3 | cut -d "/" -f1`
                  ddl=`date -d "2022-10-09 23:59" +%s --utc`
                  current_time=`date +%s`
                  [ $current_time -le $ddl ]
                  [ $result -ge 36 ]
```

or using the following method (3. [Plus] Using Concourse github resource instead of clone the repo manually (refer to [the tutorial](https://github.com/concourse/git-resource)))

```yaml
resources:
- icon: github
  name: csapp
  source:
    uri: https://github.com/Lily127Yang/Computer-Systems-Labs.git
  type: git

jobs:
    - name: lab2
      public: true
      plan:
        - get: csapp
          trigger: true
        - task: execute-the-tasks
          config:
            inputs:
              - name: csapp
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               path: /bin/sh
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  apt-get update
                  apt-get install gcc -y
                  apt-get install gcc-multilib -y
                  apt-get install make -y
                  apt-get install git -y
                  cd csapp/lab2/datalab-handout
                  make
                  ./btest -T 50
                  result=`./btest -T 50 | grep "Total point" | cut -d " " -f3 | cut -d "/" -f1`
                  ddl=`date -d "2022-10-09 23:59" +%s --utc`
                  current_time=`date +%s`
                  [ $current_time -le $ddl ]
                  [ $result -ge 36 ]
```

start docker container inspect the `run` command

```shell
sudo docker run -it ubuntu bash
```

### Debug from Concourse CI

```shell
fly -t ci_name login # login to concourse
fly targets
fly -t ci_name hijack -u build_url # http://localhost:8080/teams/main/pipelines/core/jobs/coredump/builds/10
apt-get install gdb -y
gdb xxx coredump_file
```

### Download from Concourse

```shell
fly -t ci hijack -u http://localhost:8080/teams/main/pipelines/core/jobs/coredump/builds/10 -s execute-the-tasks -- base64 /tmp/core.1665048466.test.6 | base64 -d > core # -s is at pipeline.yml or fly -t ci hijack -u http://localhost:8080/teams/main/pipelines/core/jobs/coredump/builds/10 output
```

## <span id="ga">GitHub Actions</span>

Reference by [my github action workflow](https://github.com/abcdabcd3899/Computer-Systems-Labs/tree/main/.github/workflows)

## Reference

1. [Concourse](https://github.com/concourse/concourse)
