### Working with dynamic inventory

  # https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html
  # https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options

- 3 Amazon Linux `2023` instances.

    1. control node ----> (SSH PORT 22, HTTP PORT 80)
    2. node1        ----> (SSH PORT 22, HTTP PORT 80)
    3. node2        ----> (SSH PORT 22, HTTP PORT 80)

- Connect to the control node via SSH.
- Install Ansible

```bash (pwd: /home/ec2-user)
sudo dnf update -y
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
pip3 install --user ansible
ansible --version       # Confirm Installation
```
  
- `arrow.pem`i lokalden `/home/ec2-user` dizini içerisine kopyala

```bash (pwd: /home/ec2-user)
chmod 400 arrow.pem
```

  # python ile AWS kaynaklarına ulaşmak için `boto3 , botocore` kurulumuna ihtiyacımız var.

```bash (pwd : /home/ec2-user)
sudo dnf install pip
pip install --user boto3 botocore 
mkdir project
cd project
```

  # control-node'un AWS'den instance bilgilerini alabilmesi için control-node'a  `AmazonEC2FullAccess` role'ü atamalısın, yoksa role oluşturup atamalısın.

- go to AWS Management Consol and select the IAM roles : "create role" then create a role with `AmazonEC2FullAccess`
- go to EC2 instance Dashboard, and select the control-node instance : control-node ==>> Actions - Security - Modify IAM role - Choose role - Update IAM role



- Create file named ```ansible.cfg``` under the the ```/home/ec2-user``` directory.

```bash (pwd : /home/ec2-user)
nano ansible.cfg
```
```ansible.cfg
[defaults]
host_key_checking = False
inventory=/home/ec2-user/project/inventory_aws_ec2.yml
interpreter_python=auto_silent
private_key_file=~/arrow.pem
```


- Farklı bir klasör oluşturup içinde çalışalım (şart değil ama home dizindeki dosyalarla karışmasın, görüntü daha anlaşılır oluyor.)

```bash (pwd : /home/ec2-user)
mkdir project && cd project   # farklı bir klasör oluşturup içinde çalışalım
vi inventory_aws_ec2.yml  # (pwd : /home/ec2-user/project)
```

- Create file named ```inventory_aws_ec2.yml``` in the `/home/ec2-user/project` directory.
  # BU DOSYANIN İSMİ ÖNEMLİ SONU `aws_ec2.yml`  İLE BİTMELİ , YOKSA HATA VERİR.

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
ansible-inventory --graph
```

```
@all:
  |--@aws_ec2:
  |  |--ec2-34-201-69-79.compute-1.amazonaws.com
  |  |--ec2-54-234-17-41.compute-1.amazonaws.com
  |--@ungrouped:
```