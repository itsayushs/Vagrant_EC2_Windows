# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  config.vm.provider :aws do |aws, override|
    aws.access_key_id = ""
    aws.secret_access_key = ""
    aws.region = 'ap-south-1'
    aws.ami = "ami-0572a5af4c190f430"
    aws.keypair_name = "aws_win"
    aws.instance_type = "t2.micro"
    aws.terminate_on_shutdown = true
    aws.security_groups = ["sg-1575267f"]
    aws.subnet_id = "subnet-a1a0e3c9"
    aws.associate_public_ip = true
    config.vm.communicator = "winrm"
    config.winrm.username = "Administrator"
    override.winrm.password = :aws
    override.ssh.private_key_path = "/home/ayush/ansible_stuff/win_enduser/Vagrant/aws_win"
    aws.user_data = File.read("startupps.txt")
end
end

