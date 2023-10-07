##### Part 1 - Install Ansible  #####

- 2 adet Amazon Linux 2 and 1 adet ubuntu 22.04 instances ayağa kaldır:
  
    1. control_node ----> (SSH PORT 22, HTTP PORT 80)          (Amazon Linux 2)
    2. node1        ----> (SSH PORT 22, HTTP PORT 80)          (Amazon Linux 2)
    3. node2        ----> (SSH PORT 22, HTTP PORT 80)          (Ubuntu 22.04)

```bash (.bashrc dosyası içine)
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "
sudo hostnamectl set-hostname "control-node"
bash
```

  ### "control_node"a SSH ile bağlan ve ```Amazon Linux 2``` içerisine "ansible" kurmak için aşağıdaki komutları çalıştır :

  - https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

```bash
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "  # sarı-mavi-sarı
export PS1="\[\e[1;34m\]\u\[\e[38;5;214m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "	# sarı-mavi-sarı  (SARI DAHA KOYU)
sudo hostnamectl set-hostname "Control-node"
bash
```

  ### "control_node"a SSH ile bağlan ve ```Amazon Linux 2023``` içerisine "ansible" kurmak için aşağıdaki komutları çalıştır :

```bash  (pwd: /home/ec2-user) (control_node) (Amazon Linux 2023)
# BU KURULUMLA "config file = None" , YANİ "config" DOSYASI OLUŞMUYOR. "config file" "ansible.cfg"  BİZ KENDİMİZ OLUŞTURMAMIZ GEREKİYOR.
sudo dnf update -y
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
pip3 install --user ansible
```

  ### ```Ubuntu 20.04``` içerisine "ansible" kurmak için aşağıdaki komutlar kullanılır:
```bash
sudo apt update && apt upgrade -y
sudo apt install ansible -y
```


```bash
ansible --version   # Kurulum başarılı mı kontrol et
```

  ### control_node içerisinde ansible konfigürasyonunda ayarlama yapacağız.

- https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters


- control_node içerisinde ```/etc/ansible/hosts``` dosyasına ekleme yaparak `temel` seviyede `inventory` oluşturacağız, yani dosya içerisinde kontrol edeceğimiz node'ların bilgilerini gireceğiz ki control_node ile ulaşalım ve yönetelim, ayrıca belirli başlıklar altında node'ları gruplandıracağız ki yönetirken/komutları koşarken bize yardımcı olsun. 

```bash (pwd: /home/ec2-user)
cd /etc/ansible
ls
cat ansible.cfg
sudo vim hosts

[linuxserver]
node1 ansible_host=<node1_ip> ansible_user=ec2-user   # <node1_ip> : Private IP ==>> public IP de olabilir ama private tercih etmeliyiz, public IP'ler değişebilir.
# node1 ansible_host=172.31.94.235 ansible_user=ec2-user

[ubuntuserver]
node2 ansible_host=<node2_ip> ansible_user=ubuntu   # <node2_ip> : Private IP ==>> public IP de olabilir ama private tercih etmeliyiz, public IP'ler değişebilir.
# node2 ansible_host=172.31.80.6 ansible_user=ubuntu     

[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/<pem file>
# ansible_ssh_private_key_file=/home/ec2-user/arrow.pem
```

- Lokalde bulunan arrow.pem (pem file) dosyasını control_node içerisine "/home/ec2-user"a kopyalamalısın, istersen komutla / istersen tut-çek-bırak ile :

```bash (pwd : Lokalde pem dosyasının olduğu dizin)
scp -i <pem file> <pem file> ec2-user@<public DNS name of Control Node>:/home/ec2-user
# scp -i arrow.pem arrow.pem ec2-user@ec2-18-206-223-246.compute-1.amazonaws.com:/home/ec2-user # dns veya IP farketmez
# scp -i arrow.pem arrow.pem ec2-user@18.206.223.246:/home/ec2-user   # dns veya IP farketmez
```

