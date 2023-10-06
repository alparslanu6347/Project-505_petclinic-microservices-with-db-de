### Install Docker Engine on CentOS

# https://docs.docker.com/engine/install/centos/

```bash
# Uninstall old versions
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# Install using the rpm repository , Set up the repository
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Engine (Latest)    # spesifik bir versiyon kurmak istiyorsan ilgili sayfaya bakmalısın
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl start docker # Start Docker
sudo docker run hello-world # Verify that the Docker Engine installation is successful by running the hello-world image.
```


### Amazon Linux 2 (ami-026dea5602e368e96) EC2 Instance Docker Kurulumu

```bash
sudo hostnamectl set-hostname "docker-arrow"
bash
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "
```

```bash
sudo yum update -y      # Update the installed packages and package cache on your instance.
# "Amazon Linux 2023" için de bu install komutu çalışır.
sudo dnf install docker # Install the most recent Docker Community Edition package. 
      # sudo yum install docker     ###   EN ESKİ KOMUT
sudo systemctl start docker     # Start docker service.
sudo systemctl enable docker    # Enable docker service so that docker service can restart automatically after reboots.
sudo systemctl status docker
usermod -a -G docker ec2-user   # `ec2-user` isimli user'ı docker grubuna dahil ederek komutlarda artık sudo kullanmama gerek kalmayacak.
newgrp docker                   # 


##### install docker-compose
sudo curl -SL https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose # v2.16.0
# sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" \ -o /usr/local/bin/docker-compose   # v1.29.2       Eski versiyon
sudo chmod +x /usr/local/bin/docker-compose
```

```bash
docker version
sudo systemctl status docker    # Check if the docker service is up and running.
```

# Docker kurulu makinada komutlar


```bash
docker help
docker run --help
docker run hello-world # (docker hub repodan çekti)
docker run -it ubuntu bash  # container ayağa kaldırır ve içerisine "ubuntu" image yükler, ve içine gireriz
cat /etc/os-release     # işletim sistemini öğrenme , container içerisinde koşulur
docker run -it --name arrow ubuntu bash   # "arrow" isimli bir container ayağa kaldırır ve içerisine "ubuntu" image yükler, ve içine gireriz

docker ps --help    
docker ps           # çalışan container'ları gösterir
docker container ls # çalışan container'ları gösterir

docker run ubuntu ls    # "ubuntu" ismindeki image'den docker'ın belirleyeceği bir isimle container ayağa kaldırır ve içerisine girmeden içerisinde komut koşturur.
docker container run ubuntu ls # yukarıdaki komutla aynı işi yapar.

docker ps -a    # process statusu ne olursa olsun tüm containerları gösterir
docker container --help
docker container ls -a    # process statusu ne olursa olsun tüm containerları gösterir

docker start --help
docker start aca2c1676e37   # Stop durumundaki aca2c1676e37 Id'li container'ı  start etmek için
docker rm --help
docker stop levent  # levent isimli container'ı STOP'a çek
docker rm levent        # "STOP'taki" levent isimli container'ı sil
docker rm -f levent     # "ÇALIŞAN" levent isimli container'ı sil
docker container prune  # "STOP'taki" TÜM containerlar'ı sil

docker container ps -aq     # veya   `docker ps -aq`  tüm container'ların (çalışan-çalışmayan) SADECE ID'sini GÖSTERİR
docker rm -f $(docker ps -aq)   # (çalışan-çalışmayan tüm containerları siler)

docker image --help
docker image ls # image'ları listeler
docker image inspect ubuntu # image bilgilerini incelemek
docker image prune  # tüm image'ları siler
docker image prune -af  # tum imajlari silmek icin

docker volume --help
docker volume prune # tüm volume'ları siler

docker container ps -aq    # veya docker ps -aq   tüm container'ların SADECE ID'sini GÖSTERİR
docker rm -f $(docker ps -aq)   # Çalışan/Çalışmayan tüm containerları siler

docker run --name arrow -d nginx    # nginx isimli image'den arrow isimli container ayağa kaldır ve (-d : detached ) arka planda çalıştır
docker exec arrow ls    # "ÇALIŞMAKTA" olan arrow isimli container içine girmeden bağlan ve içinde komut çalıştır
docker exec -it arrow bash  # "ÇALIŞMAKTA" olan arrow isimli container'a bağlan, içine "GİR" ve içinde komut çalıştır

docker container run -dit --name levent alpine ash # alpine isimli image'den levent isimli container ayağa kaldır, (-d : detached ) arka planda çalıştır -->> çalışır durumdadır. , d olmadan sadece -it yazsaydık bağlanır içine girerdik ve `exit` yapsan da çalışır konumda kalır.

docker container run -dit --name levent alpine ash    # `-it` KULLANARAK BAĞLANDIYSAN `exit` YAPSAN DA container ÇALIŞMAYA DEVAM EDER


docker run -d  --name arrow-database --network india-net -e MYSQL_ROOT_PASSWORD=R1234r -e MYSQL_DATABASE=todo_db -e MYSQL_USER=arrow -e MYSQL_PASSWORD=Arrow_1  mysql:5.7 #   mysql image'ndan container ayağa kaldırırken mysql database içerisinde kullanılan variables'ları yazarken -e flagini kullanıyoruz
```

