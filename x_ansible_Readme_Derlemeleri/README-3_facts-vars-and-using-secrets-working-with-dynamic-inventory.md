### Part 1 - Install Ansible

- Kurulumu Ansible 3. dersteki Terraform dosyaları ile yaptık: 3 Amazon Linux `2023` instances.

    1. control node ----> (SSH PORT 22, HTTP PORT 80)
    2. node1        ----> (SSH PORT 22, HTTP PORT 80)
    3. node2        ----> (SSH PORT 22, HTTP PORT 80)

- Connect to the control node via SSH.
  
- Terraform ile kurulum yaparken `arrow.pem`i <provisioner "file" ile  "/home/ec2-user/${var.mykey}.pem"> lokalden çektik ve <provisioner "remote-exec" ile "chmod 400 ${var.mykey}.pem"> yaptık. Control node'dan ssh ile node1 ve node2'ye bağlanabiliriz.
  
- Lokalde oluşturduğumuz ve bulundurduğum `ansible.cfg` dosyasını terraform ile kurulum yaparken <provisioner "file" ile  "/home/ec2-user/ansible.cfg"> lokalden ec2 içine çektim.

```bash (ansible.cfg)  (pwd: /home/ec2-user)
[defaults]
host_key_checking = False
inventory = inventory.txt
deprecation_warnings=False
interpreter_python=auto_silent
```
  
```bash (pwd: /home/ec2-user)
ansible --version
```

  ## Configure Ansible on the Control Node

  # https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters
  # https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups
  # https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#assigning-a-variable-to-many-machines-group-variables
  # https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#default-groups

- control_node içerisinde ```/etc/ansible/hosts``` dosyasına ekleme yaparak `temel` seviyede `inventory` oluşturacağız, yani dosya içerisinde kontrol edeceğimiz node'ların bilgilerini gireceğiz ki control_node ile ulaşalım ve yönetelim, ayrıca belirli başlıklar altında node'ları gruplandıracağız ki yönetirken/komutları koşarken bize yardımcı olsun. 


```bash (pwd: /home/ec2-user) 
sudo su
cd /etc/ansible
ls          # hosts
vim hosts   # `temel` seviyede `inventory` oluşturuyoruz

  [webservers]
  node1 ansible_host=<node1_ip> ansible_user=ec2-user

  [dbservers]
  node2 ansible_host=<node2_ip> ansible_user=ec2-user

  [all:vars]
  ansible_ssh_private_key_file=/home/ec2-user/arrow.pem
```

### Part 2 - Ansible Facts

  # https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html

- Gathering Facts:

```bash (pwd: /home/ec2-user)
ansible node1 -m setup      # ekrana gelen bilgilere bak, incele
ansible node1 -m setup | grep os
ansible node1 -m setup -a "filter=os_family"    # başına ansible kelimesini koymadan yazmalısın
ansible node1 -m setup -a "filter=hostname"
```

```json
ec2-34-201-69-79.compute-1.amazonaws.com | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.31.20.246"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::88c:37ff:fe8f:3b71"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "08/24/2006",
        "ansible_bios_vendor": "Xen",
        "ansible_bios_version": "4.2.amazon",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "NA",
        "ansible_board_serial": "NA",
```

- create a playbook named ``facts.yml``

```yml  (facts.yml) (pwd: /home/ec2-user)
# why debug module : " var: " ile çekmek istediğimiz facts'leri ekrana yazsın ve görelim diye
- name: show facts
  hosts: all
  tasks:
    - name: print facts
      ansible.builtin.debug:    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html
        var: ansible_facts

    - name: print os_family
      ansible.builtin.debug:
        var: ansible_facts["os_family"]

    - name: print os_family
      ansible.builtin.debug:
        var: ansible_facts["system"]

    - name: print os_family
      ansible.builtin.debug:
        var: ansible_facts["virtualization_type"]  
```

```bash (pwd: /home/ec2-user)
ansible-playbook facts.yml
ansible node1 -m setup # ekrana gelen bilgilere bak, incele
```

- create a playbook named ``ipaddress.yml``

