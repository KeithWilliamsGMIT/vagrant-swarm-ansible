BOX_IMAGE = "bento/ubuntu-18.04"
SWARM_MANAGER_COUNT = 1
SWARM_WORKER_COUNT = 1
NODE_COUNT = SWARM_MANAGER_COUNT + SWARM_WORKER_COUNT

Vagrant.configure("2") do |config|
  # Create VMs in a loop
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = BOX_IMAGE
      node.vm.hostname = "node#{i}"
      # Avoid assigning static IP address ending in .1 to nodes
      # as it is often  used by the router and can cause issues
      # with the network.
      node.vm.network :private_network, ip: "192.168.2.#{i + 1}"
      node.vm.network :public_network, ip: "192.168.1.#{i + 149}", bridge: "wlo1"

      disk_name = "./disks/disk#{i}.vdi"

      node.vm.provider "virtualbox" do |vb|
        # Build disk files if they don't exist
        if not File.exists?(disk_name)
          vb.customize ['createhd', '--filename', disk_name, '--variant', 'Fixed', '--size', 10 * 1024]
        end

        # Attach disk using the SATA controller
        vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', i, '--device', 0, '--type', 'hdd', '--medium', disk_name]
      end
    end

    # Enable X11 forwarding for debugging purposes
    config.ssh.forward_agent = true
    config.ssh.forward_x11 = true

    # Provision all VMs with Anisble playbook and generate inventory file
    config.vm.provision 'ansible' do |ansible|
      ansible.playbook = "provision.yml"
      ansible.groups = {
        "docker_swarm_manager" => ["node[1:#{SWARM_MANAGER_COUNT}]"],
        "docker_swarm_worker" => ["node[#{SWARM_MANAGER_COUNT + 1}:#{NODE_COUNT}]"],
        "vagrant" => ["node[1:#{NODE_COUNT}]"]
      }
    end
  end
end