------------------------------------------------------------------------------------------------------
#####   VOLUME  #####

```bash
docker volume create arrow-vol  # Create a volume named `arrow-vol`.
docker volume ls    # List the volumes available in Docker, should see local volume `arrow-vol` in the list.
docker volume inspect arrow-vol  # volume bilgilerini gösterir. !!! Mountpoint !!!
                                 # Note the mount point: `/var/lib/docker/volumes/arrow-vol/_data`.
sudo ls -al  /var/lib/docker/volumes/arrow-vol/_data    # List all files/folders under the mount point of the volume `arrow-vol`, should see nothing listed.
docker run -it --name levent -v arrow-vol:/arrow alpine ash # image: alpine , container: levent , volume: arrow-vol , /arrow: mountpoint in the container , -v: volume flag ,
ls  # levent isimli container içine girdim ve arrow klasörü gelmiş.
docker run -it --name levent -v arrow-vol:/arrow:ro ubuntu bash # Bu container'a read only:ro özelliği verdik, içerisindeki bilgileri sadece okuyabileceğiz, yeni bir dosya oluşturma veya dosya içerinde değişiklik YAPAMAYIZ.
```

   ###  Bind Mounts ###

```bash
# Attach the `nginx-default` container, show the index.html in the /usr/share/nginx/html directory.
docker run -d --name nginx-default -p 80:80  nginx  # image: nginx , container: nginx-default , -p: port flag
docker exec -it nginx-default bash
root@4a1c7e5f394a:/# cd /usr/share/nginx/html
root@4a1c7e5f394a:/usr/share/nginx/html# ls
50x.html  index.html
root@4a1c7e5f394a:/usr/share/nginx/html# cat index.html
```
http://<public-ip>:80

```bash (pwd : /home/ec2-user)
# Create a folder named  webpage, and an index.html file.
mkdir webpage && cd webpag
pwd # /home/ec2-user/webpage
echo "<h1>Welcome to Arrow Web Page</h1>" > index.html
```

```bash
# MECBURİ BAĞLANTI YAPIYOR, VOLUME'U BAŞKA BİR DİZİNE BAĞLAR GİBİ =>KLASÖRÜ VOLUME GİBİ ALGILIYOR, KLASÖR İÇİNDEKİ BİLGİLERİ BAĞLADIĞIM YERİN ÜZERİNE YAZIYOR

docker run -d --name nginx-new -p 8080:80 -v /home/ec2-user/webpage:/usr/share/nginx/html nginx # image: nginx , container: nginx-new , volume: /home/ec2-user/webpage , /usr/share/nginx/html: mountpoint in the container , -v: volume flag , -p: port flag
```
http://<public-ip>:8080

```bash
docker exec -it nginx-new bash
root@a7e3d276a147:/# cd usr/share/nginx/html
root@a7e3d276a147:/usr/share/nginx/html# ls 
index.html
root@a7e3d276a147:/usr/share/nginx/html# cat index.html 
<h1>Welcome to Arrow Web Page</h1>
```

------------------------------------------------------------------------------------------------------
#####   NETWORK  #####

```bash
docker network ls # List all networks available in Docker
docker network inspect bridge # Show the details of `bridge` network
docker network create --help
docker network create --driver bridge myson-leventnet # Create a bridge network `myson-leventnet`.
docker network rm myson-leventnet   # Delete `myson-leventnet` network

# nginx  ==>> portu 80 , http://ec2-18-232-70-124.compute-1.amazonaws.com:8080   # Lokalde (http://127.0.0.1:8080)
docker container run --rm -d -p 8080:80 --name arrow nginx ## --rm  container ile işimiz bitince  stopa çekince otomatik siler.
docker container run --rm -dit --network host --name arrow nginx  # host => host makinanın network'unu kullanırız   ,  -p 8080:80  yazmadık

```

