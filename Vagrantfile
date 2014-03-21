require 'yaml'

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'rackspace'

dir = File.dirname(File.expand_path(__FILE__))

configValues = YAML.load_file("#{dir}/puphpet/config.yaml")
apiValues = YAML.load_file("#{dir}/puphpet/config-api.yaml")
data = configValues['vagrantfile-rackspace']

Vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  config.vm.hostname = "#{data['vm']['hostname']}"

  config.ssh.private_key_path = "#{data['ssh']['private_key_path']}"
  config.ssh.username = "#{apiValues['ssh_username']}"

  config.vm.provider :rackspace do |rs, override|
    rs.username           = "#{apiValues['username']}"
    rs.api_key            = "#{apiValues['api_key']}"
    rs.flavor             = /#{data['vm']['provider']['rackspace']['size']}/
    rs.image              = /#{data['vm']['provider']['rackspace']['image']}/
    rs.rackspace_region   = "#{data['vm']['provider']['rackspace']['region'].downcase}"
    rs.public_key_path    = "#{data['ssh']['public_key_path']}"
    rs.server_name        = "#{data['vm']['provider']['rackspace']['server_name']}"
    override.ssh.username = "root"
  end

  data['vm']['synced_folder'].each do |i, folder|
    if folder['source'] != '' && folder['target'] != '' && folder['id'] != ''
      config.vm.synced_folder "#{folder['source']}", "#{folder['target']}", id: "#{folder['id']}"
    end
  end

  config.vm.provision "shell" do |s|
    s.path = "puphpet/shell/initial-setup.sh"
    s.args = "/vagrant/puphpet"
  end
  config.vm.provision :shell, :path => "puphpet/shell/update-puppet.sh"
  config.vm.provision :shell, :path => "puphpet/shell/r10k.sh"

  config.vm.provision :puppet do |puppet|
    ssh_username = !apiValues['ssh_username'].nil? ? apiValues['ssh_username'] : "vagrant"
    puppet.facter = {
      "ssh_username" => "#{ssh_username}"
    }
    puppet.manifests_path = "#{data['vm']['provision']['puppet']['manifests_path']}"
    puppet.manifest_file = "#{data['vm']['provision']['puppet']['manifest_file']}"

    if !data['vm']['provision']['puppet']['options'].empty?
      puppet.options = data['vm']['provision']['puppet']['options']
    end
  end

  config.vm.provision :shell, :path => "puphpet/shell/execute-files.sh"

  if !data['ssh']['host'].nil?
    config.ssh.host = data['ssh']['host']
  end
  if !data['ssh']['port'].nil?
    config.ssh.port = "#{data['ssh']['port']}"
  end
  if !data['ssh']['guest_port'].nil?
    config.ssh.guest_port = data['ssh']['guest_port']
  end
  if !data['ssh']['shell'].nil?
    config.ssh.shell = "#{data['ssh']['shell']}"
  end
  if !data['ssh']['keep_alive'].nil?
    config.ssh.keep_alive = data['ssh']['keep_alive']
  end
  if !data['ssh']['forward_agent'].nil?
    config.ssh.forward_agent = data['ssh']['forward_agent']
  end
  if !data['ssh']['forward_x11'].nil?
    config.ssh.forward_x11 = data['ssh']['forward_x11']
  end

end