```bash (pwd : /home/ec2-user)
ls -l
chmod 400 arrow.pem   # pem file'ı chmod 400 yapmalısın
    -r-------- 1 ec2-user ec2-user 1678 May  1 08:49 arrow.pem
```


##### Part 2 - Ansible Ad-hoc Commands  #####

- Hosts (managed nodes)'ları kontrol edelim , bilgiler doğru mu, Ad-hoc komut ile yapalım:

```bash (pwd : /home/ec2-user)
ansible all --list-hosts
ansible linuxserver --list-hosts
ansible ubuntuserver --list-hosts
```

- Hosts'larımızın ulaşılabilir olduğundan emin olmak için çeşitli ad-hoc komutlar kullanacağız, ve bu komutlarda ==>> " ping module " kullanacağız

```bash (pwd : /home/ec2-user)
ansible all -m ping
ansible linuxserver -m ping
ansible ubuntuserver -m ping
ansible node1 -m ping
ansible node2 -m ping
ansible node1:node2 -m ping
```

- Linux 2 makinada Python2 kullanınca "warning" veriyor, çalışmaya engel değil, fakat biz yine de bu uyarıyı kapatmak istersek: Aşağıdaki "interpreter_python=auto_silent" ifadesini `ansible.cfg` içerisindeki "[defaults]" başlığı altına eklemeliyiz.

```bash (pwd : /home/ec2-user)
sudo vim /etc/ansible/ansible.cfg
vim ansible.cfg
    [defaults]
    interpreter_python=auto_silent
```

  ### Aşağıdaki ad-hoc komutları çalıştır ve incele.
  
```bash (pwd : /home/ec2-user)
ansible-doc ping    # ping modülünü inceleyebilirsin
ansible all -m ping -o    # -o : dönüşleri tek satırda almak için

ansible linuxserver -m command -a "uptime"
ansible linuxserver -a "uptime"  # ansible default module'ü "command" olduğu için yazmasak da olur.
      node2 | CHANGED | rc=0 >>
      07:57:22 up  2:50,  1 user,  load average: 0.00, 0.00, 0.00
      node1 | CHANGED | rc=0 >>
      07:57:22 up  2:50,  1 user,  load average: 0.00, 0.00, 0.00

ansible linuxserver -m shell -a "systemctl status sshd"

ansible linuxserver -m command -a 'df -h'  # df: disk free  ,  h: human readable
ansible linuxserver -a 'df -h'
```


  ### Shell Module kullanımı :

- /home/ec2-user/etc/ansible/  altındaki `ansible.cfg` dosyamız aşağıdaki gibi:

```bash (ansible.cfg)
[linuxserver]
node1 ansible_host=172.31.94.235 ansible_user=ec2-user

[ubuntuserver]
node2 ansible_host=172.31.80.6 ansible_user=ubuntu     

[all:vars]
ansible_ssh_private_key_file=/home/ec2-user/arrow.pem
```

  # Amazon Linux 2'ye ve Ubuntu 22.04'ye nginx kur : (/usr/share/nginx/html/ : nginx'de index.html dosyasının bulunduğu dizindir)

```bash (pwd : /home/ec2-user)
ansible linuxserver -b -m shell -a "amazon-linux-extras install -y nginx1 ; systemctl start nginx ; systemctl enable nginx"
ansible node1 -b -m shell -a "echo '<h1> Hello everbody, I am Arrow</h1>' > /usr/share/nginx/html/index.html"

ansible ubuntuserver -b -m shell -a "apt update -y ; apt install nginx -y ; systemctl start nginx ; systemctl enable nginx"
ansible node2 -b -m shell -a "echo '<h1> Hello everbody, I am Arrow</h1>' > /usr/share/nginx/html/index.html"
```


  # Amazon Linux 2'ye ve Ubuntu 22.04'ye apache kur :    (/var/www/html : apache'de  index.html dosyasının bulunduğu dizindir)

