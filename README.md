# Домашнее задание
1.Установка Ubuntu 20.04 на AH Websa
2.Установка VirtualBox
3.Установка Vagrant
4.Установка Packer
5.Настройка Git
6.Создать образ с обновленным ядром
7.Выгрузить образ в Vagrant Vloud

2. ## Установка VirtualBox с терминала Ububntu
  1. Установка утилиты curl
     ```
     apt install curl -y
     ```
  2. Установка Virtualbox
     ```
     curl -O https://download.virtualbox.org/virtualbox/6.1.32/virtualbox-6.1_6.1.32-149290~Ubuntu~eoan_amd64.deb
     sudo dpkg -i virtualbox-6.1_6.1.32-149290~Ubuntu~eoan_amd64.deb
     ```
3. ## Установка Vagrant
   Стандартная процедура, описанная на официальном сайте, устанвыливаем для Ubuntu.
   ```
   sudo apt-get update && sudo apt-get install vagrant
   ```
4. ## Установка Packer
   Анологично, как и с Vagrant
   ```
   sudo apt-get update && sudo apt-get install packer
   ```
5. ## Настройка Git
   1. Создаем учетную запись на Git
   2. Выполняем установку Git
   3. Генерируем SSH ключ и добовляем его в настройках нашего профиля
   4. Делаем fork репозитория: ```https://github.com/dmitry-lyutenko/manual_kernel_update```
   5. Создаем директории на локальном ПК, куда разместим клон репозитория:
      ```
      mkdir -p /home/adminroot/git/
      cd /home/adminroot/git/
      ```
   6. Создаем свой репозиторий на git для домашней работы
      ```
      SATimur/otus-homework
      ```
   7. Клонируем репозитории на локальный компьютер
      ```
      git clone git@github.com:SATimur/manual_kernel_update.git
      git clone git@github.com:SATimur/otus-homework.git
      ```
   8. Переносим файлы с manual_kernel_update в otus-homework
      ```
      cp -r /home/adminroot/git/manual_kernel_update/packer /home/adminroot/git/otus-homework/
      cp /home/adminroot/git/manual_kernel_update/Vagrantfile /home/adminroot/git/otus-homework/
      ```
6. ## Создаем образ с обновленным ядром
   1.Вносим изменения в Vagrantfile по пути ```/home/adminroot/git/otus-homework  ; nano Vagrantfile
   меняем наименование box_name => "centos-7" ```
   Моя конфигурация
   ```
      # Describe VMs
      MACHINES = {
      # VM name "kernel update otus"
      :"kernel-update-otus" => {
              # VM box
              :box_name => "centos-7",
              # VM CPU count
              :cpus => 2,
              # VM RAM size (Mb)
              :memory => 1024,
              # networks
              :net => [],
              # forwarded ports
              :forwarded_port => []
            }
      }

     Vagrant.configure("2") do |config|
     config.vm.box = "SATimur/centos-7"
     config.vm.box_version = "25599"
     MACHINES.each do |boxname, boxconfig|
     # Disable shared folders
     config.vm.synced_folder ".", "/vagrant", disabled: true
     # Apply VM config
     config.vm.define boxname do |box|
      # Set VM base box and hostname
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      # Additional network config if present
      if boxconfig.key?(:net)
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
      end
      # Port-forward config if present
      if boxconfig.key?(:forwarded_port)
        boxconfig[:forwarded_port].each do |port|
          box.vm.network "forwarded_port", port
        end
      end
      # VM resources config
      box.vm.provider "virtualbox" do |v|
        # Set VM RAM size and CPU count
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
        end
      end
    end
   end
   ```
   2.Потом переходим в директорию где лежит у нас файл centos.json ```/home/adminroot/git/otus-homework/packer```
     в файле centos.json удалим строчку "iso_checksum_type": "sha256" и заменил название образа
      ```"post-processors": [
      {
          "output": "centos-{{user `artifact_version`}}-kernel-5-x86_64-Minimal-otus.box"
      ```
7.## Запускаем Packer
  ```
  packer build centos.json
  ```
  послу успешного завершения у меня создался образ centos-7.7.1908-kernel-5-x86_64-Minimal-otus.box
8. Запускаем тестирование образа(если запустить Vagrant без теста, выдаст ошибку)
   ```
   adminroot@wvds136386:~/git/otus-homework/packer$ vagrant up
   Bringing machine 'kernel-update-otus' up with 'virtualbox' provider...
      ==> kernel-update-otus: Box 'centos-7' could not be found. Attempting to find and install...
    kernel-update-otus: Box Provider: virtualbox
    kernel-update-otus: Box Version: 25599
      ==> kernel-update-otus: Box file was not detected as metadata. Adding it directly...
    You specified a box version constraint with a direct box file
    path. Box version constraints only work with boxes from Vagrant
    Cloud or a custom box host. Please remove the version constraint
    and try again.
    ```
   ```
   vagrant box add centos-7 /home/adminroot/git/otus-homework/packer/centos-7.7.1908-kernel-5-x86_64-Minimal-otus.box
   ```
   Проверим его в списке имеющихся образов 
   ```
   vagrant box list
   centos-7            (virtualbox, 0)
   ```
   Запуск теста
   ```
   vagrant init centos-7
   ```
   ```
   adminroot@wvds136386:~/git/otus-homework/packer$ vagrant init centos-7
   A `Vagrantfile` has been placed in this directory. You are now
   ready to `vagrant up` your first virtual environment! Please read
   the comments in the Vagrantfile as well as documentation on
   `vagrantup.com` for more information on using Vagrant.
   Далее запускаем уже сам Vagrant командой vagrant up и подключаемся к виртуальной машине по ssh
   adminroot@wvds136386:~/git/otus-homework/packer$ ssh vagrant@127.0.0.1 -p2222
   vagrant@127.0.0.1's password:
   Last login: Wed Feb  9 06:15:19 2022 from 10.0.2.2
   [vagrant@localhost ~]$ uname -r
   5.16.8-1.el7.elrepo.x86_64
   [vagrant@localhost ~]$
   ```
7. Выгрузка образа в Vagrant Cloud
   Регистрируемся в Vagrant.com через web интерфейс и подклюучаемся на хостовой машине к облаку
   ```
   vagrant cloud auth login
   Vagrant Cloud username or email: <user_email>
   Password (will be hidden): 
   Token description (Defaults to "Vagrant login from DS-WS"):
   You are now logged in.
   Теперь выгружаем наш образ в облако
   vagrant cloud publish --release SATimur/centos-7 1.0 virtualbox \
        centos-7.7.1908-kernel-5-x86_64-Minimal-otus.box
   ```
 