```yml  (ipaddress.yml) (pwd: /home/ec2-user)
# neden debug modülü kullandık : " msg: " ile çekmek istediğimiz facts'leri ekrana yazsın ve görelim diye
- hosts: all
  tasks:
  - name: show IP address
    ansible.builtin.debug:
      msg: >
       This host uses IP address {{ ansible_facts.default_ipv4.address }}

  - name: show distribution
    ansible.builtin.debug:
      msg: This host uses distribution {{ ansible_facts.distribution }}

  - name: show policyvers
    ansible.builtin.debug:
      msg: This host uses policyvers {{ ansible_facts.selinux.policyvers }}

  - name: show ansible_facts as a variable
    ansible.builtin.debug:
      var: ansible_facts["selinux"]
  
  - name: show ansible_facts as a variable
    ansible.builtin.debug:
      var: ansible_facts["selinux"]["policyvers"]
  
  - name: show ansible_facts as a variable
    ansible.builtin.debug:
      var: ansible_facts.selinux.policyvers
```

```bash (pwd: /home/ec2-user)
ansible-playbook ipaddress.yml 
```

### Part 3 - Working with sensitive data

- Create encypted variables using ``ansible-vault`` command

```bash (pwd: /home/ec2-user)
ansible-vault
ansible-vault create secret.yml # harhangi bir .yaml  dosyası olur, "secret" isminin bir anlamı yok
New Vault password: xxxx  # 2 defa şifre ister    123456
Confirm Nev Vault password: xxxx    #             123456
```

```yml  (secret.yml) (pwd: /home/ec2-user)
username: arrow
password: 99abcd
```

- look at the content

```bash (pwd: /home/ec2-user)
cat secret.yml    # şifrelemiş
ansible-vault
ansible-vault view secret.yaml    # bu komutla içerisini görebiliyorum.
        Vault password: xxxx  # şifre ister     123456
        username: arrow
        password: 99abcd
```

```yml  (secret.yml içeriği) (pwd: /home/ec2-user)
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```

- how to use it:

  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html

- create a file named ``create-user.yml``

```bash (pwd: /home/ec2-user)
nano create-user.yml
```

```yml  (create-user.yml) (pwd: /home/ec2-user)
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret.yml    # bu dosyada => username: arrow  ,  password: 99abcd
  tasks:
    - name: creating user
      ansible.builtin.user:
        name: "{{ username }}"        # `username` : biz yukarıda secret.yaml içinde bu şekilde tanımladık, kelimeyi rastgele yazmıyoruz
        password: "{{ password }}"    # `password` : biz yukarıda secret.yaml içinde bu şekilde tanımladık, kelimeyi rastgele yazmıyoruz
```

```bash (pwd: /home/ec2-user)
ansible-playbook create-user.yml  # OLMADI : ERROR! Attempting to decrypt but no vault secrets found !
# Çünkü bu komutu çalıştırırken ``--ask-vault-pass``  veya   ``--ask-vault-password``   ifadesini eklemelisin. 

ansible-playbook --ask-vault-pass create-user.yml
    Vault password: xxxx  # şifre ister     123456
# [WARNING]: The input password appears not to have been hashed. The 'password' argument must be encrypted for this module to work properly.
# Bu uyarıya göre şifrem ŞİFRELENMEDİ.

ansible all -b -m command -a "grep arrow /etc/shadow"   # kulanıcı oluştu:

      PLAY RECAP ******************************************************************************************
      node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
      node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
      ```
      node1 | CHANGED | rc=0 >>
      arrow:99abcd:18569:0:99999:7:::
```

- Create another encypted variables using "ansible-vault" command but this time use `SHA (Secure Hash Algorithm)` for your password:

```bash (pwd: /home/ec2-user)
ansible-vault create secret-1.yml
        New Vault password: xxxx          # 123
        Confirm Nev Vault password: xxxx  # 123
```

```yml  (secret-1.yml)  (pwd: /home/ec2-user)
username: arrow
mysifre: 14abcd
```

- look at the content

```bash (pwd: /home/ec2-user)
cat secret-1.yml
```
```
33663233353162643530353634323061613431366332373334373066353263353864643630656338
6165373734333563393162333762386132333665353863610a303130346362343038646139613632
62633438623265656330326435646366363137373333613463313138333765366466663934646436
3833386437376337650a636339313535323264626365303031366534363039383935333133306264
61303433636266636331633734626336643466643735623135633361656131316463
```
- how to use it:
  # https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html
- create a file named ``create-user-1.yml``

```bash (pwd: /home/ec2-user)
nano create-user-1.yml
```

```yml  (create-user-1.yml) (pwd: /home/ec2-user)
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret-1.yml
  tasks:
    - name: creating user
      ansible.builtin.user:
        name: "{{ username }}"
        password: "{{ mysifre | password_hash ('sha512') }}" # https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html
