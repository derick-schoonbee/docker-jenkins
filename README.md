# Jenkins Server in Docker

## Components used

- [Vagrant](http://www.vagrantup.com/) to provision a virtual machine.
- [Docker](https://docs.docker.com/) to run Jenkins in a container (in a VM)
- [CoreOS](https://coreos.com/) as VM since it also clusters machines and has **Docker** installed

## Persisting data between reboots

Docker uses [volumes](https://docs.docker.com/userguide/dockervolumes/) to share data / filesystems between containers. So we start a named **Data Volume Container** who's sole purpose is just to keep our data. To ensure that the data is kept between destorying and provisioning my VM, I expose part of my local filesystem:

   *MacBook* ==> *VM* ==> *Jenkins Data* (shared):

  - Host (MacBook) : */User/xxx/projects/github.com/coreos/coreos-vagrant/jenkins* (Kept on Host)
  - Vagrant VM (CoreOS) : */home/core/share/* => Host **/User/xxx/projects/github.com/coreos/coreos-vagrant**
  - Jenkins Data (Container) :  */root/* => CoreOS **/home/core/share/jenkins**
  - Jenkins Server (Container) : */root* => Jenkins Data **/root** (Shared)


## Installation

### Docker environment

#### Test Vagrant

Ensure vagrant is running:

    $ vagrant version
    ..
    Installed Version: 1.6.3
    Latest Version: 1.6.3
    ..
    You're running an up-to-date version of Vagrant!

#### Clone CoreOS Vagrant config

Note: The following section is to modify the standard CoreOS vagrant file. TODO: Provision a Vagrantfile that does these steps automatically.

Clone the Vagrant provision file for CoreOS:

    $ git clone https://github.com/coreos/coreos-vagrant.git
    $ cd coreos-vagrant

Ensure that we are sharing our local cloned directory:

    $ vi Vagrantfile
    ..
    Uncomment this line:
    config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']


You can setup a cluser or setup one machine as per CoreOS installation instructions. We'll use a single machine for:

    $ cp user-data.sample user-data
    $ cp config.rb.sample config.rb

Setup single machine:

    $ vi config.rb

    #Update the following variables / uncomment:
    $num_instances=1
    $update_channel='beta'
    $expose_docker_tcp=2375
    $vb_gui = false
    $vb_memory = 1024
    $vb_cpus = 1

    #Add this line (See next section)
    $expose_jenkins_port=8080

#### Share Jenkins Port

To ensure that the jenkins instance is acessible from the outside we need to map a port from the Host to the CoreOS vm. There are various ways to do it like using *vboxmanage* or the VirtualBox UI. For now we are in the config file so we just duplicate this line and change:

    #Find this line
    config.vm.network "forwarded_port", guest: 2375, host: ($expose_docker_tcp + i - 1), auto_correct: true
    #Duplicate the line above and change as below:
    config.vm.network "forwarded_port", guest: 8080, host: ($expose_jenkins_port + i - 1), auto_correct: true


#### Run Virtual machine

Ensure you have changed the configuration files and mapped a port.

    $ vagrant up

### Data Container:


Create the data container:


    $ docker run --name=jenkins-data-vol -v /home/core/share/jenkins:/root/ ubuntu:precise true
    # If you would like to create a container that resets all data when the VM is destroyed
    # you do not need to map to the host volume:
    docker run --name=jenkins-data-vol -v /root/ ubuntu:precise true

Check is the data volume is available:

    docker ps -a | grep "jenkins-data-vol"

When want to rebuild the data you can run this:

    $ docker rm  jenkins-data-vol

To inspect the data you can run a temporary container:

    $ docker run --rm --volumes-from=jenkins-data-vol -i -t ubuntu:precise bash

### Jenkins Container

The build the image:

    $ docker build -t derick/jenkins-server server

Start the container. (Ensure that the data container was created before)

    $ docker run --name=jenkins-server --volumes-from=jenkins-data-vol -p 8080:8080 -d -t derick/jenkins-server

To inspect the jenkins logs:

    $ docker logs -f jenkins-server

To stop and restart the server:

    $ docker stop jenkins-server
    #We need to remove the container if we want to start it again
    $ docker rm jenkins-server
    #We can now start the container again.

The container should now be available via:

    $ open http://localhost:8080
