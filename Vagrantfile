Vagrant.configure(2) do |config|
  config.vm.box = "chef/centos-6.5"

  config.vm.define "base" do |base|
  end

  config.vm.define "extra_storage" do |extra_storage|
    devices = [
      {disk: 'C:\tmp\extra_disk_1.vdi', size: 1},
      {disk: 'C:\tmp\extra_disk_2.vdi', size: 2}
    ]

    extra_storage.vm.provider "virtualbox" do |vbox|
      vbox.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata"]

      devices.each do |device|
        disk = device[:disk]
        size = device[:size]
        port = devices.index device

        vbox.customize ["createhd", "--filename", disk, "--size", size * 1024] unless File.exist? disk
        vbox.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", port, "--device", 0, "--type", "hdd", "--medium", disk]
      end
    end
  end
end