``` 

```bash (pwd: /home/ec2-user)
ansible-playbook --ask-vault-pass create-user-1.yml
    Vault password: xxxx          # 123

    PLAY RECAP ******************************************************************************************
    node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    node2                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- to verrify it:

```bash (pwd: /home/ec2-user)
ansible all -b -m command -a "grep arrow /etc/shadow"

    node1 | CHANGED | rc=0 >>
    arrow:#665fffgkg6&fkg689##2£6466?%^^+&%+:18569:0:99999:7:::     ### ŞİFRELENMİŞ
```

```bash (pwd: /home/ec2-user)
ansible-playbook
        --vault-password-file   # değişkeni bir file'dan da çekebiliriz.
 
ansible-vault create secret-2.yml
        New Vault password: xxxx          # 12345
        Confirm Nev Vault password: xxxx  # 12345
```

```yml    (secret-2.yml)  (pwd: /home/ec2-user)
username: levent
mysifre: 987654
```

```bash (pwd: /home/ec2-user)
ansible-vault view secret-2.yml
          Vault password:   # 12345
          username: levent
          mysifre: 987654

ansible-vault decrypt secret-2.yml
    Vault password:   # 12345
    Decryption successful

ansible-vault encrypt secret-2.yml
        New Vault password: xxxx          # 123
        Confirm Nev Vault password: xxxx  # 123

ansible-vault edit secret-2.yml
          Vault password: xxxx          # 123
          username: levent
          mysifre: 112233   #  şifreyi değiştirdim.

ansible-vault view secret-2.yml
          username: levent
          mysifre: 112233   #  yeni şifreyi gir

ansible-vault rekey secret-2.yml
          Vault password: xxxx          # 123
          New Vault password: xxxx          # 123abc
          Confirm Nev Vault password: xxxx  # 123abc
```

- vault şifresini dosyadan çekeceğim, yani ekranda bana sormayacak

```bash (pwd: /home/ec2-user)
code mypassword.yml         # Aa12345     # vault şifresini bu dosyadan çekeceğim, yani ekranda bana sormayacak.
                            # ansible-playbook --vault-password-file mypassword.yml  ==>> komutu şifreyi bu dosyadan çeker
```
```yml    (mypassword.yml)  (pwd: /home/ec2-user)
Aa12345       # vault şifresini bu dosyadan çekeceğim, yani ekranda bana sormayacak.
              # ansible-playbook --vault-password-file mypassword.yml  ==>> komutu şifreyi bu dosyadan çeker
```

```yml    (secret-2.yml)  (pwd: /home/ec2-user)
username: levent
mysifre: Aa12345    # Bu şifrenin bir hükmü yok , şifreyi "ansible-playbook --vault-password-file mypassword.yml" ==>> mypassword.yml'den çeker.
```

```bash (pwd: /home/ec2-user)
nano create-user-33.yml
```

```yml  (create-user-33.yml)  (pwd: /home/ec2-user)
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret-2.yml  # bunun içerisinde ne kadar değişken varsa çeker
  tasks:
    - name: creating user
      ansible.builtin.user:
        name: "{{ username }}"
        password: "{{ mysifre }}"
```

```bash
ansible-playbook --vault-password-file mypassword.yml
ansible all -b -m command -a "grep levent /etc/shadow"
# AMA BU SEFERDE mypassword.yml içerisinde benim vault password görünüyor, yine EMNİYETSİZ.
```

  ###### vault password'ümü AWS'de saklayarak daha güvenli olmasını sağlamak:

  ### AWS `Parameter Store`

```t
create parameter
Name        : ansible-lesson
Description : ansible lesson
Tier        : Standart
Type        : String
Data type   : text
Value       : Aa12345    # mypassword.yml içerisindeki vault password'ümü AWS'de saklayacağım        # Aa12345
create parameter
```

- control-node'a role atamalısın, yoksa role oluşturup atamalısın. "Admin"  veya  "AmazonEC2FullAccess" .
    Actions - Security - Modify IAM role - Choose role - Update IAM role

```bash (pwd: /home/ec2-user)
aws --region=us-east-1 ssm get-parameters --name "ansible-lesson" --query "Parameters[*].{Value:Value}" --output text
       Aa12345      #  gitti ve AWS'den şifreyi aldı.

###         veya      ==>> Bu komutu bir script ile kullanabilirim
```
  ### Bu komutu bir script ile kullanalım:

```bash (pwd: /home/ec2-user)
code mypassword.yml
```