```bash (pwd : /home/ec2-user)
ansible linuxserver -b -m shell -a "yum update -y ; yum install -y httpd ; systemctl start httpd; systemctl enable httpd"
ansible node1 -b -m shell -a "chown -R www-data:www-data /var/www/html"
ansible node1 -b -m shell -a "echo '<html><body><h1>Hello everbody, I am Arrow</h1></body></html>' > /var/www/html/index.html"

ansible ubuntuserver -b -m shell -a "apt update -y ; apt-get install -y apache2 ; systemctl start apache2; systemctl enable apache2"
ansible node2 -b -m shell -a "chown -R www-data:www-data /var/www/html"
ansible node2 -b -m shell -a "echo '<html><body><h1>Hello everbody, I am Arrow</h1></body></html>' > /var/www/html/index.html"
```


  # Amazon Linux 2'den ve Ubuntu 22.04'ten  nginx kaldır :

```bash (pwd : /home/ec2-user)
ansible linuxserver -b -m shell -a "yum -y remove nginx"
ansible ubuntuserver -b -m apt -a "autoclean=yes autoremove=yes"
```

  ### Yum ve Package Module kullanımı :


```bash (pwd : /home/ec2-user)
ansible-doc yum
ansible linuxserver -b -m yum -a "name=nginx state=present"    # "yum" modülü ile sadece "linuxserver" nginx kuracak
ansible -b -m package -a "name=nginx state=present" all     #  package ile  "all" dediğim için her 2 "linuxserver ve ubuntuserver" nginx kuracak
```


##### Part 3 - Kendi "inventory.txt"ini kullanmak  #####  `kendi inventory ==>> inventory.txt`

  # /home/ec2-user/etc/ansible/ansible.cfg  İÇERİSİNE hosts BİLGİLERİNİ YAZMAK YERİNE, AYRI BİR YERDE inventory OLUŞTURUYORUZ

- ```inventory``` oluştur ve düzenle : ```/home/ec2-user``` İÇERİSİNDE YENİ BİR inventory  HAZIRLA, KOMUTLARI BU DOSYAYA GÖRE KOŞ.

```bash (pwd : /home/ec2-user)    
vim inventory.txt

    [linuxserver]
    linux1 ansible_host=<node1_ip> ansible_user=ec2-user
    # web1 ansible_host=172.31.94.235 ansible_user=ec2-user

    [linuxserver:vars]
    ansible_ssh_private_key_file=/home/ec2-user/<YOUR-PEM-FILE-NAME>.pem
    # ansible_ssh_private_key_file=/home/ec2-user/arrow.pem
```

- Install/uninstall Apache server to node1. (Amazon Linux 2)

```bash (pwd : /home/ec2-user)
ansible -i inventory.txt linux1 -b -m yum -a "name=httpd state=present"   # inventory.txt : host bilgilerimin içerisinde olduğu inventory 
ansible -i inventory.txt linux1 -b -m service -a "name=httpd state=started enabled=yes"
ansible -i inventory.txt linux1 -b -m yum -a "name=httpd state=absent" 
```

- Install/uninstall Apache server to node1. (Amazon Linux 2)

```bash
ansible -i inventory.txt linux1 -b -m package -a "name=httpd state=present"
ansible -i inventory.txt linux1 -b -m service -a "name=httpd state=started enabled=yes"
ansible -i inventory.txt linux1 -b -m yum -a "name=httpd state=absent"
```


##### Part 4 - Kendi "inventory.txt"ini ve "ansible.cfg"ini kullanmak  #####  `kendi inventory ==>> inventory.txt` , `kendi ansible.cfg ==>> /home/ec2-user  dizini içerisine`

  # /home/ec2-user/etc/ansible/ansible.cfg  İÇERİSİNE hosts BİLGİLERİNİ YAZMAK YERİNE, AYRI BİR YERDE `inventory.txt` OLUŞTURUYORUZ.

