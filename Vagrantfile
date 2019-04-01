################################################################################
# Big Data Seminar. Vagrant virtual machine setup.
################################################################################
# Run 'vagrant' in this directory to interact with the virtual machines. Some
# example command lines:
# - To get help on all commands or a specific command:
#   vagrant help
#   vagrant help up
# - To create and start the VMs:
#   vagrant up --provision --parallel
# - To connect to the first VM:
#   vagrant ssh node0
# - To shut down the VMs (can be restarted with 'vagrant up'):
#   vagrant halt
# - To tear down *all* VMs (also deletes all files outside the 'share' folder!):
#   vagrant destroy --parallel --force
# - To tear down individual VMs (also deletes all files outside the 'share'
#   folder!):
#   vagrant destroy --parallel --force node0 node1
################################################################################

# The number of nodes in the virtual cluster. If this number is reduced after
# creating the VMs, the surplus ones are not destroyed automatically -- you need
# to manually run the destroy command for the corresponding nodes before
# reducing the number.
$num_nodes = 1

# If set to true, run additional setup scripts in the 'scripts' folder during VM
# provisioning
$run_scripts = false

################################################################################

$network_setup_script = <<SCRIPT
echo "Setting up networking for $(hostname)"

/bin/cat > /etc/hosts <<EOF
#{$num_nodes.times.collect {|i| "10.42.23.#{100+i} node#{i} node#{i}.cluster"}.join("\n")}
EOF

# ssh-config
sudo -u vagrant echo "StrictHostKeyChecking no" > /home/vagrant/.ssh/config
SCRIPT

require 'find'
$additional_setup_scripts = Find.find("scripts").select { |f| (File.file? f) and (f.end_with? ".sh") }.sort


Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  $num_nodes.times do |i|
    config.vm.define :"node#{i}" do |node|
      name = "node#{i}"
      config.vm.provider "virtualbox" do |v|
        v.linked_clone = true
        v.name = name
        v.customize ["modifyvm", :id, "--groups", "/bigdata-seminar"]
        v.memory = 512
        v.cpus = 1
      end
      config.vm.network :private_network, ip: "10.42.23.#{100 + i}", :mac => "0E#{sprintf("%010X", i)}"
      config.vm.network "forwarded_port", guest: 22, id: "ssh", host: 2200 + i, auto_correct: false
      config.vm.hostname = "#{name}.cluster"
      config.vm.provision :shell, :inline => $network_setup_script
      config.vm.synced_folder "share", "/home/vagrant/share", create: true
      if $run_scripts then
        $additional_setup_scripts.each do |s|
          config.vm.provision :shell, :path => s
        end
      end
    end
  end

end