```yml  (mypassword.yml)
# İÇERİĞİNDE ŞİFRE OLMAYACAK, İÇERİSİNDE SCRİPT OLACAK ŞEKİLDE YENİDEN DÜZENLE
#!/bin/bash
aws --region=us-east-1 ssm get-parameters --name "ansible-lesson" --query "Parameters[*].{Value:Value}" --output text
```

```yml    (secret-2.yml)
username: levent
mysifre: Aa12345 # Bunun olmasına gerek yok , şifreyi AWS'de sakladım, komut girerken belirteceğim
```

```yml  (create-user-33.yml)
- name: create a user
  hosts: all
  become: true
  vars_files:
    - secret-2.yml
  tasks:
    - name: creating user
      ansible.builtin.user:
        name: "{{ username }}"
        password: "{{ mysifre }}"
```

```bash
chmod +x mypassword.yml   # artık scritp yazdığım için içine, çalıştırmak için bu komut girmelisin
ls -l
ansible-playbook --vault-password-file mypassword.yml create-user-33.yml  # script çalıştı, şifreyi AWS'den çekti.
```


### Debug kullanımı:

```bash
code myfile13
      myuser: orhun
      mypassword: abc123
```

```yml  (create-user-debug.yml)
# neden debug modülü kullandık : " msg: " ile "vars_files" içerisindeki "myuser" değişkeni  ekrana yazsın ve görelim diye
- name: create a user
  hosts: all
  become: true
  vars_files:
    - myfile13
  tasks:
    - name: debug using forms
      ansible.builtin.debug:
        msg: My name is {{ myuser  }}
        # msg: My password is {{ mypassword | upper}} #https://jinja.palletsprojects.com/en/3.0.x/templates/#builtin-filters    
```

```bash
ansible-playbook create-user-debug.yml
```



### Part 4 - Working with Dynamic Inventory Using EC2 Plugin

  # Bu aşağıdaki statik inventory hazırlama, dinamik bundan sonra anlatılıyor.

- Create a file named ```inventory.txt``` under the the ```/home/ec2-user``` directory.

  # Terraform `provisioner "remote-exec"  "echo [webservers] >> inventory.txt" `  ve devamındaki komutlarla `inventory.txt ` `/home/ec2-user` içinde oluştu. Eğer oluşmasaydı biz şu şekilde yapardık:

```bash (pwd : /home/ec2-user)
nano inventory.txt

  [webservers]
  # node1  ansible_host=<YOUR-WEB-SERVER-IP> ansible_ssh_private_key_file=~/<YOUR-PEM-FILE> ansible_user=ec2-user
  node1  ansible_host=34.201.69.79 ansible_ssh_private_key_file=~/arrow.pem ansible_user=ec2-user

  [dbservers]
  # node2   ansible_host=<YOUR-DB-SERVER-IP> ansible_ssh_private_key_file=~/<YOUR-PEM-FILE> ansible_user=ec2-user
  node2   ansible_host=35.187.45.88 ansible_ssh_private_key_file=~/arrow.pem ansible_user=ec2-user
```

- Create file named ```ansible.cfg``` under the the ```/home/ec2-user``` directory.
  # Terraform <provisioner "file" { source = "./ansible.cfg" destination = "/home/ec2-user/ansible.cfg"> komutla `ansible.cfg` `/home/ec2-user` içinde oluştu. Eğer oluşmasaydı biz şu şekilde yapardık:

```bash (pwd : /home/ec2-user)
nano ansible.cfg
```

```ansible.cfg
[defaults]
host_key_checking = False
inventory=/home/ec2-user/inventory.txt  # inventory.txt adresini giriyorum ki host bilgilerini ordan çeksin
interpreter_python=auto_silent
private_key_file=~/arrow.pem
```

- Create a file named ```ping-playbook.yml``` and paste the content below.

```bash (pwd : /home/ec2-user)
nano ping-playbook.yml
```

```yml
- name: ping them all
  hosts: all
  tasks:
    - name: pinging
      ansible.builtin.ping:
```

```bash (pwd : /home/ec2-user)
ansible-playbook ping-playbook.yml
```

### Working with dynamic inventory

  # https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html
  # https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options


  ## python ile AWS kaynaklarına ulaşmak için `boto3 , botocore` ihtiyacımız var.