-----------------------------------------------------------------------------------------------------
#####   Container LOG KAYITLARI ve KISITLAMALAR  #####

```bash
docker container run --name ng -d -p 80:80 nginx
# docker ps ,  docker exec -it ng sh  , ls  , cd /var/log/
docker logs ng      #  ng container içindeki log kayıtları
docker logs --details ng  # detaylı olarak ng container içindeki log kayıtları
docker logs -t ng  # -t  zaman
docker logs --help
docker logs --until 2023-03-18T11:45:15.416919612 ng
docker logs --since 2m ng  # son 2 dakika
docker logs --tail 5 ng  # son 5 satır log kayıtları getir
docker stats ng
docker container run --name ng --memory=200m -d nginx # container memory limitle
docker container run --name ng -cpus=”1.5” –cpuset-cpus=”0,3” -d nginx # container cpu limitle
docker container run --name ng --cpus=1 -d nginx # container cpu limitle


docker run -it --name mycontainer ubuntu bash  # yeni bir container
mkdir added-folder
rmdir opt
cd mnt
touch test.txt
exit
docker diff mycontainer

docker image prune -af  # tum imajlari silmek icin
```



-----------------------------------------------------------------------------------------------------
#####   Docker Hub  #####

```bash
# Defaults to ubuntu:latest
docker image pull ubuntu
docker image inspect ubuntu
docker image pull ubuntu:18.04
docker image inspect ubuntu:18.04
docker search ubuntu
docker image ls
```

   ### Dockerfile ile image oluşturma ve Docker Hub hesabımıza push etme :

```bash
# Create application code and save it to file, and name it `welcome.py`
echo '
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "<h1>Welcome to Arrow Docker Class</h1>"
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
' > welcome.py
```

```Dockerfile     (pwd : docker-lesson/)
# Create a Dockerfile and name it `Dockerfile`.
FROM ubuntu
RUN apt-get update -y
RUN apt-get install python3 -y
RUN apt-get install python3-pip -y
RUN pip3 install flask
COPY . /app   ## "." => tüm dosyaları root dizininde "/app" dizinine kopyala  ## COPY welcome.py /app     
WORKDIR /app
CMD python3 ./welcome.py
```
   # 2 Dockerfile'dan istediğini kullanabilirsin, alltaki daha küçük boyutludur.

```Dockerfile     (pwd : docker-lesson/)
# Create a Dockerfile and name it `Dockerfile`.
FROM python:alpine
RUN pip install flask
COPY . /app
WORKDIR /app
EXPOSE 80
CMD python ./welcome.py
```

```bash
docker build docker-lesson/   # docker-lesson/ : Dockerfile'ın içerisinde bulunduğu dizin
docker build .    # . : Dockerfile'ın bulunduğu dizinde build edeceksen
docker build -t "alparslanu6347/flask-app:1.0" .      # alparslanu6347 : Docker Hub hesap/kullanıcı adı , flask-app:1.0 : image'ime vereceğim tag/isim

docker run -d --name arrow_container -p 80:80 alparslanu6347/flask-app:1.0

docker login  # Login in to Docker with credentials  -->> kullanıcı adı ve şifre soracak
docker push alparslanu6347/flask-app:1.0  # Push newly built image to Docker Hub

docker image tag alparslanu6347/flask-app:2.0 alparslanu6347/flask-app:latest # We can also tag the same image with different tags.
```

-----------------------------------------------------------------------------------------------------
#####   Docker Compose  #####

```bash
##### install docker-compose
sudo curl -SL https://github.com/docker/compose/releases/download/v2.16.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose # v2.16.0
# sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" \ -o /usr/local/bin/docker-compose   # v1.29.2       Eski versiyon
sudo chmod +x /usr/local/bin/docker-compose

docker compose --version      # eski versiyon "version 1.29.2" ==>> docker-compose --version
```

- https://hub.docker.com/_/mysql
- https://docs.docker.com/compose/compose-file/compose-versioning/
- https://docs.docker.com/compose/profiles/
- https://github.com/docker/awesome-compose
- https://docs.docker.com/compose/gettingstarted/
- https://docs.docker.com/develop/develop-images/dockerfile_best-practices/


   ### docker-lesson klasörü içerisinde uygulama ile ilgili dosyalarımı oluşturalım -> docker-compose örneği: ÖRNEK-1
   - app.py
   - requirements.txt
   - Dockerfile
   - docker-compose.yml

