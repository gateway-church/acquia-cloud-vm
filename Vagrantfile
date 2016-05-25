# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
if !File.exist?('./config.yml')
  raise 'Configuration file not found! Please copy example.config.yml to config.yml and try again.'
end
vconfig = YAML::load_file("./config.yml")

plugin_no = 0
required_plugins = %w( vagrant-hostsupdater )
required_plugins.each do |plugin|
  (system "vagrant plugin install #{plugin}"; plugin_no += 1) unless Vagrant.has_plugin? plugin
end
(puts "Installed #{plugin_no} new plugins. Please restart the last command."; exit ) if plugin_no > 0

if ! vconfig['acquia_ssh_private_key']
  raise 'Configuration variable not found! Please set acquia_ssh_private_key in config.yml and try again.'
end

# Check add the private key to ssh if it is missing
added_keys = `ssh-add -l`
private_key = File.expand_path(vconfig['acquia_ssh_private_key'])
if ! added_keys.include? private_key
  system 'ssh-add', private_key
  check_added_keys = `ssh-add -l`
  if ! check_added_keys.include? private_key
    raise "#{private_key} wasn't added"
  end
end

Vagrant.configure("2") do |config|
  config.vm.hostname = vconfig['vagrant_hostname']
  config.vm.network :private_network, ip: vconfig['vagrant_ip']
  config.hostsupdater.aliases = vconfig['vagrant_aliases']
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  config.vm.box = "geerlingguy/ubuntu1404"

  for synced_folder in vconfig['vagrant_synced_folders'];
    config.vm.synced_folder synced_folder['local_path'], synced_folder['destination'],
      type: synced_folder['type'],
      rsync__auto: "true",
      rsync__exclude: synced_folder['excluded_paths'],
      rsync__args: ["--verbose", "--archive", "--delete", "-z", "--chmod=ugo=rwX"],
      id: synced_folder['id']
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.sudo = true
  end

  # VMware Fusion.
  config.vm.provider :vmware_fusion do |v, override|
    # HGFS kernel module currently doesn't load correctly for native shares.
    override.vm.synced_folder ".", "/vagrant", type: 'nfs'

    v.gui = false
    v.vmx["memsize"] = vconfig['vagrant_memory']
    v.vmx["numvcpus"] = vconfig['vagrant_cpus']
  end

  # VirtualBox.
  config.vm.provider :virtualbox do |v|
    v.name = vconfig['vagrant_hostname']
    v.memory = vconfig['vagrant_memory']
    v.cpus = vconfig['vagrant_cpus']
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define :gwayppl do |cloudvm_config|
  end

end