```bash (pwd : /home/ec2-user)
# sudo yum install pip
pip install --user boto3 botocore   # python ile AWS kaynaklarına ulaşmak için `boto3 , botocore` ihtiyacımız var.
mkdir project
cd project
```

  ## control-node'un AWS'den instance bilgilerini alabilmesi için control-node'a  `AmazonEC2FullAccess` role'ü atamalısın, yoksa role oluşturup atamalısın.

- go to AWS Management Consol and select the IAM roles : "create role" then create a role with `AmazonEC2FullAccess`
- go to EC2 instance Dashboard, and select the control-node instance : control-node ==>> Actions - Security - Modify IAM role - Choose role - Update IAM role


- Create another file named ```inventory_aws_ec2.yml``` in the `/home/ec2-user/project` directory.

  ## BU DOSYANIN İSMİ ÖNEMLİ SONU `aws_ec2.yml`  İLE BİTMELİ , YOKSA HATA VERİR.

```bash (pwd : /home/ec2-user)
mkdir project && cd project   # farklı bir klasör oluşturup içinde çalışalım
nano inventory_aws_ec2.yml  # (pwd : /home/ec2-user/project)
```

```yml   (inventory_aws_ec2.yml) (pwd : /home/ec2-user/project)
plugin: aws_ec2   # https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html
regions:
  - "us-east-1"
keyed_groups:     # https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options
  - key: tags.Name
compose:
  ansible_host: public_ip_address
```

  # `inventory_aws_ec2.yml` dosyasını Çalıştırmak istiyorsan, onu gören dizinde olmalısın !!!

```bash (pwd : /home/ec2-user/project)
ansible-inventory --graph                             # static olan "inventory.txt" gösterir
ansible-inventory -i inventory_aws_ec2.yml --graph    # dinamik olan "inventory_aws_ec2.yml" gösterir
#  her defasında `-i inventory_aws_ec2.yml` yazmak istemiyorsan ==>> " /home/ec2-user " dizinindeki ansible.cg GÜNCELLE  !!!!
```
  #  Her defasında `-i inventory_aws_ec2.yml` yazmak istemiyorsan ==>> `ansible.cg` GÜNCELLE  !!!!
  
```ansible.cfg
[defaults]
host_key_checking = False
inventory=/home/ec2-user/project/inventory_aws_ec2.yml    #### BURASI DEĞİŞTİ
interpreter_python=auto_silent
private_key_file=~/arrow.pem
```

```bash
ansible-inventory --graph   # ARTIK DİNAMİK "inventory_aws_ec2.yml" gösterir, her defasında `-i inventory_aws_ec2.yml` yazmana gerek yok.
```


### control-node ve host'larımıza tag'ler atayabilelim ki farklı şekilde listeleme yapabilelim  `Tag`

  # AWS Console'a git ve control-node dahil 3 EC2'ya da tag ata ==>> `Key: environment , Value: dev`

  # `filters -->> tag`   SADECE BU TAG FİLTRELEMESİNE UYANLARI GÖSTERİR.

  # `keyed_groups -->> name`  FİLTRELENENLERİ GÖSTERİRKEN GRUPLANDIRARAK GÖSTERİR.

```yml   (inventory_aws_ec2.yml) (pwd : /home/ec2-user/project)
plugin: aws_ec2   # https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html
regions:
  - "us-east-1"
filters:
  tag:environment: dev  # AWS Console'a git ve control-node dahil 3 EC2'ya da tag ata ==>> `Key: environment , Value: dev`
keyed_groups:     # https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options
  - key: tags.Name
  - key: tags.host   # AWS Console'a git ve 1 managed EC2'ya da tag ata ==>> `Key: host , Value: managed`
compose:
  ansible_host: public_ip_address
```

```bash (pwd : /home/ec2-user/project)
ansible-inventory --graph   # ARTIK SADECE tag atadıklarımız gelir.
```

```
@all:
  |--@ungrouped:
  |--@aws_ec2:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |--@_control_host:
  |  |--ec2-54-186-45-99.compute-1.amazonaws.com
  |--@_managed:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com 
```

  # SADECE `managed` olanlara ping atmak için:

```bash (pwd : /home/ec2-user/project)
ansible _managed -m ping      # SADECE `managed` olanlara ping atmak için
```


```yml  (inventory_aws_ec2.yml) (pwd : /home/ec2-user/project)
plugin: aws_ec2   
regions:
  - "us-east-1"
filters:
  tag:environment: dev 
keyed_groups:
  - key: tags.Name
  - key: tags.environment
  # - key: tags.host
compose:
  ansible_host: public_ip_address
```

  # SADECE `dev` olanlara ping atmak için:

