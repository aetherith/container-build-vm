# Building Containers with Ansible and Buildah
This little collection of configuration and playbooks exists to try and create
a viable builder image that packs up all the tools that we need to set up VMs
with Ansible and provide a Python3 environment for doing it. I also intend it to
be a model of how we can create containers outside or without the action of OKD.

## Requirements
* Vagrant
* VirtualBox
* Ansible

## Process
* Bring a new VM up and let it configure using `vagrant up`. If you want to see
  what the VM is seeded with, the playbook is `provision/bootstrap.yml`.
* To keep your VM and local development environment in sync you will also want
  to keep `vagrant rsync-auto` running while editing files so they are synced
  to and from the VM.
* To actually build the container, log into the VM with `vagrant ssh` and use the
  following to switch to the root user.
  ```bash
  $ deactivate && sudo su
  ```
* From there execute the following to build the container image:
  ```bash
  $ ansible-bender build /vagrant/provision/jnlp-container.yml
  $ ansible-bender build /vagrant/provision/builder-container.yml
  ```

## Notes
* It is *possible* to run `buildah` as a non-root user but apparently it is both
  less efficient and much fussier to set up. Thus for this testing/training
  experience I've simplified to just running it as root.
* There are some gyrations involved getting Ansible to run on the remote using
  Python3 since Ansible seems to prefer whatever is installed at `/usr/bin/python`
  or whatever it detects through "magic". This is why in both the bootstrap and
  container playbooks I install Python3, then start an entirely new play specifying
  the Ansible Python interpreter to be the Python3 one I installed in the previous
  step.
* We could likely simplify the playbooks a bit if we included more configuration
  in `/etc/profile` around setting PATH to include `/usr/local/bin` so that when
  running tasks as root we can access binaries installed by PIP.
* All of the Ansible Bender configuration and container spec stuff is *only* read
  from the first play. So while you can have multiple plays modify the same
  container, you can't define multiple containers in a single playbook.
* The JNLP configuration was cribbed off of a combination of `jenkinsci/docker-agent`
  and `jenkinsci/docker-inbound-agent`. The JNLP script itself is a direct lift
  from the `jenkinsci/docker-inbound-agent` repository.
* Since I want to build the container in two stages I *actually* have to specify
  the entrypoint on the second definition, otherwise `podman` throws an error
  when it can't load the JNLP JAR from Jenkins.

## TODO
* Test pushing the built images to a registry of some kind.
* Test different methods of minimizing container size.
* Test that JNLP container actually works as expected.