```python  (app.py) (pwd : docker-lesson)
# Create a file called `app.py` in your `docker-lesson` folder and paste the following python code in it. In this example, the application uses the Flask framework and maintains a hit counter in Redis, and  `redis` is the hostname of the `Redis container` on the application’s network. We use the default port for Redis, `6379`.
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

```bash (requirements.txt)    (pwd : docker-lesson)
# Create another file called `requirements.txt` in your docker-lesson folder, add `flask` and `redis` as package list.
flask
redis
```

```bash (Dockerfile)    (pwd : docker-lesson)
# Create a Dockerfile which builds a Docker image
FROM python:3.7-alpine  # Build an image starting with the Python 3.7 image.
WORKDIR /code     # Set the working directory to `/code`, böyle bir dizin yoksa da oluşturur, oluşturduğu dizin içinde komut çalışır
ENV FLASK_APP app.py    # Set environment variables used by the flask command.
ENV FLASK_RUN_HOST 0.0.0.0    # Set environment variables used by the flask command.
RUN apk add --no-cache gcc musl-dev linux-headers     # Install gcc and other dependencies
COPY requirements.txt requirements.txt    # Copy `requirements.txt`
RUN pip install -r requirements.txt       # install the Python dependencies.
EXPOSE 5000 # Add metadata to the image to describe that the container is listening on port 5000 : SADECE BİLGİ AMAÇLI BİR SATIRDIR.
COPY . .    # Copy the current directory `.` in the project to the workdir `.` in the image. Bulunduğun dizini (docker-lesson) komutun çalışcağı dizine yani /code dizinine kopyala
CMD ["flask", "run"]    # Set the default command for the container to flask run.
```

```bash    (docker-compose.yml) (pwd : docker-lesson)
# Create a file called `docker-compose.yml` in your docker-lesson folder
version: "3"
services:
  web:
    build: .      # bulunduğun dizindeki (docker-lesson) Dockerfile ile image oluşturacak
    ports:
      - "5000:5000"
  redis:        # app.py içinde ismi "redis" ==>> cache = redis.Redis(host='redis', port=6379)
    image: "redis:alpine"     # Docker Hub'dan redis image pull edecek ve kullanacak
```

```bash     (pwd : docker-lesson)
# Build and run your app with `Docker Compose`
docker-compose up
docker network ls
docker image ls
docker ps

docker-compose help
docker-compose ps
docker-compose images
docker-compose config   # Run `docker-compose config` command to see the configuration of docker-compose file

docker-compose down

- http://127.0.0.1:5000  ==>> lokalden görürsün  veya alttaki
- http://`ec2-host-name`:5000/ in a browser to see the application running.   # ec2 sec grp'a 5000 portu(from anywhere) eklemeyi unutma

docker-compose up --build     # docker-compose down YAPTIKTAN SONRA app.py İÇERİSİNDE DEĞİŞİKLİK YAPTIYSAN BU KOMUT İLE up YAPABİLİRSİN, docker-compose up KOMUTU KULLANIRSAN OLMAZ, DEĞİŞKLİKLERİ ALGILAMAZ.
```


   ### docker-lesson klasörü içerisinde uygulama ile ilgili dosyalarımı oluşturalım -> docker-compose örneği: ÖRNEK-2
   - app.py
   - requirements.txt
   - Dockerfile
   - docker-compose.yml

```bash    (to-do-app.py) (pwd : docker-lesson)
# Create a file called `app.py` in your `docker-lesson` folder and paste the following python code in it.
# Import Flask modules
from flask import Flask, jsonify, abort, request, make_response
from flaskext.mysql import MySQL

# Create an object named app
app = Flask(__name__)

# Configure mysql database
app.config['MYSQL_DATABASE_HOST'] = 'arrow-database'      # docker-compose.yml içerisinde ==>> services: arrow-database(service'in adı)
app.config['MYSQL_DATABASE_USER'] = 'arrow'
app.config['MYSQL_DATABASE_PASSWORD'] = 'Arrow_1'
app.config['MYSQL_DATABASE_DB'] = 'todo_db'
app.config['MYSQL_DATABASE_PORT'] = 3306
mysql = MySQL()
mysql.init_app(app)
connection = mysql.connect()
connection.autocommit(True)
cursor = connection.cursor()