- ```inventory``` oluştur ve düzenle : ```/home/ec2-user``` İÇERİSİNDE YENİ BİR `inventory`  HAZIRLA.

```bash (pwd : /home/ec2-user)    
vim inventory.txt

    [linuxserver]
    node1 ansible_host=172.31.94.235 ansible_user=ec2-user
    # node1 ansible_host=<node1_ip> ansible_user=ec2-user

    [ubuntuserver]
    node2 ansible_host=172.31.80.6 ansible_user=ubuntu   
    # node2 ansible_host=<node2_ip> ansible_user=ubuntu
      
    [all:vars]
    ansible_ssh_private_key_file=/home/ec2-user/arrow.pem
    # ansible_ssh_private_key_file=/home/ec2-user/<pem file>
```

  # /home/ec2-user/etc/ansible/  altındaki `ansible.cfg` DOSYASINI KULLANMAK YERİNE `/home/ec2-user` İÇERİSİNDE `ansible.cfg` OLUŞTURUYORUZ.

- ```ansible.cfg``` oluştur ve düzenle : ```/home/ec2-user``` İÇERİSİNDE YENİ BİR `ansible.cfg` HAZIRLA.

```bash (pwd : /home/ec2-user)
vim ansible.cfg
    [defaults]
    host_key_checking = False   # her defasında SSH ile bağlanırken sormasın diye 
    inventory = inventory.txt   # managed-node'larımızı yazdığımız dosya, bu dosya adını istediğimiz gibi değiştirebiliriz.
    deprecation_warnings=False  # uyarılar gelmesin diye
    interpreter_python=auto_silent  # uyarılar gelmesin diye
```
  # `arrow.pem` lokalden control node içerisine `/home/ec2-user` kopyalamıştık  ve   `chmod 400 arrow.pem` yapmıştık.

- To confirm the successful installation of Ansible. You can run the following command.

```bash
ansible all -m ping # Bu ad-hoc komutu playbook haline getirip de ping atabiliriz.
```

```yml  (playbook1.yml) (pwd : /home/ec2-user)
---
- name: Test Connectivity   # Play adı 
  hosts: all
  tasks:
    - name: Ping test       # task adı
      ansible.builtin.ping: # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html
```

```bash
ansible-playbook playbook1.yml
```

  ### Farklı farklı playbook'lar oluşturulabilir, örnekler aşağıda:

  # Kurulum gereken işlemlerde playbook içerisinde `become: true` belirtmezseniz komutunuz şu şekilde olmalı `-b` `ansible-playbook -b playbook.yml`

```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
# önceden oluşturduğun bir dosyayı kopyalamak : echo "Hello Everbody, I'm Arrow." > testfile1   # Create a text file named "testfile1"
- name: Copy for linux  
  hosts: linuxserver
  tasks:
   - name: Copy your file to the linuxserver
     ansible.builtin.copy:
       src: /home/ec2-user/testfile1
       dest: /home/ec2-user/testfile1

- name: Copy for ubuntu
  hosts: ubuntuserver
  tasks:
   - name: Copy your file to the ubuntuserver
     ansible.builtin.copy:
       src: /home/ec2-user/testfile1
       dest: /home/ubuntu/testfile1
       mode: u+rw,g-wx,o-rwx

- name: Copy for node1
  hosts: node1
  tasks:
   - name: Copy using inline content
     ansible.builtin.copy:
       content: 'This is content of file2'
       dest: /home/ec2-user/testfile2

   - name: Create a new text file
     ansible.builtin.shell: "echo Hello World > /home/ec2-user/testfile3"
```


```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Apache installation for webservers
  hosts: linuxserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: install the latest version of Apache
     ansible.builtin.yum:
       name: httpd
       state: latest

   - name: start Apache
     ansible.builtin.shell: "service httpd start"

- name: Apache installation for ubuntuserver
  hosts: ubuntuserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: update
     ansible.builtin.shell: "apt update -y"
     
   - name: install the latest version of Apache
     ansible.builtin.apt:
       name: apache2
       state: latest
```

