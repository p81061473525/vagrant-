################################################################
# 單人電腦使用版本，請自行評估自身電腦效能修改被控端節點數量。
# Version : 2021-08-31
################################################################

#############################
# 待處理
# share folder
# disk SATA 問題
# 開發環境轉移問題
# MAC 權限問題
# TGBOG,LINEBOT 安裝完成通知問題
#############################

# 走記得更改 第 35 行。 public ip 
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"

  config.vm.provision "shell", inline: <<-SHELL
    # 套件升級，安裝常用程式
    sudo yum update -y
    sudo yum install epel-release -y
    sudo yum install git htop wget vim tmux screen telnet -y

    # 開放 ssh 密碼訪問
    sudo sed -i '/^PasswordAuthentication no/d' /etc/ssh/sshd_config
    sudo systemctl restart sshd
  SHELL
  
    # 控制端另外寫。
  config.vm.define "as-50-200" do |machine|
    machine.vm.box = "centos/7"
    machine.vm.hostname = "as-50-200"
    machine.vm.network "private_network", ip: "192.168.50.200"
    # machine.vm.network "forwarded_port", guest: 22, host: 2222, host_ip: "127.0.0.1", id: "ssh", auto_correct: true
    machine.vm.provision "shell", inline: <<-SHELL
    # 套件升級，安裝常用程式
      sudo yum install ansible -y
    SHELL

    machine.vm.provider "virtualbox" do |v|
      v.memory = 4096
      v.cpus = 4
    end

    machine.vm.provider "virtualbox" do |vb|
      unless File.exist?("./storage/home_disk-50-200-0.vmdk")
        vb.customize ['storagectl', :id, '--name', 'SATA', '--add', 'sata']
      end
        # 設定 SATA controller 掛載 五顆硬碟 2GB 硬碟，
        # https://github.com/numenta/nupic/issues/2311 , commit : elliotholden commented on 28 Jul 2020
        # 迴圈參考 vagrant 官方網站，https://www.vagrantup.com/docs/provisioning/ansible 寫法教學。
        # M = 被控端節點數量。
      M = 4
      (0..M).each do |bb|
        unless File.exist?("./storage/home_disk-50-200-#{bb}.vmdk")
          vb.customize ['createmedium', '--filename', "./storage/home_disk-50-200-#{bb}.vmdk", '--format', 'VMDK', '--variant', 'Standard', '--size', 2 * 1024]
          vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', "#{bb}", '--device', 0, '--type', 'hdd', '--medium', "./storage/home_disk-50-200-#{bb}.vmdk"]
        end
      end
    end
  end

# 迴圈參考 vagrant 官方網站，https://www.vagrantup.com/docs/provisioning/ansible 寫法教學。
# N = 總機器數量
  N = 4
  (1..N).each do |machine_id|
    config.vm.define "ac-50-20#{machine_id}" do |machine|
      machine.vm.hostname = "ac-50-20#{machine_id}"
      machine.vm.network "private_network", ip: "192.168.50.#{200+machine_id}"
     
      machine.vm.provider "virtualbox" do |vb|
        unless File.exist?("./storage/home_disk-50-20#{machine_id}-0.vmdk")
          vb.customize ['storagectl', :id, '--name', 'SATA', '--add', 'sata']
        end
        
        # 設定 SATA controller 掛載 五顆硬碟 2GB 硬碟，
        # https://github.com/numenta/nupic/issues/2311 , commit : elliotholden commented on 28 Jul 2020
        # 迴圈參考 vagrant 官方網站，https://www.vagrantup.com/docs/provisioning/ansible 寫法教學。
        # M = 被控端節點數量。

          M = 4
          (0..M).each do |bb|

          unless File.exist?("./storage/home_disk-50-20#{machine_id}-#{bb}.vmdk")
            vb.customize ['createmedium', '--filename', "./storage/home_disk-50-20#{machine_id}-#{bb}.vmdk", '--format', 'VMDK', '--variant', 'Standard', '--size', 2 * 1024]
            vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', "#{bb}", '--device', 0, '--type', 'hdd', '--medium', "./storage/home_disk-50-20#{machine_id}-#{bb}.vmdk"]
          end
        end
      end
    end
  end
end