```bash (pwd : /home/ec2-user/project)
ansible-inventory --graph
ansible _dev -m ping      # SADECE `dev` olanlara ping atmak için
```


```yml  (inventory_aws_ec2.yml)
plugin: aws_ec2   
regions:
  - "us-east-1"
filters:
  tag:environment: dev 
keyed_groups:
  - key: tags.environment
    prefix: arrow
    # seperator: un
compose:
  ansible_host: public_ip_address
```

```bash (pwd : /home/ec2-user/project)
ansible-inventory --graph
```
@all:
  |--@ungrouped:
  |--@aws_ec2:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |  |--ec2-54-186-45-99.compute-1.amazonaws.com
  |--@arrow_dev:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |  |--ec2-54-186-45-99.compute-1.amazonaws.com


```yml  (inventory_aws_ec2.yml)
plugin: aws_ec2   
regions:
  - "us-east-1"
filters:
  instance-state-name: running 
keyed_groups:
  - key: tags.environment
    prefix: arrow
    seperator: un    # ==>> burayı açınca -->> arrow_dev -->> arrowundev
compose:
  ansible_host: public_ip_address
```

```bash (pwd : /home/ec2-user/project)
ansible-inventory --graph
```

@all:
  |--@ungrouped:
  |--@aws_ec2:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |  |--ec2-54-186-45-99.compute-1.amazonaws.com
  |--@arrowundev:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |  |--ec2-54-186-45-99.compute-1.amazonaws.com



  # neden debug modülü kullandık : " msg: " ile çekmek istediğimiz `compose` başlığı altındaki `facts`'leri ekrana yazsın ve görelim diye

```yml  (inventory_aws_ec2.yml) (pwd : /home/ec2-user/project)
plugin: aws_ec2
regions:
  - "us-east-1"
filters:
  instance-state-name: running
  tag:environment: dev
keyed_groups:
  - prefix: arrow
    separator: un
    key: tags.environment
compose:
  ansible_host: public_ip_address
  foo: private_ip_address
  boo: instance_id
  key: key_name
```

```yml    (test.yml)  (pwd : /home/ec2-user/project)
- name: test
  hosts: all
  become: true
  tasks:
    - debug:
        msg: "My private ip is {{ foo }}"     # My private ip is "private_ip_address  ne ise o"
    - debug:
        msg: "My instance_id is {{ boo }}"    # My instance_id is "instance_id  ne ise o"
    - debug:
        msg: "My key pem is {{ key }}"        # My key pem is "key_name  ne ise o"
    - debug:
        msg: "My host is {{ ansible_host }}"  # My host is "public_ip_address  ne ise o"
```

```bash (pwd : /home/ec2-user/project)
ansible-playbook test.yml
```

--------------------------------------------------------------------------------------------------------------------------
######  BUNU DİREKT OLARAK LOKAL BİLGİSAYARDA KULLANABİLİRİM.

```yml  (inventory_aws_ec2.yml)
plugin: aws_ec2
regions:
  - "us-east-1"
filters:
  instance-state-name: running
  tag:environment: dev
keyed_groups:
  - prefix: arrow
    separator: un
    key: tags.environment
compose:
  ansible_host: public_ip_address
```
```bash
ansible all -i inventory_aws_ec2.yml --key-file ~/.ssh/arrow.pem -u ec2-user -m ping
```


- To make sure that all our hosts are reachable with dynamic inventory, we will run various ad-hoc commands that use the ping module.

```bash
ansible all -m ping --key-file "~/<pem file>"
```

- create a playbook name ``user.yml``

```yml
---
- name: create a user using a variable
  hosts: all
  become: true
  vars:
    user: lisa
    ansible_ssh_private_key_file: "/home/ec2-user/<pem file>"
  tasks:
    - name: create a user {{ user }}
      ansible.builtin.user:
        name: "{{ user }}"
```
- run the playbook

```bash
ansible-playbook user.yml -i inventory_aws_ec2.yml
```

```bash
ansible all -a "tail -2 /etc/passwd"
```

### .bashrc  dosyasına yazılacaklar

```bash
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "  # sarı-mavi-sarı
export PS1="\[\e[1;34m\]\u\[\e[38;5;214m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "	# sarı-mavi-sarı  (SARI DAHA KOYU)
sudo hostnamectl set-hostname "Control-node"
bash
```
----------------------------
```bash
pip install ansible-lint
```