```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Remove Apache from linuxserver
  hosts: linuxserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: Remove Apache
     ansible.builtin.yum:
       name: httpd
       state: absent
       autoremove: yes

- name: Remove Apache from ubuntuserver
  hosts: ubuntuserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: Remove Apache
     ansible.builtin.apt:
       name: apache2
       state: absent
       autoremove: yes
       purge: yes
```

```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Apache installation and configuration for ubuntuserver
  hosts: ubuntuserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: installing apache
     ansible.builtin.apt:
       name: apache2
       state: latest

   - name: index.html
     ansible.builtin.copy:
       content: "<h1>Hello IT World, I am Arrow</h1>"
       dest: /var/www/html/index.html

   - name: restart apache2
     ansible.builtin.service:   # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
       name: apache2
       state: restarted
       enabled: yes

- name: Apache and wget installation for linuxserver           ### 1. VERSİYON
  hosts: linuxserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
    - name: installing httpd and wget
      ansible.builtin.yum:
        pkg: "{{ item }}"     # name ve pkg  aynı , yum module docs içerisinde   aliases
        state: present
      loop:         #  https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html
        - httpd
        - wget

# - name: Apache and wget installation for linuxserver         ### 2. VERSİYON
#   hosts: linuxserver
#   become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
#   tasks:
#     - name: installing httpd and wget
#       ansible.builtin.yum:
#         name:
#           - httpd
#           - wget
#         state: present

# - name: Apache and wget installation for linuxserver       ### 3. VERSİYON
#   hosts: linuxserver
#   become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
#   tasks:
#     - name: installing httpd
#       ansible.builtin.yum:
#         pkg: httpd
#         state: present
#     - name: installing wget
#       ansible.builtin.yum:
#         pkg: wget
#         state: present    

```


```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Remove Apache from ubuntuserver
  hosts: ubuntuserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: Uninstalling Apache
     ansible.builtin.apt:
       name: apache2
       state: absent
       update_cache: yes
       autoremove: yes
       purge: yes

- name: Remove Apache and wget from linuxserver
  hosts: linuxserver
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
   - name: removing apache and wget
     ansible.builtin.yum:
       pkg: "{{ item }}"
       state: absent
     loop:
       - httpd
       - wget
```


```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Create users
  hosts: "*"
  become: true  # sudo yetkisi aldım , komut girerken  -b  yazmama gerek yok
  tasks:
    - name: Create user for REDHAT OS FAMILY
      ansible.builtin.user:       # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
        name: "{{ item }}"
        state: present
      loop:
        - joe
        - matt
        - james
        - oliver
      when: ansible_os_family == "RedHat"     # https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html
                                              # https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html

    - name: Create user for SUSE OS FAMILY
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - david
        - tyler
      when: ansible_os_family == "SUSE"

    - name: Create user for DEBIAN OS FAMILY
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - john
        - aaron
      when: ansible_os_family == "Debian" or ansible_distribution_version == "20.04"
```

```bash
ansible-playbook playbook2.yml
ansible all -m shell -a "tail -4 /etc/passwd | cut -d: -f1 "    # oluşan user'ları gör 
```


# Using handler   (terraform'daki "depend on" gibi)


```yml  (playbook2.yml  örnekleri) (pwd : /home/ec2-user)
---
- name: Copy for linux
  hosts: linuxserver
  vars:
    key: value
  tasks:
   - name: test value
     ansible.builtin.shell:
      cmd: echo {{ key }}

   - name: Copy your file to the linuxserver
     ansible.builtin.copy:
       src: /home/ec2-user/myfile
       dest: /home/ec2-user/bestfile
     notify:
      - Copy

  handlers:     #  BURASININ ÇALIŞMASI İÇİN ÜST TARAFIN `notify` YAPMASI LAZIM
    - name: Copy
      ansible.builtin.shell: "cp /home/ec2-user/bestfile /mnt/testfile"
```