# Write a function named `init_todo_db` which initializes the todo db
# Create P table within sqlite db and populate with sample data
# Execute the code below only once.
def init_todo_db():
    drop_table = 'DROP TABLE IF EXISTS todo_db.todos;'
    todos_table = """
    CREATE TABLE todo_db.todos(
    task_id INT NOT NULL AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    description VARCHAR(200),
    is_done BOOLEAN NOT NULL DEFAULT 0,
    PRIMARY KEY (task_id)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
    """
    data = """
    INSERT INTO todo_db.todos (title, description, is_done)
    VALUES
        ("Project 2", "Work on project 2 with teammates", 1 ),
        ("Cloudformation Documentation", "Study and learn how to read cloudformation docs", 0),
        ("Work on CC Phonebook", "Solve python coding challenge about phonebook app", 0);
    """
    cursor.execute(drop_table)
    cursor.execute(todos_table)
    cursor.execute(data)

# Write a function named `get_all_tasks` which gets all tasks from the todos table in the db,
# and return result as list of dictionary 
# `[{'task_id': 1, 'title':'XXXX', 'description': 'XXXXXX', 'is_done': 'Yes' or 'No'} ]`.
def get_all_tasks():
    query = """
    SELECT * FROM todos;
    """
    cursor.execute(query)
    result = cursor.fetchall()
    tasks =[{'task_id':row[0], 'title':row[1], 'description':row[2], 'is_done': bool(row[3])} for row in result]
    return tasks

# Write a function named `find_task` which finds task using task_id from the todos table in the db,
# and return result as list of dictionary 
# `{'task_id': 1, 'title':'XXXX', 'description': 'XXXXXX', 'is_done': 'Yes' or 'No'}`.
def find_task(id):
    query = f"""
    SELECT * FROM todos WHERE task_id={id};
    """
    cursor.execute(query)
    row = cursor.fetchone()
    task = None
    if row is not None:
        task = {'task_id':row[0], 'title':row[1], 'description':row[2], 'is_done': bool(row[3])}
    return task

# Write a function named `insert_task` which inserts task into the todos table in the db,
# and return the newly added task as dictionary 
# `{'task_id': 1, 'title':'XXXX', 'description': 'XXXXXX', 'is_done': 'Yes' or 'No'}`.
def insert_task(title, description):
    insert = f"""
    INSERT INTO todos (title, description)
    VALUES ('{title}', '{description}');
    """
    cursor.execute(insert)

    query = f"""
    SELECT * FROM todos WHERE task_id={cursor.lastrowid};
    """
    cursor.execute(query)
    row = cursor.fetchone()
    return {'task_id':row[0], 'title':row[1], 'description':row[2], 'is_done': bool(row[3])}

# Write a function named `change_task` which updates task into the todos table in the db,
# and return updated added task as dictionary 
# `{'task_id': 1, 'title':'XXXX', 'description': 'XXXXXX', 'is_done': 'Yes' or 'No'}`.
def change_task(task):
    update = f"""
    UPDATE todos
    SET title='{task['title']}', description = '{task['description']}', is_done = {task['is_done']}
    WHERE task_id= {task['task_id']};
    """
    cursor.execute(update)

    query = f"""
    SELECT * FROM todos WHERE task_id={task['task_id']};
    """
    cursor.execute(query)
    row = cursor.fetchone()
    return {'task_id':row[0], 'title':row[1], 'description':row[2], 'is_done': bool(row[3])}

# Write a function named `remove_task` which removes task from the todos table in the db,
# and returns True if successfully deleted or False.
def remove_task(task):
    delete = f"""
    DELETE FROM todos
    WHERE task_id= {task['task_id']};
    """
    cursor.execute(delete)

    query = f"""
    SELECT * FROM todos WHERE task_id={task['task_id']};
    """
    cursor.execute(query)
    row = cursor.fetchone()
    return True if row is None else False

# Write a function named `home` which returns 'Welcome to the Callahan's To-Do API Service',
# and assign to the static route of ('/')
@app.route('/')
def home():
    return "Welcome to Callahan's To-Do API Service"


# Write a function named `get_tasks` which returns all tasks in JSON format for `GET`,
# and assign to the static route of ('/todos')
@app.route('/todos', methods=['GET'])
def get_tasks():
    return jsonify({'tasks':get_all_tasks()})


