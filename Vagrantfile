# -*- mode: ruby -*-

Vagrant.configure(2) do |config|
  config.vm.box = "aws"
  config.ssh.username = "centos"
  config.vm.synced_folder ".", disabled: true
  
  N = 3
    (1..N).each do |machine_id|

      config.vm.define "machine#{machine_id}" do |machine|
        machine.vm.provider :aws do |aws, override|
           aws.access_key_id = ENV['AWS_ACCESS_KEY']
           aws.secret_access_key = ENV['AWS_SECRET_KEY']
           aws.region = "us-west-1"
           aws.availability_zone = "us-west-1b"
           aws.subnet_id = "subnet-1d46ee79"
           aws.associate_public_ip = true
           aws.ami =  "ami-f5d7f195"
           aws.keypair_name = "aws-esikachev"
           aws.instance_type = "t2.micro"
           aws.security_groups = ["sg-5fb6a638"]
           aws.block_device_mapping = [
              {
                'DeviceName' => "/dev/sda1",
                'Ebs.DeleteOnTermination' => true
              }
           ]

          override.ssh.private_key_path = ENV['AWS_PRIVATE_KEY']
        end

        machine.vm.hostname = "machine#{machine_id}"
        if machine_id == N
          machine.vm.provision :ansible do |ansible|
            ansible.limit = "all,localhost"
            ansible.playbook = "site.yml"
            ansible.groups = {
              "hadoop"              => ["machine[1:]"],
              "all:children"        => ["hadoop"],
            }
            if N > 1
              ansible.groups["namenodes"] = "machine1"
              ansible.groups["oozie"] = "machine1"
              ansible.groups["yarnresourcemanager"] = "machine1"
              ansible.groups["datanodes"] = "machine[2:#{N}]"
            end
            ansible.sudo = true
          end
        end
      end
  end
end
