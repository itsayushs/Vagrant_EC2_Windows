# Vagrant to Provision AWS EC2 instance with Windows AMI & WinRM support
 ### Deps:
 - vagrant plugins : vagrant plugin install [vagrant-aws | vagrant-aws-winrm]
 - ansible : pip install pywinrm

### How to use:
- Install the plugins and add a Dummy Vagrant box. 
```
# vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```
- Create a public and private_key pairs and register there Keypair_name on aws.
```
# ssh-keygen -t rsa -f ./aws_win
```
- Create a Vagrant File with following content:
```
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

```
 - Here a startup powershell script is used called statupsps.txt. This makes the Instance capable to do following tasks:
 -- enables PSRemoting i.e. allows powershell to take remote inputs.
-- sets a firewall rule for WinRM to ingress and egress.
-- Configs WinRM with Basic Auth over HTTP
-- Sets Some WinRM Params required for ansible to run.
```
<powershell>
Enable-PSRemoting -Force
netsh advfirewall firewall add rule name="WinRM HTTP" dir=in localport=5985 protocol=TCP action=allow
winrm quickconfig -force
Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
</powershell>
```
- Save the files into a folder and then run.
-- Provision may take upto 4-6 mins.
```
#  vagrant up --provider=aws
```
### Using Ansible:
```
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
    config.vm.provision :ansible do |ansible|
       ansible.host_vars = {"ansible_port" => 5985,"ansible_user" => 'Administrator',"ansible_connection" => "winrm"}
       ansible.playbook = "playbook.yml"
  end
 end
end

```
- Contents inside playbook.yml
```
- hosts: all
  tasks:
  - name: Create directory structure
    win_file:
     path: C:\Temp\folder\subfolder
     state: directory

```
- Use following commands:
```
#  vagrant up
#  vagrant provision
```