# Write a function named `get_tasks` which returns the task with given task_id in JSON format for `GET`,
# and assign to the static route of ('/todos/<int:task_id>')
@app.route('/todos/<int:task_id>', methods = ['GET'])
def get_task(task_id):
    task = find_task(task_id)
    if task == None:
        abort(404)
    return jsonify({'task found': task})

# Write a function named `add_task` which adds new task using `POST` methods,
# and assign to the static route of ('/todos')
@app.route('/todos', methods=['POST'])
def add_task():
    if not request.json or not 'title' in request.json:
        abort(400)
    return jsonify({'newly added task':insert_task(request.json['title'], request.json.get('description', ''))}), 201

# Write a function named `update_task` which updates an existing task using `PUT` method,
# and assign to the static route of ('/todos/<int:task_id>')
@app.route('/todos/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = find_task(task_id)
    if task == None:
        abort(404)
    if not request.json:
        abort(400)
    task['title'] = request.json.get('title', task['title'])
    task['description'] = request.json.get('description', task['description'])
    task['is_done'] = int(request.json.get('is_done', int(task['is_done'])))
    return jsonify({'updated task': change_task(task)})

# Write a function named `delete_task` which updates an existing task using `DELETE` method,
# and assign to the static route of ('/todos/<int:task_id>')
@app.route('/todos/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = find_task(task_id)
    if task == None:
        abort(404)
    return jsonify({'result':remove_task(task)})

# Write a function named `not_found` for handling 404 errors which returns 'Not found' in JSON format.
@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify({'error': 'Not found'}), 404)

# Write a function named `bad_request` for handling 400 errors which returns 'Bad Request' in JSON format.
@app.errorhandler(400)
def bad_request(error):
    return make_response(jsonify({'error': 'Bad request'}), 400)

# Add a statement to run the Flask application which can be reached from any host on port 80.
if __name__== '__main__':
    init_todo_db()
    # app.run(debug=True)
    app.run(host='0.0.0.0', port=80)
```


```bash (requirements.txt) (pwd : docker-lesson)
# Create another file called `requirements.txt` in your docker-lesson folder
flask
flask-mysql
```


```Dockerfile    (Dockerfile) (pwd : docker-lesson)
# Create a Dockerfile which builds a Docker image
FROM python:alpine
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 80
CMD python ./to-do-app.py
```


```yaml   (docker-compose.yml)  (pwd : docker-lesson)
# Create a file called `docker-compose.yml` in your docker-lesson folder
version: "3.7"

services:
    arrow-database:   # service'in adı
        image: mysql:5.7
        environment:    # https://hub.docker.com/_/mysql  ==>> variables  buradan geliyor. Sadece 1 tanesi zorunlu.
            MYSQL_ROOT_PASSWORD: R1234r # https://hub.docker.com/_/mysql  sayfasında "Environment Variables" "MYSQL_ROOT_PASSWORD" : This variable is mandatory
            MYSQL_DATABASE: todo_db     # to-do-app.py  içindeki bilgilerle örtüşmeli
            MYSQL_USER: arrow           # to-do-app.py  içindeki bilgilerle örtüşmeli
            MYSQL_PASSWORD: Arrow_1     # to-do-app.py  içindeki bilgilerle örtüşmeli
        networks:
            - india-net # bulunacağı network, aşağıda oluşturuyoruz
    myapp:
        build: .
        restart: always
        depends_on:     # önce arrow-database service , yani "arrow-database" isimli host, yani mysql:5.7 image'den mysql database'in container'ı ayağa kalkacak
            - arrow-database
        ports:
            - "80:80"
        networks:
            - india-net

networks:
    india-net:
        driver: bridge
```


```bash (pwd : docker-lesson)
# oluşacak container ve image'lara isim belirtmediğimiz için, service isimleri olan "arrow-database" ve "myapp" isimleri kullanılacak
docker-compose up -d    # -d : detached modda
docker network ls
    NETWORK ID     NAME                  DRIVER    SCOPE
    4375e3c6ff71   bridge                bridge    local
    805aff3af913   host                  host      local
    b64dfa26e573   none                  null      local
    8eb6dca7089d   to-do-app_india-net   bridge    local

docker container ls
    CONTAINER ID   IMAGE             COMMAND                  CREATED              STATUS              PORTS                               NAMES
    bd74627cd0a5   to-do-app-myapp   "/bin/sh -c 'python …"   About a minute ago   Up 57 seconds       0.0.0.0:80->80/tcp, :::80->80/tcp   to-do-app-myapp-1
    60a11afabdd0   mysql:5.7         "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp, 33060/tcp                 to-do-app-arrow-database-1

docker image ls
    to-do-app-myapp   latest    f82e1e604135   2 minutes ago   73.3MB
    mysql             5.7       b6ee2207ee7a   4 days ago      455MB

docker ps

docker-compose ps
docker-compose images
docker-compose config   # Run `docker-compose config` command to see the configuration of docker-compose file

http://`ec2-host-name`    # http://52.3.241.41/
http://`ec2-host-name`/todos    # http://52.3.241.41/todos
curl http://52.3.241.41/todos
curl http://52.3.241.41/todos/3

curl -H "Content-Type: application/json" -X POST -d '{"title":"Get some REST", "description":"REST in Peace"}' http://52.3.241.41/todos
curl -H "Content-Type: application/json" -X DELETE http://52.3.241.41/todos/1
curl http://52.3.241.41/todos

#####   docker compose çalıştırmadan bu kurulumu manuel olarak yapalım:
docker network create --driver bridge india-net # önce "india-net" adında network oluşturuyoruz

docker build -t "arrow-image" . # image build ediyoruz Dockerfile dosyamızdan

docker run -d  --name arrow-database --network india-net -e MYSQL_ROOT_PASSWORD=R1234r -e MYSQL_DATABASE=todo_db -e MYSQL_USER=arrow -e MYSQL_PASSWORD=Arrow_1  mysql:5.7 #   docker run diyerek mysql komutlarını yazacağız: environment vairable olarak mysql komutları yazarken -e flagini kullanıyoruz

docker run -d  --name myapp -p 80:80 -network india-net arrow-image # arrow-database konteynırı oluştu. myapp konteynırını oluşturma komutu (arrow-image : yukarıda oluşan image varsayalım)

docker-compose down
```


#####   Dockerfile Örnekleri #####

``````Dockerfile     (pwd : docker-lesson)
FROM nginx:latest
COPY index.html /usr/share/nginx/html   # Dockerfile'ın bulunduğu aynı dizinde index.html dosyası bulundurun, kendi index.html'inizi görüntüleyin
COPY aaa.png /usr/share/nginx/html      # Dockerfile'ın bulunduğu aynı dizinde index.html dosyası'nda belirtilen resim dosyası
EXPOSE 80 443 	
CMD ["nginx", "-g", "daemon off;"]
```bash

``````Dockerfile     (pwd : docker-lesson)
FROM ubuntu
RUN apt-get update -y
RUN apt-get install python3 -y
RUN apt-get install python3-pip -y
RUN pip3 install flask
COPY . /app
WORKDIR /app
CMD python3 ./phone.py  # kopyaladığın dosyalar içerisindeki python ile çalışacak uygulamanın adı
```

``````Dockerfile    (pwd : docker-lesson)
FROM ubuntu:latest
RUN apt-get update -y 
RUN apt install -y apache2 
RUN apt install -y apache2-utils 
COPY /myapp /var/www/html/  # kopyaladığın /myapp dizini içerisinde resimler (jpg) ve index.html mevcut , Dockerfile ile /myapp dizini aynı hizada/yerde olmalı
EXPOSE 80
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

``````Dockerfile     (pwd : docker-lesson)
FROM nginx:latest
RUN apt-get update -y
COPY /myapp /usr/share/nginx/html   # kopyaladığın /myapp dizini içerisinde resimler (jpg) ve index.html mevcut , Dockerfile ile /myapp dizini aynı hizada/yerde 
WORKDIR /myapp
EXPOSE 8090 	
CMD ["nginx", "-g", "daemon off;"]
```

```Dockerfile   (pwd : docker-lesson)
FROM python:alpine
COPY . /app     # Dockerfile , requirements.txt ve bookstore-api.py hepsi aynı dizinde olmalı
WORKDIR /app
RUN pip install -r requirements.txt # requirements.txt içeriği :  flask , flask-mysql
EXPOSE 80
CMD python ./bookstore-api.py
```


```Dockerfile
# postgres kurmak için Dockerfile
FROM postgres

COPY ./init.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
```

```Dockerfile
# nodejs kurmak için Dockerfile
FROM node:14
WORKDIR /usr/src/app    # Create app directory
COPY package*.json ./
RUN npm install # If you are building your code for production  # RUN npm ci --only=production
COPY . .    # copy all files into the image
EXPOSE 5000
CMD ["node","app.js"]
```

```Dockerfile
# react kurmak için Dockerfile
FROM node:14
WORKDIR /app    # Create app directory
COPY package*.json ./
RUN yarn install
COPY . .    # copy all files into the image
EXPOSE 3000
CMD ["yarn", "run", "start"]
```


#####   Multi-stage Dockerfile  #####


```bash (Dockerfile)    (pwd : docker-lesson)
FROM openjdk:8-jdk AS builder   # AS builder : buradan oluşturacağım image'ı aşağıda kullanabilmek için isimlendirme yapıyorum
COPY . /app                     # bulunduğum dizindeki dosyalar image içerisinde app klasörüne gönder, böyle bir dizin yoksa oluştur
WORKDIR /app                    # image içerisindeki app dizinine geç
RUN javac app.java              # app.java isimli uygulamayı compile et
# CMD ["java", "app"]           # compile edince app isimli class oluştu, bunu çalıştır

FROM openjdk:8-jre-alpine
WORKDIR /myapp                  # Bu image içerisinde çalışmak için myapp dizinine geç, böyle bir dizin yoksa oluştur.
COPY --from=builder /app .      # yukarıdaki image'ın app dizinindekileri dosyaları bulunduğum "." yani /myapp dizinine kopyala. Gelecek dosyalar içerisinde "app.java" ve "app.class" bulunur
CMD ["java", "app"]
```




























-----------------------------------------------------------------------------------------------------
#####   BÜYÜK CAPSTONE PROJE KAPSAMINDA  #####

- Elimizde bu şekilde `java` ile yazılmış uygulama (app.java) olduğunu varsayalım, `maven` yüklemeden `jar` paketini nasıl oluştururuz, jar paketi lazım. Maven yüklemeden jar paketleri oluşsun istiyorum. Nasıl yaparız ==> Docker :  docker.hub'ta bunu yapabilen bir image vardır : `maven:3.6-openjdk-11`. Bunu kullanabilmek için bilgisayarda docker kurulu olması lazım
- m2'ye dependency'leri nasıl aktarırım, aktarmazsam eğer her docker komutu çalıştığında container içine dependency'lerin inmesini bekleyeceksin.
- docker run yapıp maven image'ını çalıştırdığımda `pom` dosyamı gören bir yerde run etmeliyim

1. jar paketleri lokale çekilecek,
2. dependency'ler m2'ye aktarılacak.    ==>>  m2 folder'ı `volume` olarak bağla

```bash (pwd : /home/ec2-user)
# docker.hub'tan image: maven:3.6-openjdk-11
# docker kurmak için komutlar:
sudo dnf install docker
sudo usermod -a -G docker ec2-user
newgrp docker
sudo systemctl start docker
sudo systemctl enable docker
```

- Bir image'dan container oluşturacağım, run ederken sonunda komut girebiliyorum


```bash  (pwd : /home/ec2-user/maven-experiment)
ls          # effective.xml     pom.xml     src     target
pwd         # /home/ec2-user/maven-experiment
echo `pwd`  # /home/ec2-user/maven-experiment               ### `pwd`  ==  $(pwd)       AYNI ŞEY
echo $(pwd) # /home/ec2-user/maven-experiment


# --rm : oluştuktan sonra container ile işin bitecek, container'ı sil , volume kalmaya devam edecek.
# $HOME : /home/c2-user     ==>> .m2 klasörü : /home/ec2-user içerisinde ==>>  $HOME/.m2  klasörünü volume olarak bağlayacak
# /root/.m2 : container içerisinde root içerisinde .m2 klasörüne bağlanacak

# pwd : /home/ec2-user/maven-experiment ==>> pom dosyamın da içerisinde olduğu, `mvn clean package` komutunu çalıştırdığım dizin
# `pwd` : /home/ec2-user/maven-experiment   klasörünü volume olarak bağlayacak
# container içerisinde /app diye folder'a bağlanacak, böyle bir folder olmasa da oluşacak

# -w : working directory , docker run ile beraber komut yazabilirim, komutun çalışcağı directory'i gösteriyorum
# -w /app : komut oluşan /app dizini içerisinde çalışacak
# mvn clean package : /app dizini içerisinde çalışacak komut, /app dizinine `pwd` bağlanmıştı, yani /home/ec2-user/maven-experiment dizini bağlanmıştı,
# maven:3.6-openjdk-11 : image ismi 

docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.6-openjdk-11 mvn clean package

```



