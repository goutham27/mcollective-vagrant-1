# -*- mode: ruby -*-
# vi: set ft=ruby :

# apart from the middleware node, create
# this many nodes in addition to the middleware
INSTANCES=5

# the nodes will be called middleware.example.net
# and node0.example.net, you can change this here
DOMAIN="cmfl2.net"

# these nodes do not need a lot of RAM, 384 is
# is enough but you can tweak that here
# This is for each node you spin up.
MEMORY=384

# activemq + mcollective broker memory requirements
MIDDLEWARE_MEMORY=768

# the instances is a hostonly network, this will
# be the prefix to the subnet they use
SUBNET="192.168.2"

# The default vagrant box to use
BOX=ENV['VAGRANT_BOX'] ||= 'centos6u7'

BOX_URL=ENV['VAGRANT_BOX_URL'] ||= "file:///#{ENV['HOME']}/boxes/centos6u7"

puts "USING BOX: #{BOX}"
puts "USING BOX_URL: #{BOX_URL}"

$set_puppet_version = <<EOF
/bin/rpm -Uvh http://yum.puppetlabs.com/el/6x/products/x86_64/puppetlabs-release-6-11.noarch
/usr/bin/yum clean expire-cache
/usr/bin/yum versionlock delete '0:puppet*'
/usr/bin/yum versionlock delete '1:facter*'
/usr/bin/yum -y install puppet-3.8.6-1.el6
EOF

Vagrant.configure("2") do |config|
  config.vm.define :middleware do |vmconfig|
    vmconfig.vm.box = "#{BOX}"
    vmconfig.vm.box_url = "#{BOX_URL}"
    vmconfig.vm.network :private_network, ip: "#{SUBNET}.10"
    vmconfig.vm.hostname = "middleware.#{DOMAIN}"
    vmconfig.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", MIDDLEWARE_MEMORY]
        vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
        vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    end
    vmconfig.vm.box = "#{BOX}"
    vmconfig.vm.box_url = "#{BOX_URL}"
    vmconfig.vm.provision :shell, :inline => $set_puppet_version
    vmconfig.vm.provision :puppet, :options => ["--pluginsync --hiera_config /vagrant/deploy/hiera.yaml"], :module_path => "deploy/modules", :facter => { "middleware_ip" => "#{SUBNET}.10" } do |puppet|
      puppet.manifests_path = "deploy"
      puppet.manifest_file = "site.pp"
    end
  end

  INSTANCES.times do |i|
    config.vm.define "node#{i}".to_sym do |vmconfig|
      vmconfig.vm.box = "#{BOX}"
      vmconfig.vm.box_url = "#{BOX_URL}"
      vmconfig.vm.network :private_network, ip: "#{SUBNET}.%d" % (10 + i + 1)
      vmconfig.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", MEMORY]
          vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
          vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      end
      vmconfig.vm.hostname = "node%d.#{DOMAIN}" % i
      vmconfig.vm.box = "#{BOX}"
      vmconfig.vm.box_url = "#{BOX_URL}"

      vmconfig.vm.provision :puppet, :options => ["--pluginsync --hiera_config /vagrant/deploy/hiera.yaml"], :module_path => "deploy/modules", :facter => { "middleware_ip" => "#{SUBNET}.10" } do |puppet|
        puppet.manifests_path = "deploy"
        puppet.manifest_file = "site.pp"
      end
    end
  end
end
