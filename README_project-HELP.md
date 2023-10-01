# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 1 - Prepare Development Server Manually on EC2 Instance***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* `JaCoCo Java Code Coverage`
Code coverage is a software metric used to measure how many lines of our code are executed during automated tests.
JaCoCo is a software metric used to measure how many lines of our code(written in java) are executed during automated tests.

* Prepare `development server` manually on Amazon Linux 2023 (t3a.medium) , enabled with `Docker`,  `Docker-Compose`,  `Java 11`,  `Git`.

``` bash
#! /bin/bash
sudo dnf update -y
sudo dnf install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker    # active -- running
sudo usermod -a -G docker ec2-user
sudo curl -SL https://github.com/docker/compose/releases/download/v2.17.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker version
docker compose --version       # v2.16.0
sudo dnf install git -y
git -v                         # git version 2.40.1
sudo dnf install java-11-amazon-corretto -y
java -version
newgrp docker
```

# have to write it in .bashrc

```bash
sudo hostnamectl set-hostname "dev-server"

# to see branch name on bash terminal
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/*\s*\(.*\)/(\1)/'
}
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[33m\]\[\e[1;34m\]\$(parse_git_branch)$\[\033[00m\] "
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 2 - Prepare GitHub Repository for the Project***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Create a new repository on your Github account with the name **petclinic-microservices-with-db**.

* Connect to your Development Server via `ssh` and clone the petclinic app from the repository [Spring Petclinic Microservices App](https://github.com/clarusway/petclinic-microservices-with-db.git).
* Initiate the cloned repository to make it a git repository and push the local repository to your remote repository.
* You should delete the ".git" directory of the repository you cloned, so that you can initiate (create your own local repo with `git init`) and connect it to the remote repository.
* Clone yaptığın reponun ".git" dizinini silmelisin ki yeniden init yapıp kendi lokal reponu oluştur ve remote repon ile bağlantısını kur.

```bash (pwd : /home/ec2-user)
git clone https://github.com/clarusway/petclinic-microservices-with-db.git
cd petclinic-microservices-with-db
rm -rf .git     ### You should delete the ".git" directory  ## .git  SİLMELİSİN.

git init
git add .
git config --global user.email "alparslanu6347@gmail.com"
git config --global user.name "alparslanu6347"
git commit -m "first commit"
git branch -M main
git remote add origin https://alparslanu6347:*****TOKEN*****@github.com/alparslanu6347/petclinic-microservices-with-db.git
git remote -v   # display a list of remote repositories that your local Git repository is connected to, along with their URLs; Check if the local repo is connected to the remote repo. if not, use command below
# git remote set-url origin https://*****TOKEN*****@github.com/alparslanu6347/petclinic-microservices-with-db.git
git push origin main  # push the local branch named "main" to the remote repository named "origin." 
```

* Prepare base branches namely `dev` and `release` for DevOps cycle.
  * Create `dev` and `release` base branch.

``` bash
git checkout main                   
git branch dev                      
git checkout dev                    
git push --set-upstream origin dev  # push your local branch "dev" to the remote repository "origin" and set it as the upstream branch.

git checkout dev
git branch release
git checkout release
git push --set-upstream origin release
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 3 - Check the Maven Build Setup on Dev Branch***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Test the compiled source code.
* Komuttan sonra .m2 lokal repo oluştu , .m2 içerisinde `wrapper` oluştu --->> `apache-maven-3.5.4-bin`  geldi, bütün developer'lar proje boyunca bunu kullanacak
* `mvn` komutunu direkt olarak kullanmak istesek her bir instance içerisine `maven` kurmak zorundayız. Üstelik maven versiyon değişikliğinde - yeni instance ihtiyacında - yeni developer geldiğinde yeniden maven'in latest versiyonunu yüklemek zorunda kalırız.
* Maven'ın en önemli özelliği zaten versiyon kontrol sağlaması, ama biz daha maven'ın versiyonunu ayarlayamaz hale geliriz.
* Maven wrapper zaten içerisinde belirlenmiş ayarlara göre ilgili maven plugin'i indirir ve komutları oradan çalıştırır.
```bash
package  # take the compiled code and package it in its distributable format, such as a JAR.
install  # install the package into the local repository (.m2), for use as a dependency in other projects locally
deploy   # done in the build environment, copies the final package to the remote repository for sharing with other developers and projects. if we have nexus/packet repo we can send/push the jar packet there
clean    # Clean up the existing build artifacts and target directory

# we can use 2 commands at the same time in the same command line
mvnw clean test #  validate + compile + test (firstly delete the target directory)
mvnw clean package  #  validate + compile + test + package (firstly delete the target directory)
mvnw clean install  # Clean up the existing build artifacts and target directory.
                    # Compile the source code.
                    # Run tests (if any are defined in the project).
                    # Package the project's artifacts.
                    # Install the project's artifacts into your local Maven repository.
                    # This command is commonly used during the development and build process of Java projects managed with Maven. It ensures that your project is built from a clean state, and the resulting artifacts are available for use by other projects or modules within the same build.
```

``` bash
./mvnw clean test
> Note: If you get `permission denied` error, try to give execution permission to **mvnw**.  

    chmod +x mvnw
```


* Install distributable `JAR`s into local repository.

``` bash
./mvnw clean install

# https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.
```

- Install  nereye yapacak --->>> pom.xml  --- gav  : group ID , artifact ID , version

* Take the compiled code and package it in its distributable `JAR` format.
``` bash
./mvnw clean package  #  target silindi yeniden paketlendi, jar dosyaları hazır.
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 4 - Prepare a Script for Packaging the Application***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-4` branch from `dev`, `feature/msp-.......`
git checkout dev
git branch feature/msp-4
git checkout feature/msp-4
```

- Prepare a script to package the application with maven wrapper and save it as `package-with-mvn-wrapper.sh` under `petclinic-microservices-with-db` folder.

``` bash
./mvnw clean package #  target silindi yeniden paketlendi, jar dosyaları hazır.
```

``` bash
# Commit and push the new script to remote repo.
git add .
git commit -m 'added packaging script'
git push --set-upstream origin feature/msp-4 # git push -u origin feature/msp-4 (same command)
git checkout dev
git merge feature/msp-4
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 5 - Prepare Development Server Terraform Files***
Create the ec2-instance that we have prepared manually for the developers with the terraform file.
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
- Developer'lar için manuel olarak hazırlamış olduğumuz ec2-instance'ı terraform dosyası ile oluşturun.
- Create `feature/msp-5` branch from `dev`.

``` bash
git checkout dev
git branch feature/msp-5
git checkout feature/msp-5
```

- Create a folder for infrastructure setup with the name of `infrastructure` under `petclinic-microservices-with-db` folder.

``` bash
mkdir infrastructure
```

- En başta manuel olarak yaptığımız kurulumu `infrastructure/msp-5-dev-server-of-petclinic/` klasörü altındaki terraform dosyalarına aktardık ve bu dosyaları çalıştıracak `petclinicserver-userdata.sh` isminde bir script hazırladık. Çlıştırmadan önce `dev.auto.tfvars` içerisindeki `mykey` değiştirmeyi unutma.

* 3 adet `Amazon Linux 2023` `t3a.medium` makinalarda `Docker , Docker-Compose , Java 11 , Git` yüklenecek.

``` bash
# Commit and push the new script to remote repo.
git add .
git commit -m 'added terraform files for dev server'
git push --set-upstream origin feature/msp-5
git checkout dev
git merge feature/msp-5
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 6 - Prepare Dockerfiles for Microservices***
Create images for each microservice by preparing Dockerfiles.
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-6` branch from `dev`.
git checkout dev
git branch feature/msp-6
git checkout feature/msp-6
```

* Her bir server için kendi klasörü (`spring-petclinic-admin-server` gibi) içerisinde olacak şekilde Dockerfile hazırla:

```bash	(pwd : home/ec2-user)
# Dockerfile hazırlamayı anlat
mkdir test
cd test
code Dockerfile
```

``` Dockerfile
# base image java version 11 olacak ,  mvnw clean package komutu ile compile edildiği için bizim zaten .jar dosyalarımız oluştu , burada jdk'ya ihtiyacımız yok , jre : java runtime engine yeterli -- image daha da az yer kaplar.
FROM openjdk:11-jre
COPY ./target/*.jar /app.jar
ENTRYPOINT [ "java", "-jar", "/app.jar" ]
```

*** "-Djava.security.egd=file:/dev/./urandom","-jar"!!!!!!!  `urandom`  !!!!!!!!***
*******************EXPLAIN*******************



*** base image `java version 11` olacak ,  `mvn clean package` komutu ile `compile` edildiği için bizim zaten her bir mikroservice içinde kendi`.jar` dosyaları oluştu , burada jdk'ya ihtiyacımız yok , `jre : java runtime engine` yeterli -- image daha da az yer kaplar***

``` Dockerfile
FROM openjdk:11-jre
# https://github.com/spring-projects/spring-petclinic
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/v0.6.1/dockerize-alpine-linux-amd64-v0.6.1.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
# best practice olarak eğer biz url'den kopyalama yapmayacaksak COPY kullanılır
# Dockerfile ilgili server klasöründe, .jar dosyaları da aynı klasör içerisindeki target klasörü içerisinde
COPY ./target/*.jar /app.jar
# bilgi mahiyetli expose
EXPOSE 9090
# java -jar komutu .jar dosyalarını çalıştırır
ENTRYPOINT [ "java", "-jar", "/app.jar" ]
```

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=9090
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

*** `COPY` ve `ADD` arasındaki `fark***

  - `ADD` komutu `COPY` ile aynı işi yaparken ek özelliklere sahiptir : URL'lerden dosyaları indirebilir, otomatik olarak dosyaları açabilir ve tar arşivlerini çözebilir.
  - Genel olarak, sadece yerel dosyaları kopyalamak istiyorsanız `COPY` komutunu tercih etmek daha iyi olabilir.
  - `ADD` komutunun ek özelliklerine ihtiyaç duyuyorsanız veya Dockerfile'ınızı daha esnek hale getirmek isterseniz `ADD` komutunu kullanabilirsiniz.

*** Dockerize***
* 1. Önce Cloud Config Server çalışıyor, github reposundan konfigürasyon bilgilerini çekiyor.
* 2. Sonra Discovery server çalışıyor, ama önce Config Server'ın ayağa kalktığından emin olması lazım.
* 3. Diğer tüm server'ler ayağa kalkıyor, Config Server'dan konfigürasyon bilgilerini çekiyorlar, sonrasında Discovery server'a biz hazırız diye rapor ediyorlar  ki Discovery server bunları hata var mı yok mu takip ediyor.
* Peki Discovery server --->> Config Server   veya  Diğer server'lar --->> Config Server  ayağa kalktığını nasıl kontrol edeceğim  ==>> `dockerize`
* docker-compose'daki depend-on container ayaktamı diye bakar, application ayaktamı bakmaz
* Fakat dockerize server'ın/server içindeki application'ın çalışıp çalışmadığına bakar
# <https://github.com/jwilder/dockerize/releases>

``` bash
wget https://github.com/jwilder/dockerize/releases/download/v0.6.1/dockerize-alpine-linux-amd64-v0.6.1.tar.gz dockerize.tar.gz
ls    # Dockerfile    dockerize-alpine-linux-amd64-v0.6.1.tar.gz
mv dockerize-alpine-linux-amd64-v0.6.1.tar.gz  dockerize.tar.gz
ls    # Dockerfile    dockerize.tar.gz
tar -xzf dockerize.tar.gz
ls    # Dockerfile    dockerize   dockerize.tar.gz
chmod +x dockerize
```

*** ARG anlamak***
- The ARG instruction defines a variable that users can pass at `build-time` to the builder with the docker build command using the `--build-arg <varname>=<value>` flag. If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning. 

- An ARG instruction can optionally include a default value. If an ARG instruction has a default value and if there is no value passed at build-time, the builder uses the default.
# image `build` ederken `ARG` değiştirebiliriz. docker build.
# !!! Dockerfile  hiç dokunmadan versiyon değiştirmek için komut kullanıyorum. SADECE image build EDERKEN DEĞİŞTİRİYORUM.

# container'ı var olan image üzerinden `run` ederken `ENV` değiştirebiliriz. docker run.

- Finally, we don't update the Dockerfile but we update the image with `--build-arg <varname>=<value>` flag.
```bash  
# Bu Dockerfile kullanarak bulunduğu dizinde komut çalıştırarak image build edelim ki --->> ARG anlayalım.
docker build -t arg-anlamak .
docker run arg-anlamak        # UYGULAMAYI BU TERMİNALDE AÇIK BIRAK , YAN TARAFTA YENİ BİR TERMİNAL AÇ
docker ps     # 8c07e3da32984 
docker exec -it 8c07 bash   # root@8c07e3da32984:/#
root@8c07e3da32984:/# ls  # app.jar   bin   boot    dev   dockerize   dockerize.tar.gz    .......
root@8c07e3da32984:/# ./dockerize -version    # v0.6.1
root@8c07e3da32984:/#  exit 

docker image prune  # tüm image'ları siler
docker image prune -af  # tum imajlari silmek icin
docker container ps -aq     # veya   `docker ps -aq`  tüm container'ların (çalışan-çalışmayan) SADECE ID'sini GÖSTERİR
docker rm -f $(docker ps -aq)   # (çalışan-çalışmayan tüm containerları siler)
#  DİĞER TERMİNALDE AÇTIĞIN UYGULAMAYI DA KAPAT
```

```bash
# ARG DOCKERIZE_VERSION=v0.6.1
# we don't update the Dockerfile but we update the image with `--build-arg <varname>=<value>` flag.
docker build --build-arg DOCKERIZE_VERSION=v0.7.0 -t arg-understand .
docker run arg-understand
docker ps   # 144cc485eca6
docker exec -it 144c bash   # root@144cc485eca6:/#
root@144cc485eca6:/# ls  # app.jar   bin   boot    dev   dockerize   dockerize.tar.gz    .......
root@144cc485eca6:/#  ./dockerize -version    # v0.7.0
```

*** ENV SPRING_PROFILES_ACTIVE docker,mysql***
* ENV SPRING_PROFILES_ACTIVE docker,mysql  : eğer uygulamayı container içerisinde çalıştıracaksan ve mysql database kullanacaksan bunu Dockerfile içine ekle demiştir Developer.
  * Normalde default database olarak in-memory database HSQLDB kullanıyor.
  * Her bir microservice klasörü içerisinde `../mvnw spring-boot:run` komutunu girerseniz default değerlerle (container olmadan ve in-memory database HSQLDB ile) ayağa kalkar. Localhost yani instance içerisinden yayın yapar

* microservice'lerin server'ları basıl birbirini görecek, isimler uygulamada içinde belirtilmiş.
  * spring-petclinic-discovery-server/src/main/resources/bootstrap.yml --->> AÇ :
    * default olursa nereye gidecek -->> <http://localhost:8888>
    * profiles:docker olursa        -->> <http://config-server:8888>   , YANİ SPRING_PROFILES_ACTIVE docker  YAPARSAM
  * PROFİL  ==>> maven konusu , ana dizinde pom.xml  aç -->> profile kısımlarına bak  ==>> `buildDocker` ve `springboot`

## SIRASIYLA 8 ADET DOCKERFILE

* Prepare a Dockerfile for the `admin-server` microservice with following content and save it under `spring-petclinic-admin-server`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=9090
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `api-gateway` microservice with the following content and save it under `spring-petclinic-api-gateway`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8080
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `config-server` microservice with the following content and save it under `spring-petclinic-config-server`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8888
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `customer-service` microservice with the following content and save it under `spring-petclinic-customer-service`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8081
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `discovery-server` microservice with the following content and save it under `spring-petclinic-discovery-server`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8761
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `hystrix-dashboard` microservice with the following content and save it under `spring-petclinic-hystrix-dashboard`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=7979
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `vets-service` microservice with the following content and save it under `spring-petclinic-vets-service`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8083
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

* Prepare a Dockerfile for the `visits-service` microservice with the following content and save it under `spring-petclinic-visits-service`.

``` Dockerfile
FROM openjdk:11-jre
ARG DOCKERIZE_VERSION=v0.6.1
ARG EXPOSED_PORT=8082
ENV SPRING_PROFILES_ACTIVE docker,mysql
ADD https://github.com/jwilder/dockerize/releases/download/${DOCKERIZE_VERSION}/dockerize-alpine-linux-amd64-${DOCKERIZE_VERSION}.tar.gz dockerize.tar.gz
RUN tar -xzf dockerize.tar.gz
RUN chmod +x dockerize
ADD ./target/*.jar /app.jar
EXPOSE ${EXPOSED_PORT}
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
``` bash
# Commit the changes, then push the Dockerfiles to the remote repo.
git add .
git commit -m 'added Dockerfiles for microservices'
git push --set-upstream origin feature/msp-6
git checkout dev
git merge feature/msp-6
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 7 - Prepare Script for Building Docker Images***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-7` branch from `dev`.
git checkout dev
git branch feature/msp-7
git checkout feature/msp-7
```

* Docker image'ları build etmek için `petclinic-microservices-with-db` klasörü altında `build-dev-docker-images.sh` isminde script oluştur:

- Removing Intermediate Containers: By default, Docker keeps intermediate containers around after each instruction is executed. This allows for faster builds when you make changes to your Dockerfile because Docker can reuse the cached layers. However, if you use the --force-rm flag, Docker will always remove these intermediate containers after a/an `successful/unsuccessful` build to keep the system clean.
- https://docs.docker.com/engine/reference/commandline/compose_build/
- 
```bash
# --force-rm : Always remove intermediate containers

docker build --force-rm -t "petclinic-admin-server:dev" ./spring-petclinic-admin-server
```

``` bash
./mvnw clean package
docker build --force-rm -t "petclinic-admin-server:dev" ./spring-petclinic-admin-server
docker build --force-rm -t "petclinic-api-gateway:dev" ./spring-petclinic-api-gateway
docker build --force-rm -t "petclinic-config-server:dev" ./spring-petclinic-config-server
docker build --force-rm -t "petclinic-customers-service:dev" ./spring-petclinic-customers-service
docker build --force-rm -t "petclinic-discovery-server:dev" ./spring-petclinic-discovery-server
docker build --force-rm -t "petclinic-hystrix-dashboard:dev" ./spring-petclinic-hystrix-dashboard
docker build --force-rm -t "petclinic-vets-service:dev" ./spring-petclinic-vets-service
docker build --force-rm -t "petclinic-visits-service:dev" ./spring-petclinic-visits-service
docker build --force-rm -t "petclinic-grafana-server:dev" ./docker/grafana
docker build --force-rm -t "petclinic-prometheus-server:dev" ./docker/prometheus
```

```bash
# Give execution permission to build-dev-docker-images.sh. 
chmod +x build-dev-docker-images.sh

# Build the images.
./build-dev-docker-images.sh
```

``` bash
# Commit the changes, then push the new script to the remote repo.
git add .
git commit -m 'added script for building docker images'
git push --set-upstream origin feature/msp-7
git checkout dev
git merge feature/msp-7
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 8 - Create Docker Compose File for Local Development***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-8` branch from `dev`.
git checkout dev
git branch feature/msp-8
git checkout feature/msp-8
```

* Toplam 8 tane container çalışacak ya ben her biri için docker run komutu kullanırım, ya da declerative method : docker-compose kullanırım.
*** `version2`  --->>> version3    geçiş yaparken  ``mem_limit`` özelliği kaldırılmış, bir container'e memory limit belirlemezsek içinde bulunduğu hostun tüm memory'sini sömürebilir, host çöker. `mem_limit: 512M` kullanabilmek için version2 kullanıyorum. Version3'te bu memory limit özelliği kullanabilmek için başka bir özellik `deploy` kullanılıyor ki bu da daha çok docker swarm içinde kullanılıyor.***

* Prepare docker compose file to deploy the application locally and save it as `docker-compose-local.yml` under `petclinic-microservices-with-db-de` folder.

```yml
version: '2'
# https://docs.docker.com/compose/compose-file/compose-versioning/#version-3
services: 
  config-server:    # src/main/resources/bootstrap.yaml,  Dockerfile anlatırken bu ismim nereden geldiğini ve neden bu ismi kullandığımı anlattım.
    image: petclinic-config-server:dev  # Yukarıda Dockerfile ile oluşturduğumuz image isimleri
    container_name: config-server   # best-practice olarak container ismi = service ismi verirsiniz
    mem_limit: 512M
    ports: 
      - 8888:8888   # 8888 yazarsan müsait olan herhangi bir porttan yayın yapar , 8888:8888 yazarsan sadece 8888'den (birinci 8888) yayın yapar, Dış dünyaya yayın yapmasın istiyorsan port koymayacaksın

  discovery-server:
    image: petclinic-discovery-server:dev
    container_name: discovery-server
    mem_limit: 512M
    ports: 
      - 8761:8761   # Bu portları developer'lar söylüyor
    depends_on:     # discovery-server ----BEKLEYECEK---->> config-server'İN container'I AYAĞA KALKMASINI
      - config-server
    entrypoint: ["./dockerize", "-wait=tcp://config-server:8888", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"] # "entrypoint" Dockerfile içerisinde "ENTRYPOINT" üzerine yazar
# depends on : container'ın ayağa kalkmasını bekler, AMA UYGULAMA AYAĞA KALKMAMIŞ/HAZIR OLMAMIŞ OLABİLİR
# dockerize  : uygulamanın hazır hale gelmesini bekler
# ./dokerize", "-wait=tcp://config-server:8888", "-timeout=160s" : config-server yayın yapıyor mu? Yapmıyorsa 160 saniye bekledikten sonra tekrar kontrol et, hazır değilse tekrar 160 saniye bekledikten sonra kontrol et, yayın yaptığını gördükten sonra komutu çalıştır.
# "--" : 2 komut bloğunu birbirinden ayırmak için java komutları arasında kullanılır, linux'teki ; veya && gibi


  customers-service:
    image: petclinic-customers-service:dev
    container_name: customers-service
    mem_limit: 512M
    ports:
     - 8081:8081
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]


  visits-service:
    image: petclinic-visits-service:dev
    container_name: visits-service
    mem_limit: 512M
    ports:
     - 8082:8082
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]
  
  vets-service:
    image: petclinic-vets-service:dev
    container_name: vets-service
    mem_limit: 512M
    ports:
     - 8083:8083
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]


  api-gateway:
    image: petclinic-api-gateway:dev
    container_name: api-gateway
    mem_limit: 512M
    ports:
     - 8080:8080
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]
  
  admin-server:
    image: petclinic-admin-server:dev
    container_name: admin-server
    mem_limit: 512M
    ports:
     - 9090:9090
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]


  hystrix-dashboard:
    image: petclinic-hystrix-dashboard:dev
    container_name: hystrix-dashboard
    mem_limit: 512M
    ports:
     - 7979:7979
    depends_on: 
     - config-server
     - discovery-server
    entrypoint: ["./dockerize", "-wait=tcp://discovery-server:8761", "-timeout=160s", "--", "java", "-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar" ]


  tracing-server: # logları tutuyor, bekleme/rapor etmesine gerek yok.
    image: openzipkin/zipkin
    container_name: tracing-server
    mem_limit: 512M
    ports:
     - 9411:9411

  grafana-server:
    image: petclinic-grafana-server:dev
    container_name: grafana-server
    mem_limit: 256M
    ports:
    - 3000:3000

  prometheus-server:
    image: petclinic-prometheus-server:dev
    container_name: prometheus-server
    mem_limit: 256M
    ports:
    - 9091:9090

  mysql-server:
    image: mysql:5.7.8
    container_name: mysql-server
    environment: 
      MYSQL_ROOT_PASSWORD: petclinic  # developer belirlemiş, bize de bu bilgiyi veriyor
      MYSQL_DATABASE: petclinic # developer belirlemiş, bize de bu bilgiyi veriyor
    mem_limit: 256M
    ports:
    - 3306:3306
```


* Uygulamayı ayağa kaldır ve
* Uygulamayı lokal olarak `docker-compose-local.yml` dosyasını kullanarak deploy etmek için `petclinic-microservices-with-db` klasörü altında `test-local-deployment.sh` isminde script hazırla:

``` bash
docker-compose -f docker-compose-local.yml up
                ### VEYA ###
# Give execution permission to test-local-deployment.sh.
chmod +x test-local-deployment.sh
# Execute the docker compose.
./test-local-deployment.sh
```

# browser'da `http://publicIP:8080` -->> api-gateway üzerinden uygulamaya erişebilirsin.

``` bash
# Commit the change, then push the docker compose file to the remote repo.
git add .
git commit -m 'added docker-compose file and script for local deployment'
git push --set-upstream origin feature/msp-8
git checkout dev
git merge feature/msp-8
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 9 - Prepare Jenkins Server for CI/CD Pipeline***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-9` branch from `dev`.
git checkout dev
git branch feature/msp-9
git checkout feature/msp-9
```

* `infrastructure` klasörü altında `msp-9-jenkins-server-template` klasörü oluştur ve terraform dosyalarını içerisine kopyala-yapıştır.

* Set up a Jenkins Server and enable it with `Git`,  `Docker`,  `Docker Compose`,  `AWS CLI v2`, `Python`,  `Ansible` and `Boto3`.  To do so, prepare a [Terraform file for Jenkins Server](./msp-9-jenkins-server-tf-template) with following scripts (jenkins_variables.tf, jenkins-server.tf, jenkins.auto.tf.vars, jenkinsdata.sh) and save them under `infrastructure` folder.

``` bash
#! /bin/bash
# update os
dnf update -y
# set server hostname as jenkins-server
hostnamectl set-hostname jenkins-server
# install git
dnf install git -y
# install java 11
dnf install java-11-amazon-corretto -y
# install jenkins
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
dnf upgrade
dnf install jenkins -y
systemctl enable jenkins
systemctl start jenkins
# install docker
dnf install docker -y
systemctl start docker
systemctl enable docker
usermod -a -G docker ec2-user
usermod -a -G docker jenkins
# configure docker as cloud agent for jenkins
cp /lib/systemd/system/docker.service /lib/systemd/system/docker.service.bak
sed -i 's/^ExecStart=.*/ExecStart=\/usr\/bin\/dockerd -H tcp:\/\/127.0.0.1:2376 -H unix:\/\/\/var\/run\/docker.sock/g' /lib/systemd/system/docker.service
systemctl daemon-reload
systemctl restart jenkins
# install docker compose
curl -SL https://github.com/docker/compose/releases/download/v2.17.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
# install python 3
dnf install -y python3-pip python3-devel
# install ansible
pip3 install ansible
# install boto3
pip3 install boto3 botocore
# install terraform
wget https://releases.hashicorp.com/terraform/1.4.6/terraform_1.4.6_linux_amd64.zip
unzip terraform_1.4.6_linux_amd64.zip -d /usr/local/bin
```

``` bash
# Commit the change, then push the terraform files file to the remote repo.
git add .
git commit -m 'added jenkins server terraform files'
git push --set-upstream origin feature/msp-9
git checkout dev
git merge feature/msp-9
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 10 - Configure Jenkins Server for Project***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* `msp-9-jenkins-server-tf-template`  klasörü içerisindeki terraform dosyaları ile `lokalde-Masaüstünde` "terraform init , terraform apply" ile Jenkins-server ayağa kaldır.
* Remote SSH ile bağlan
* After launch we will go on jenkins server. So, clone the project repo to the jenkins server.

```bash
# clone the project repo to the jenkins-server
git clone https://alparslanu6347:***TOKEN***@github.com/alparslanu6347/petclinic-microservices-with-db.git
```

* Get the initial administrative password.
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword # şifreyi al
```

* Jenkins aç : <http://publicIP:8080>
* Enter the temporary password to unlock the Jenkins.
* Install suggested plugins.
* Open your Jenkins dashboard and navigate to `Manage Jenkins` >> `Manage Plugins` >> `Available` tab
*** If your dashboard language is not `English` additionally first step -->> install plugin`Locale` second step `Manage Jenkins` -->> `System Configuration` -->> `Change System Configuration (Sistem Konfigürasyonu Değiştir)` -->>  `Locale->Default Language : en`   +  `Ignore browser preference and force this language to all users`  tick  -->> Save**
* Search and select `GitHub Integration`,  `Docker`,  `Docker Pipeline`, and `Jacoco` plugins, then click `Install without restart`. Note: No need to install the other `Git plugin` which is already installed can be seen under `Installed` tab.

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 11 - Setup Unit Tests and Configure Code Coverage Report***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-11` branch from `dev`.
git checkout dev
git branch feature/msp-11
git checkout feature/msp-11
```

* `Unit test` :Create following unit tests for `Pet.java` under `customer-service` microservice using the following `PetTest` class and save it as `PetTest.java` under `./spring-petclinic-customers-service/src/test/java/org/springframework/samples/petclinic/customers/model/` folder.
* Önce `model` klasörünü oluştur ve sonrasında içinde `PetTest.java` dosyasını oluştur.

``` java
package org.springframework.samples.petclinic.customers.model;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;

import org.junit.jupiter.api.Test;
public class PetTest {
    @Test
    public void testGetName(){
        //Arrange
        Pet pet = new Pet();
        //Act
        pet.setName("Fluffy");
        //Assert
        assertEquals("Fluffy", pet.getName());
    }
    @Test
    public void testGetOwner(){
        //Arrange
        Pet pet = new Pet();
        Owner owner = new Owner();
        owner.setFirstName("Call");
        //Act
        pet.setOwner(owner);
        //Assert
        assertEquals("Call", pet.getOwner().getFirstName());
    }
    @Test
    public void testBirthDate(){
        //Arrange
        Pet pet = new Pet();
        Date bd = new Date();
        //Act
        pet.setBirthDate(bd);
        //Assert
        assertEquals(bd,pet.getBirthDate());
    }
}
```

* Testi anlatmak için `main-model` içindeki `Pet.java` ile `test-model` içerisindeki `PetTest.java` dosyalarını yan yana VSC'da aç, ve ilk testi anlat ve hatta ilk testin ikinci kısmını` assertEquals("Fluffy", pet.getName()); ` -->> Fluffy'yi değiştirdikten sonra `../mvnw clean test` komutunu tekrar execute et ve hatayı göster.

* Implement unit tests with maven wrapper for only `customer-service` microservice locally on `Dev Server`. Execute the following command under the `spring-petclinic-customers-service folder`.

``` bash  (pwd :/home/ec2-user/petclinic-microservices-with-db-de/spring-petclinic-customers-service)
../mvnw clean test
```

``` bash  (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
# Commit the change, then push the changes to the remote repo.
# git config --global user.email "alparslanu6347@gmail.com"
# git config --global user.name "alparslanu6347"
git add .
git commit -m 'added 3 UTs for customer-service'
git push --set-upstream origin feature/msp-11
```

* Update POM file at `root` folder for Code Coverage Report using `Jacoco` tool plugin.

* Maven kullanıyorsan -- Bu plugini ekleyeceksen -- dependency olarak :
  * <https://mvnrepository.com/>  --->>> search : jacoco  --->>> 2. JaCoCo :: Maven Plugin  ===>>> Versiyonları listeleniyor --->>> 0.8.10 ===>>>

      <!-- https://mvnrepository.com/artifact/org.jacoco/jacoco-maven-plugin -->
      <dependency>
          <groupId>org.jacoco</groupId>
          <artifactId>jacoco-maven-plugin</artifactId>
          <version>0.8.10</version>
      </dependency>
* Maven kullanıyorsan -- Bu plugini ekleyeceksen -- plugin ile yapmak istediğin `goals` (test,rapor... gibi) göre ne yazacağını gösteriyor
  * <https://mvnrepository.com/>  --->>> search : jacoco  --->>> 2. JaCoCo :: Maven Plugin  ===>>> Bu plugin'in sayfası --->>> HomePage <https://www.jacoco.org/jacoco/trunk/doc/maven.html>

* Ana dizindeki `pom.xml` içerisinde `profiles` içerisinde `springboot` profili içerisinde `plugins` içerisinde bir plugin olarak ekliyoruz.

``` xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <!-- attached to Maven test phase -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

* `spring-petclinic-customers-service folder` içerisinde aşağıdaki komutu çalıştırıp test tamamlanınca `target/site/jacoco` içerisinde test sonucu için html dosyaları oluşuyor.

``` bash  (pwd : /home/ec2-user/petclinic-microservices-with-db-de/spring-petclinic-customers-service)
../mvnw test    # test yaptık
```


``` bash  (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
# Commit the change, then push the changes to the remote repo.
git add .
git commit -m 'updated POM with Jacoco plugin'
git push
git checkout dev
git merge feature/msp-11
git push origin dev
```

* Html dosyaları çalıştırıp websitesi olarak görmek için `target/site/jacoco` içerisinde yeni bir komut daha giriyoruz. Biz bu raporları jenkins ile oluşturacağız, burada halen daha developer neyi nasıl yapıyor onu anlamaya çalışıyoruz.
* Deploy code coverage report (located under relative path `target/site/jacoco` of the microservice) on Simple HTTP Server for only `customer-service` microservice on `Dev Server`.

``` bash  (/home/ec2-user/petclinic-microservices-with-db-de/spring-petclinic-customers-service/target/site/jacoco)
python -m SimpleHTTPServer # for python 2.7
python3 -m http.server # for python 3 # çalıştırdıktan sonra çıkan linki tıkla ve gör

# org.springframework.samples.petclinic.customers.model (ortadaki) folder aç ve incele
### Cyclomatic complexity gibi açıklamalar var (Report Analysis) -->> https://www.baeldung.com/jacoco
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 12 - Prepare Continuous Integration (CI) Pipeline***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-12` branch from `dev`.
git checkout dev
git branch feature/msp-12
git checkout feature/msp-12
```

* Create a folder, named `jenkins`, to keep `Jenkinsfiles` and `Jenkins jobs` of the project.

``` bash  (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
mkdir jenkins
```

- Create a ``Jenkins job`` Running `Unit Tests` on Petclinic Application

```yml
- job name: petclinic-ci-job
- job type: Freestyle project
- GitHub project: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): - */dev 
                                          - */feature**
                                          - */bugfix**
- Build triggers: GitHub hook trigger for GITScm polling
- Build Environment: Add timestamps to the Console Output
- Build Steps:
      Add build step: Execute Shell
      Command
-Post-build Actions:
     Add post-build action: Record jacoco coverage report
```

```bash (Build Steps: Add build step: Execute Shell - Comman)
echo 'Running Unit Tests on Petclinic Application'
docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.8-openjdk-11 mvn clean test
```

* Create a webhook for Jenkins CI Job :

  * Go to the project repository page and click on `Settings`.

  * Click on the `Webhooks` on the left hand menu, and then click on `Add webhook`.

  * Copy the Jenkins URL, paste it into `Payload URL` field, add `/github-webhook/` at the end of URL, and click on `Add webhook`.
  
  http://[jenkins-server-hostname]:8080/github-webhook/

* Prepare a script for Jenkins CI job (covering Unit Test only) and save it as `jenkins-petclinic-ci-job.sh` under `jenkins` folder.

``` bash
echo 'Running Unit Tests on Petclinic Application'
docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.8-openjdk-11 mvn clean test
```

# Jenkins dashboard içerisinde yaptığımız `freestyle job` açıklaması : komutları koşmak için `shell` kullandık

```bash
echo 'Running Unit Tests on Petclinic Application'
docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.8-openjdk-11 mvn clean test
# --rm : oluştuktan sonra container ile işin bitecek, container'ı siler. Exit olunca container gider: docker run --rm it alpine  -- sonrasında -->> ls && printenv HOME && whoami , docker ps -a : container yok!!!
# -w : working directory , docker run ile beraber komut yazabilirim, komutun çalışcağı directory'i gösteriyorum
# -w /app : komut root directory içerisinde olan/yoksa oluşacak olan /app dizini içerisinde çalışacak

docker run --rm -v `pwd`:/app -w /app maven:3.8-openjdk-11 mvn clean ### volume olarak bağladığın `pwd` çıktısı nereyi işaret ediyorsa, yani nerede ve kim olarak çalıştırırsan değişir. Burada yani pwd : /home/ec2-user/petclinic-microservices-with-db-de dizininde çalıştırırsan --->> target klasörlerini siler.

docker run maven:3.8-openjdk-11 mvn clean test -->> ÇALIŞMAZ ÇÜNKÜ POM.XML GÖRMESİ LAZIM
                                               -->> POM.XML ---NEREDE--->> jenkins job çalışınca github repodaki (webhook'dan dolayı) mikroservislere ait mevcut klasör ve dosyalar jenkins-user'ın home dizini olan  /var/lib/jenkins  altındaki  /workspace/petclinic-ci-job içerisine çekilir. Yani POM.XML ---NEREDE--->> /var/lib/jenkins/workspace/petclinic-ci-job
                                              # -->> Bu esnada `pwd`  ---derseniz--->>  /var/lib/jenkins/workspace/petclinic-ci-job  görünür.
- job name : petclinic-ci-job  , aynı zamanda github repodan uygulamaya ait dosya-klasörleri de aldık, job ---çalıştırınca--->> "/var/lib/jenkins/workspace/"  dizini altında  "petclinic-ci-job" dizini oluştu, job burada çalışır.
- Job neden burada çalışır -->> jenkins-server'da freestyle job olarak shell'de komutları yazıyoruz, işi yapan jenkins-user ve kullanacağı dizin de kendi home dizini /var/lib/jenkins.

docker run -v `pwd`:/app -w /app maven:3.8-openjdk-11 mvn clean test :
- `pwd`(jenkins-server içerinde job'un çalıştığı yer = /var/lib/jenkins/workspace/petclinic-ci-job):/app (container'ın root dizininde)  ===>>> pwd'yi volume olarak app'ye bağladık, artık tüm github'dan çektiğimiz tüm (pom.xml dahil) klasör-dosyalar app dizini içerisinde.
- container'ın working directory'si olarak da app dizinini gösterdik.

### mvn clean test  komutuyla birlikte, maven central repodan gerekli tüm  dosyaları container içerisinde komut ile birlikte oluşan lokal repo .m2  klasörüne indirdik. Fakat container root dizininde oluşan bu .m2 klasörü, container ölünce gidecek.
Tekrar bu komutu bir daha çalıştırsam yeniden maven central repodan indirmek için süre geçecek. Peki ne yapmamız lazım : volume ile problemi çözerim.
Container root dizinindeki .m2 dizinini -->> jenkins-server'da job çalıştıran jenkins-user'ın home dizini olan /var/lib/jenkins dizinine bağlarım.

Sonuçta job /var/lib/jenkins/workspace/petclinic-ci-job içerisindeki pom.xml'i görerek çalışacak, lokal repo olarak da jenkins-user'ın HOME dizini olan /var/lib/jenkins içerisindeki .m2 klasörünü kullanacak.

### " .m2 lokal repo " ile " /app = working directory " her ikisi de container içerisinde root içerisinde oluşacak

### $HOME/.m2 -----BAĞLA---->>> /root/.m2       ,      `pwd` -----BAĞLA---->>> /root/app  =  (/app     aynı anlama gelir.)
# container içerisinde /app diye folder'a bağlanacak, böyle bir folder olmasa da oluşacak



###   $HOME  : var/lib/jenkins    ###
### .m2  local repo will become/exist Jenkins'in HOME ###   ---->>>> var/lib/jenkins/.m2
cd                                # ec2-user@jenkins-server# ~:$
pwd                               # /home/ec2-user
whoami                            # ec2-user
sudo usermod -s /bin/bash jenkins # jenkins is not a regular user, give jenkins shell 
sudo su - jenkins                 # switch to jenkins and go to HOME 
whoami                            # jenkins
pwd                               #  /var/lib/jenkins ===>>> jenkins's HOME directory
echo $HOME                        # /var/lib/jenkins
printenv HOME                     # /var/lib/jenkins
ls                                # ............ workspace
cd workspace
ls                                # petclinic-ci-job
cd petclinic-ci-job
ls                                # files and directories from github
```

* Job build ettik ve success aldık --->>> dashboard'da `Coverage Report` oluşuyor, bunu download edip --->>> Tester ve / veya Developer veriyoruz.
*  Java Code Coverage Report içerisinde :Check this to set the build status to failure if the delta coverage thresholds are violated. Delta coverage is the difference between coverage of last successful and current build.
*  ANLAMI : If the current code is less successful than the last code we tested, it means that the bug fixing process in the last code we tested may have failed, or larger bugs may be present in this new code. In other words, the last code we tested is more reliable and accurate than the current code, in short, there is no need to continue testing the new code.

``` bash
# Commit the change, then push the Jenkinsfile to the remote repo.
git add .
git commit -m 'added Jenkins Job for CI pipeline'
git push --set-upstream origin feature/msp-12
git checkout dev
git merge feature/msp-12
git push origin dev
```


# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 13 - Prepare and Implement Selenium Tests***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

***  what I will do until the end of MSP-18 : Step by step I will try each stage of jenkinsfile in a separate job to check whether works.**


``` bash
# Create `feature/msp-13` branch from `dev`.
git checkout dev
git branch feature/msp-13
git checkout feature/msp-13
```

* Create a folder for Selenium jobs with the name of `selenium-jobs` under under `petclinic-microservices-with-db` folder.

``` bash  (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
mkdir selenium-jobs
```

* Unit testleri yaptık şimdi de `QA Automation` test yani fonksiyonel testler yapacağız, Selenium ile yapılacak testler için python ile yazılmış dosyaları `petclinic-microservices-with-db` altında `selenium-jobs` isminde oluşturduğum klasör içerisine koydum.
* Create Selenium job (`QA Automation` test) for testing `Owners >> All` page and save it as `test_owners_all_headless.py` under `selenium-jobs` folder.


* Create Selenium job (`QA Automation` test) for testing `Owners >> All` page and save it as `test_owners_all_headless.py` under `selenium-jobs` folder.

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from time import sleep
import os

# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)
# driver = webdriver.Chrome("/Users/home/Desktop/chromedriver")
# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
# url = "http://localhost:8080"
print(url)
driver.get(url)
sleep(3)
owners_link = driver.find_element("link text", "OWNERS")
owners_link.click()
sleep(2)
all_link = driver.find_element("link text","ALL")
all_link.click()
sleep(2)

# Verify that table loaded
sleep(1)
verify_table = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "table")))

print("Table loaded")

driver.quit()
```

* Create Selenium job (`QA Automation` test) for testing `Owners >> Register` page and save it as `test_owners_register_headless.py` under `selenium-jobs` folder.

``` python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from time import sleep
import random
import os
# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)

# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
print(url)
driver.get(url)
owners_link = driver.find_element("link text", "OWNERS")
owners_link.click()
sleep(2)
all_link = driver.find_element("link text", "REGISTER")
all_link.click()
sleep(2)
# Register new Owner to Petclinic App
fn_field = driver.find_element_by_name('firstName')
fn = 'Callahan' + str(random.randint(0, 100))
fn_field.send_keys(fn)
sleep(1)
fn_field = driver.find_element_by_name('lastName')
fn_field.send_keys('Clarusway')
sleep(1)
fn_field = driver.find_element_by_name('address')
fn_field.send_keys('Ridge Corp. Street')
sleep(1)
fn_field = driver.find_element_by_name('city')
fn_field.send_keys('McLean')
sleep(1)
fn_field = driver.find_element_by_name('telephone')
fn_field.send_keys('+1230576803')
sleep(1)
fn_field.send_keys(Keys.ENTER)

# Wait 10 seconds to get updated Owner List
sleep(10)
# Verify that new user is added to Owner List
if fn in driver.page_source:
    print(fn, 'is added and found in the Owners Table')
    print("Test Passed")
else:
    print(fn, 'is not found in the Owners Table')
    print("Test Failed")
driver.quit()
```

* Create Selenium job (`QA Automation` test) for testing `Veterinarians` page and save it as `test_veterinarians_headless.py` under `selenium-jobs` folder.

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from time import sleep
import os

# Set chrome options for working with headless mode (no screen)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")

# Update webdriver instance of chrome-driver with adding chrome options
driver = webdriver.Chrome(options=chrome_options)

# Connect to the application
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":8080/"
print(url)
driver.get(url)
sleep(3)
vet_link = driver.find_element("link text", "VETERINARIANS")
vet_link.click()

# Verify that table loaded
sleep(5)
verify_table = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "table")))

print("Table loaded")

driver.quit()
```

``` bash
# Commit the change, then push the selenium jobs to the remote repo.
git add .
git commit -m 'added selenium jobs written in python'
git push --set-upstream origin feature/msp-13
git checkout dev
git merge feature/msp-13
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 14 - Create Docker Registry for Dev Manually***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-14` branch from `dev`.
git checkout dev
# git checkout -b feature/msp-14    # YOU CAN ALSO WRITE ONLY THIS INSTEAD OF THE 2 COMMANDS BELOW.
git branch feature/msp-14
git checkout feature/msp-14
```

* Create a ``Jenkins Job`` to create Docker Registry for `dev` on AWS ECR manually.

```yml
- job name: create-ecr-docker-registry-for-dev
- job type: Freestyle project
- Build:
      Add build step: Execute Shell
      Command:
```

```bash
PATH="$PATH:/usr/local/bin"       # AWS CLI introduces the executable path it runs on to the PATH variable, because it was giving errors in old Jenkins versions.
APP_REPO_NAME="clarusway-repo/petclinic-app-dev"
AWS_REGION="us-east-1"

aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \  # First query: is there a repo with this name? otherwise create it.
aws ecr create-repository \
--repository-name ${APP_REPO_NAME} \
--image-scanning-configuration scanOnPush=false \     # `scanOnPush=false` https://docs.aws.amazon.com/cli/latest/reference/ecr/put-image-scanning-configuration.html
--image-tag-mutability MUTABLE \      # When you want to push the image back to the repo, you can push it with the same tag.
--region ${AWS_REGION}
```


* Bu komutları script haline getirip `create-ecr-docker-registry-for-dev.sh` isminde `infrastructure` klasörü altında muhafaza edelim.
* Prepare a script to create Docker Registry for `dev` on AWS ECR and save it as `create-ecr-docker-registry-for-dev.sh` under `infrastructure` folder.

``` bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added script for creating ECR registry for dev'
git push --set-upstream origin feature/msp-14
git checkout dev
git merge feature/msp-14
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 15 - Create a QA Automation Environment with Kubernetes - Part-1***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash
# Create `feature/msp-15` branch from `dev`.
git checkout dev
git branch feature/msp-15
git checkout feature/msp-15
```

- Create a folder for kubernetes infrastructure with the name of `dev-k8s-terraform` under the `infrastructure` folder.

- Prepare a terraform file for kubernetes Infrastructure consisting of 1 master, 2 Worker Nodes and save it as `main.tf` under the `infrastructure/dev-k8s-terraform`.

* We have done unit tests, now we will do 'QA Automation' tests, that is, functional tests, so the application needs to be deployed and to be ready for test. So, I will set up a 'cluster' consisting of 1 master and 2 workers. 3 `Ubuntu 22.04` machines. To do this, I will install it with 'kubeadm', which is almost never used in the market.

  * `S1-kubernetes-01-installing-on-ec2-linux2` klasörü içerisindeki README'den manuel olarak yaptığımız kurulumu ansible ile yapacağız, fakat sadece ec2-instance'ları terraform (user-data olmadan boş makina) ile ayağa kaldıracağız.

* `Terraform` dosyası ile makinaları ve security grupları ayağa kaldıracağız ve  buradaki maksat `1 master ve 2 worker node` kaldırmak için yazdığımız terraform dosyası çalışıyor mu kontrol etmek.

  * Daha sonrasında MSP-18 içinde `terraform` dosyasını kullanarak bu infrastructure oluşturup ec2-instance'ları ayağa kaldırıp, `Ansible playbook` ile cluster oluşturacağım (userdata içerisindeki komutlar) ve uygulamayı ayağa kaldıracağım.

# security groups : <https://kubernetes.io/docs/reference/networking/ports-and-protocols/>

  ***IF TWO DIFFERENT/SEPERATE SECURITY GROUPS NEED TO BE CONNECTED TO EACH OTHER AT THE SAME TIME IN TERRAFORM YOU WILL USE-->> `MUTUAL`***
  * Implicity dependency --->>> we can't use

  * Flannel (8472) : In the playbook we will install flannel so will provide network for nodes

  * self: This ensures that the ec2-instances to which secgrp is connected can only communicate with each other on that port, it does not open it to the outside, that is, it cannot be accessed from the outside. https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group.html#self
                    mutual : 10250 (self) , 8472 (self) , 2379-2380 (self) 
                    worker : 22 , 3000-32767
                    master : 22 , 3000-32767 , 6443 , 10257 (self) , 10259 (self)

  ### Control-plane (Master) Node(s)

      |Protocol|Direction|Port Range |               Purpose                     |     Used By        |
      |--------|---------|-----------|-------------------------------------------|--------------------|
      |TCP     |Inbound  |6443       |Kubernetes API server                      |       All          |

      |TCP     |Inbound  |2379-2380  |`etcd` server client API                   |kube-apiserver, etcd| *** If there was more than 1 master node, we would have to make this port mutual, there is only 1 here, there was no need, but we made it mutual for the sake of real using.***

      |TCP     |Inbound  |10250      |Kubelet API                                |Self, Control plane |
      |TCP     |Inbound  |10259      |kube-scheduler                             |Self                |
      |TCP     |Inbound  |10257      |kube-controller-manager                    |Self                |
      |TCP     |Inbound  |22         |remote access with ssh                     |      All           |
      |UDP     |Inbound  |8472       |Cluster-Wide Network Comm. - Flannel VXLAN |Self                |

      |TCP     |Inbound  |30000-32767|NodePort Services†                         |          All       | *** (30000-32767)WE OPENED THIS PORT IN MASTER TO SEE THE APPLICATION FROM MASTER TOO.*** It wasn't necessary but in this example we did it this way.


  ### Worker Node(s)

      |Protocol|Direction|Port Range |         Purpose                           |       Used By      |
      |--------|---------|-----------|-------------------------------------------|--------------------|
      |TCP     |Inbound  |10250      |Kubelet API                                |Self, Control plane |

      |TCP     |Inbound  |30000-32767|NodePort Services†                         |         All        | *** TO SEE THE APPLICATION FROM workers

      |TCP     |Inbound  |2379-2380  |`etcd` server client API                   |kube-apiserver, etcd| *** it is not for workers, explaining is above

      |TCP     |Inbound  |22         |remote access with ssh                     |        All         |
      |UDP     |Inbound  |8472       |Cluster-Wide Network Comm. - Flannel VXLAN |Self                |


# role --->>> instance profil --->>> makinaya profil ataması

  * Ayağa kalkan makinaların s3'e ulaşmasına izin veren bir role oluşturdum. Master Node'a uygulamayı deploy ederken `Helm repo` olarak `s3` kullanabilmek için yetkilendirme amaçlı

    resource "aws_iam_role" "petclinic-master-server-s3-role"
    managed_policy_arns = ["arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"]

  * yukarıdaki rolü bağladığım profil oluşturdum

    resource "aws_iam_instance_profile" "petclinic-master-server-profile"

  * oluşturduğum profili makinalara bağladım (terraform ile direkt olarak makinalara role ataması yapamıyoruz)

    iam_instance_profile = aws_iam_instance_profile.petclinic-master-server-profile.name

# subnet_id = "subnet-c41ba589"  # select own subnet_id of us-east-1a `change`

# key_name = "clarus" `BURASI DEĞİŞMEYECEK` daha sonra `sed (sed -i "s/clarus/$ANS_KEYPAIR/g" main.tf)` komutu ile değiştireceğim zaten

#     tags = {                      `Ansible'da dinamic inventory , filter  ve  keyed groups`

        Name = "kube-master"
        Project = "tera-kube-ans"
        Role = "master"
        Id = "1"
        environment = "dev"
    }


- Create a folder for kubernetes infrastructure with the name of `dev-k8s-terraform` under the `infrastructure` folder.

- Prepare a terraform file for kubernetes Infrastructure consisting of 1 master, 2 Worker Nodes and save it as `main.tf` under the `infrastructure/dev-k8s-terraform`.
*** `subnet_id = "subnet-c41ba589" `  # select own subnet_id of us-east-1a (kube-master , worker-1, worker-2)`CHANGE`**
- My own subnet_id of us-east-1a is `subnet_id = "subnet-069d7f45d2659c70c `

```go
provider "aws" {
  region  = "us-east-1"
}

variable "sec-gr-mutual" {
  default = "petclinic-k8s-mutual-sec-group"
}

variable "sec-gr-k8s-master" {
  default = "petclinic-k8s-master-sec-group"
}

variable "sec-gr-k8s-worker" {
  default = "petclinic-k8s-worker-sec-group"  
}

data "aws_vpc" "name" {
  default = true
}

resource "aws_security_group" "petclinic-mutual-sg" {
  name = var.sec-gr-mutual
  vpc_id = data.aws_vpc.name.id       // if you do not write anything here, it will use default VPC. YOU DO NOT NEED TO WRITE THIS LINE

  ingress {
    protocol = "tcp"
    from_port = 10250
    to_port = 10250
    self = true
  }

    ingress {
    protocol = "udp"
    from_port = 8472
    to_port = 8472
    self = true
  }

    ingress {
    protocol = "tcp"
    from_port = 2379
    to_port = 2380
    self = true
  }

}

resource "aws_security_group" "petclinic-kube-worker-sg" {
  name = var.sec-gr-k8s-worker
  vpc_id = data.aws_vpc.name.id


  ingress {
    protocol = "tcp"
    from_port = 30000
    to_port = 32767
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    protocol = "tcp"
    from_port = 22
    to_port = 22
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress{
    protocol = "-1"
    from_port = 0
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "kube-worker-secgroup"
  }
}

resource "aws_security_group" "petclinic-kube-master-sg" {
  name = var.sec-gr-k8s-master
  vpc_id = data.aws_vpc.name.id

  ingress {
    protocol = "tcp"
    from_port = 22
    to_port = 22
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    protocol = "tcp"
    from_port = 6443
    to_port = 6443
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    protocol = "tcp"
    from_port = 10257
    to_port = 10257
    self = true
  }

  ingress {
    protocol = "tcp"
    from_port = 10259
    to_port = 10259
    self = true
  }

  ingress {
    protocol = "tcp"
    from_port = 30000
    to_port = 32767
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    protocol = "-1"
    from_port = 0
    to_port = 0
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "kube-master-secgroup"
  }
}

resource "aws_iam_role" "petclinic-master-server-s3-role" {     // when we deploy application we will use S3 as a helm repo
  name               = "petclinic-master-server-role"           // ec2 (role - profile)  --AmazonS3ReadOnlyAccess-->> S3
  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF

  managed_policy_arns = ["arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"]
}

resource "aws_iam_instance_profile" "petclinic-master-server-profile" {
  name = "petclinic-master-server-profile"
  role = aws_iam_role.petclinic-master-server-s3-role.name
}

resource "aws_instance" "kube-master" {
    ami = "ami-053b0d53c279acc90"
    instance_type = "t3a.medium"
    iam_instance_profile = aws_iam_instance_profile.petclinic-master-server-profile.name
    vpc_security_group_ids = [aws_security_group.petclinic-kube-master-sg.id, aws_security_group.petclinic-mutual-sg.id]
    key_name = "clarus"
    subnet_id = "subnet-c41ba589"  // select own subnet_id of us-east-1a. Data will be transfered between instances, if the instances are in the different AZs it will cost more
    availability_zone = "us-east-1a"
    tags = {                      // Ansible dinamic inventory , filter and keyed groups -->> ansible playbook host
        Name = "kube-master"
        Project = "tera-kube-ans"
        Role = "master"
        Id = "1"
        environment = "dev"
    }
}

resource "aws_instance" "worker-1" {
    ami = "ami-053b0d53c279acc90"     // ubuntu 22.04
    instance_type = "t3a.medium"
    vpc_security_group_ids = [aws_security_group.petclinic-kube-worker-sg.id, aws_security_group.petclinic-mutual-sg.id]
    key_name = "clarus"
    subnet_id = "subnet-c41ba589"  // select own subnet_id of us-east-1a
    availability_zone = "us-east-1a"
    tags = {
        Name = "worker-1"
        Project = "tera-kube-ans"
        Role = "worker"
        Id = "1"
        environment = "dev"
    }
}

resource "aws_instance" "worker-2" {
    ami = "ami-053b0d53c279acc90"
    instance_type = "t3a.medium"
    vpc_security_group_ids = [aws_security_group.petclinic-kube-worker-sg.id, aws_security_group.petclinic-mutual-sg.id]
    key_name = "clarus"
    subnet_id = "subnet-c41ba589"  // select own subnet_id of us-east-1a
    availability_zone = "us-east-1a"
    tags = {
        Name = "worker-2"
        Project = "tera-kube-ans"
        Role = "worker"
        Id = "2"
        environment = "dev"
    }
}

output kube-master-ip {
  value       = aws_instance.kube-master.public_ip
  sensitive   = false
  description = "public ip of the kube-master"
}

output worker-1-ip {
  value       = aws_instance.worker-1.public_ip
  sensitive   = false
  description = "public ip of the worker-1"
}

output worker-2-ip {
  value       = aws_instance.worker-2.public_ip
  sensitive   = false
  description = "public ip of the worker-2"
}
```

```bash
# Commit the change, then push the cloudformation template to the remote repo.
git add .
git commit -m 'added dev-k8s-terraform  for kubernetes infrastructure'
git push --set-upstream origin feature/msp-15
git checkout dev
git merge feature/msp-15
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 16 - Create a QA Automation Environment with Kubernetes - Part-2***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash
# Create `feature/msp-16` branch from `dev`.
git checkout dev
git branch feature/msp-16
git checkout feature/msp-16
git push --set-upstream origin feature/msp-16
```

* Create a ``Jenkins Job`` to test `bash` scripts creating QA Automation Infrastructure for `dev` manually.
  * We check whether there are 'ansible and terraform' in Jenkins-server. If we had used an agent, we would have done the same check for the agent.

```yml
- job name: test-creating-qa-automation-infrastructure
- job type: Freestyle project
- GitHub project: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */feature/msp-16
- Build Environment: Add timestamps to the Console Output
- Build:
      Add build step: Execute Shell
      Command:
```

```bash
echo $PATH
whoami
PATH="$PATH:/usr/local/bin"
python3 --version
pip3 --version
ansible --version
aws --version
terraform --version
```

Click `Save`--->>> `Build Now`

* After running the job above, replace the script with the one below in order to test creating key pair for `ansible`. (Click `Configure`)

* Job configure --->>> We create a key-pair in Amazon and save it in a file, which we will use when creating and launching the machines with terraform apply in the job, and when connecting via ssh when managing managed/host nodes with Ansible. You can see it by using the commands below in Jenkin-server or via AWS Console.

```bash
PATH="$PATH:/usr/local/bin"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
AWS_REGION="us-east-1"
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR} --query "KeyMaterial" --output text > ${ANS_KEYPAIR}
chmod 400 ${ANS_KEYPAIR}
```
Click `Save`--->>> `Build Now`

```bash (pwd : home/ec2-user)
sudo su - jenkins
pwd # /var/lib/jenkins
cd workspace
ls  # test-creating-qa-automation-infrastructure (job's name is the directory name)
cd test-creating-qa-automation-infrastructure
ls    # petclinic-ansible-test-dev.key  IS HERE.
```


* After running the job above, replace the script with the one below in order to test creating kubernetes infrastructure with terraform. (Click `Configure`)
* Makinaları terraform apply ile aya kaldıran job çalıştır.

```bash
PATH="$PATH:/usr/local/bin"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
AWS_REGION="us-east-1"
cd infrastructure/dev-k8s-terraform   # main.tf ? -->> jenkins-server -->> var/lib/jenkinsworkspace/test-creating-qa-automation-infrastructure 
sed -i "s/clarus/$ANS_KEYPAIR/g" main.tf  # It replaces "clarus" in main.tf with $ANS_KEYPAIR, that is, "petclinic-ansible-test-dev.key".
terraform init
terraform apply -auto-approve -no-color # apply --change-->> destroy : If you rebuild the job, master and worker nodes will be destroyed.
```

Click `Save`--->>> `Build Now`

* Bu job'tan sonra AWS Management Console üzerinde makinaları gör, makinalar hazır olunca da master veya worker node'lardan bir tanesine mauel olarak ssh bağlan, fakat dikkat et aws'den ssh komutunu kopyalarken .pem uzantısını silmeyi UNUTMA -->> petclinic-ansible-test-dev.key

```bash (pwd : /var/lib/jenkins/workspace/test-creating-qa-automation-infrastructure)
ssh -i "petclinic-ansible-test-dev.key.pem" ubuntu@ec2-54-157-248-246.compute-1.amazonaws.com
```

* After running the job above, replace the script with the one below in order to test SSH connection with one of the instances.(Click `Configure`)

* Let's connect via SSH using the private IP of one of the instances: The aim is to see that we can connect to managed/host nodes (masters and workers) with jenkins-server, which also serves as 'ansible-control'.

# `${WORKSPACE}` I wrote this variable without introducing it in the following job: IT EXISTS IN THE DEFAULT VARIABLES DEFINED IN JENKİNS-SERVER.

  # <http://jenkins-server-hostname:8080/env-vars.html/>

  # ${WORKSPACE} : variable defined in jenkins : workspace/job's-name

# -o UserKnownHostsFile=/dev/null : do not record in known-host , known-host dosyası çok dolmasın şişmesin diye
# -o StrictHostKeyChecking=no : When you try to connect via SSH, it does not ask for a yes/no response to the desired confirmation.
# hostname : If you use only the Ssh connection command, you will connect to ec2-instance, but if you add another command to the end of the ssh command, it will allow us to run only that command without connecting to ec2-instance. But this command runs in ec2-instance. hostname command gives us the ec2-instance IP. We will see the IP in the console output of job.

```bash
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -i ${WORKSPACE}/${ANS_KEYPAIR} ubuntu@172.31.91.243 hostname # Change IP , public or private IP of master or worker node
```

Click `Save`--->>> `Build Now`

* Create a folder for ansible jobs under the `petclinic-microservices-with-db-de` folder.

```bash
mkdir -p ansible/inventory
```

* Prepare static inventory file with name of `hosts.ini` for Ansible under `ansible/inventory` folder using Docker machines private IP addresses.
* Çok basit bir inventory dosyası oluşturuyorum, sadece ping atıp/atamadığımı job  çalıştırarak görmek istiyorum.

# Derste control_node içerisinde ```/etc/ansible/hosts``` dosyasına ekleme yaparak `temel` seviyede `inventory` oluşturmuştuk. Buradaki dosya adı hosts değil de `hosts.ini` oldu

# Yine derste `/home/ec2-user/etc/ansible/ansible.cfg`  İÇERİSİNE hosts BİLGİLERİNİ YAZMAK YERİNE, `AYRI` BİR YERDE inventory OLUŞTURMUŞTUK, dosya adını da `kendi inventory ==>> inventory.txt` koymuştuk. Dosya adının uzantılı olmasına gerek yok, uzantısının ne olduğunun önemi de yok

```ini
172.31.91.243   ansible_user=ubuntu # MAKİNALARIN PRIVATE IP'LERİNİ GİRİNİZ.
172.31.87.143   ansible_user=ubuntu
172.31.90.30    ansible_user=ubuntu
```

```bash
# Commit the change, then push to the remote repo.
git add .
git commit -m 'added ansible static inventory host.ini for testing'
git push --set-upstream origin feature/msp-16
```

* Configure `test-creating-qa-automation-infrastructure` job and replace the existing script with the one below in order to test ansible by pinging static hosts.
* `Staic Inventory` ile makinalara ping atmayı deniyorum.
* `Ansible config dosyası` çalışma önceliklendirmesi :

# <https://docs.ansible.com/ansible/latest/reference_appendices/config.html>

             1-   ANSIBLE_CONFIG (environment variable if set) Terminalde ansible cofig ile ilgili environment variable belirlediysen
             2-   ansible.cfg (in the current directory)  -->> job'un çalıştığı klasörde ansible.cfg varsa
             3-   ~/.ansible.cfg (in the home directory)
             4-   /etc/ansible/ansible.cfg  -->> biz ansible kurulumunu `pip` ile yaptık yani default olarak ansible.cfg oluşmadı

* Bizim terraform ile yaptığımız kurulumda ansible kurulumunu `pip` ile yaptığımız için `BU KURULUMLA "config file = None" , YANİ "config" DOSYASI OLUŞMUYOR. "config file = ansible.cfg"  BİZ KENDİMİZ OLUŞTURMAMIZ GEREKİYOR`
  - terminalde ansible --version yazdığında  -->> config file = None
* Biz bu kurulumda şimdiye kadar ansible config dosyası oluşturmadık, biz sadece `hosts.ini` adında basit bir inventory dosyası oluşturduk. İŞTE BU YÜZDEN AŞAĞIDAKİ JOB İÇERİSİNDE `3 TANE DEĞİŞKEN EXPORT ETTİK`
```bash
# Bu değişken isimlerini hangi dökümandan alıyorum : https://docs.ansible.com/ansible/latest/reference_appendices/config.html
# Aynı değişkenleri terminalde de görebiliriz : ansible-config  list | grep -i private_key
PATH="$PATH:/usr/local/bin"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"

export ANSIBLE_INVENTORY="${WORKSPACE}/ansible/inventory/hosts.ini"   # inventory dosyamı tanıtıyorum
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"         # ssh ile control-node'lara bağlanmak için keypair tanıtıyorum
export ANSIBLE_HOST_KEY_CHECKING=False
ansible all -m ping
```

Click `Save`--->>> `Build Now`

* Prepare `dynamic inventory` file with name of `dev_stack_dynamic_inventory_aws_ec2.yaml` for Ansible under `ansible/inventory` folder using ec2 instances private IP addresses.

# dosya sonu `aws_ec2.yaml` OLMALI

# main.tf -->> `tag`
        Name          = "kube-master"
        Project       = "tera-kube-ans"
        Role          = "master"
        Id            = "1"
        environment   = "dev"

```yaml (dynamic inventory  :  dev_stack_dynamic_inventory_aws_ec2.yaml)
plugin: aws_ec2 # tek zorunlu kısım burası , diğer satırları yoruma al çalışır
regions:
  - "us-east-1"
filters:  # https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html#options --->>> tag:<key>                    
  tag:Project: tera-kube-ans    # main.tf içindeki `tag`   --->>> tag:<key>  
  tag:environment: dev          # main.tf içindeki `tag`   --->>> tag:<key>  
  instance-state-name: running  # sadece çalışanlar
keyed_groups:
  - key: tags['Project']        # Project tag'i olanları grupla ==>> 3 makinada da var
    # - key: tags.Project       # BU ŞEKİLDE DE YAZIMI DOĞRU
    prefix: 'all_instances'     # prefix yazmassan --->> alt çizgi ile başlıyor "_tera-kube-ans" gibi , yazarsan --->>> "all_instances_tera-kube-ans"
    # seperator: for            # Buraya yazdığımız ifade "alt çizgi yerini alır"  ===>>> all_instancesfortera-kube-ans
  - key: tags['Role']
    prefix: 'role'
hostnames:
  - "ip-address"
compose:  # ansible playbook içerisinde kullanmak istediğin değişkeni buraya yazarsın. https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
  ansible_user: "'ubuntu'"    # ansible facts içinde ubuntu diye bir ifade yok o yüzden çift tırnak içine tek tırnakla yazarak string ifade olarak kullanıldı
  # ansible_host: public_ip_address     : public_ip_address  ifadeleri gathering facts/ansible facts
  # playbook içinde kullanalım-->> msg: "Host is {{ ansible_host }}"     # Host is "public ip   ne ise o ifade gelir buraya"
```

```bash
# Commit the change, then push the cloudformation template to the remote repo.
git add .
git commit -m 'added ansible dynamic inventory files for dev environment'
git push
```

* Configure `test-creating-qa-automation-infrastructure` job and replace the existing script with the one below in order to check the Ansible dynamic inventory for `dev` environment. (Click `Configure`)
* `Dynamic Inventory` ile makinaları görebiliyormuyum kontrolünü yapıyorum.

```bash
APP_NAME="Petclinic"  # bu başka örnekten kalmış
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-inventory -v -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml --graph
# -v : verbose = bilgi, v sayısı artıkça bilgi de artar
# -i inventory dosyamın yerini gösteriyorum
```

Click `Save`--->>> `Build Now`

* After running the job above, replace the script with the one below in order to test all instances within dev dynamic inventory by pinging static hosts. (Click `Configure`)
* `Dynamic Inventory` kullanarak makinalara ping atmayı deniyorum.

```bash
# Test dev dynamic inventory by pinging
APP_NAME="Petclinic"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
ansible -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml all -m ping
```

Click `Save`--->>> `Build Now`

### Ansible Facts - Gathering Facts:
```bash
# job'u configure et ve çalıştır. 11 nolu job console output incele.
APP_NAME="Petclinic"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
ansible -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml role_master -m setup
```


* Create an ansible playbook to install kubernetes and save it as `k8s_setup.yaml` under `ansible/playbooks` folder.
  - önce playbooks isimli folder oluştur.
* Bu playbook ile ayağa kaldırdığımız makinalara cluster kuracağız, daha önce derste (S1-kubernetes-01-installing-on-ec2-linux2) terraform ile yaptığımızı ansible'a dönüştürdük.
# https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html
# https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html
# https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html

```yml  (ansible playbook  :  k8s_setup.yaml)
- hosts: all    #  master, worker-1 and worker-2
  become: true  # root authority
  tasks:

  - name: change hostnames    # https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
    shell: "hostnamectl set-hostname {{ hostvars[inventory_hostname]['private_dns_name'] }}" 
    # ansible facts içerisinden variables çekebiliyorsunuz : "magic variables " , makinalara hostname olarak içinde çalıştığı makinanın private_dns_name atayacak

  - name: swap off  # linux işletim sisteminde işlem yaparken RAM yetersiz geldiğinde ihtiyaç duyduğunu harddisk'ten yiyerek karşılıyor, bu da performans düşüklüğüne sebep oluyor. Kubeadm de bu durumu önermiyor ve kubelet'in düzgün çalışması için "swap off" yapın diyor. 
    shell: |
      free -m
      swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

  - name: Enable the nodes to see bridged traffic
    shell: |
      cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward                 = 1
      EOF
      sysctl --system

  - name: update apt-get
    shell: apt-get update

  - name: Install packages that allow apt to be used over HTTPS
    apt:
      name: "{{ packages }}"
      state: present
      update_cache: yes
    vars:
      packages:
      - apt-transport-https  # https ile paket yükleyebilmek için , curl komutu ile indirme yaparken https olan sayfaları kullanabilmek için
      - curl
      - ca-certificates     # key indirmek için gerekli, aşağıdaki komutlarda indireceğimiz key'ler için

  - name: update apt-get and install kube packages
    shell: |
      curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
      echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
      apt-get update -q && \
      apt-get install -qy kubelet=1.26.3-00 kubectl=1.26.3-00 kubeadm=1.26.3-00 kubernetes-cni docker.io
      apt-mark hold kubelet kubeadm kubectl

  - name: Add ubuntu to docker group
    user:
      name: ubuntu
      group: docker

  - name: Restart docker and enable
    service:
      name: docker
      state: restarted
      enabled: yes

  # change the Docker cgroup driver by creating a configuration file `/etc/docker/daemon.json` 
  # and adding the following line then restart deamon, docker and kubelet
### There two cgroup drivers on Linux : 1- Systemd 2- cgroupfs , before version 1.22 kubelet and kubeadm used cgroupfs, but now they use containerd. But by default containerd still use cgroupfs. So we need to change default value to Systemd.
  - name: change the Docker cgroup  # https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/
                                    # https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
    shell: |
      mkdir /etc/containerd
      containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
      sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

  - name: Restart containerd and enable
    service:
      name: containerd
      state: restarted
      enabled: yes


- hosts: role_master
  tasks:
      
  - name: pull kubernetes images before installation # kubernetes'de arka planda sistem için çalışan pod'ların image'larını çekiyoruz. 
    become: yes
    shell: kubeadm config images pull

  - name: initialize the Kubernetes cluster using kubeadm
    become: true
    shell: |
      kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=All
    
  - name: Setup kubeconfig for ubuntu user  # Bir kubectl ile bir makineden cluster'ı nasıl kontrol ediyoruz : ".kube" dizini içindeki "config" ile 
    become: true
    command: "{{ item }}"
    with_items:
     - mkdir -p /home/ubuntu/.kube  # master node içerisinde .kube dizini oluştur.
     - cp -i /etc/kubernetes/admin.conf /home/ubuntu/.kube/config # cluster'ı initialize (başlatınca) edince default olarak "/etc/kubernetes/" altında "admin.conf" dosyası oluştu ----bunu kopyala--->>> /home/ubuntu/.kube/config
     - chown ubuntu:ubuntu /home/ubuntu/.kube/config

  - name: Install flannel pod network # network interface olarak "flannel" kullandık burada.
    remote_user: ubuntu
    shell: kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

  - name: Generate join command # TOKEN ile worker'ları cluster'a dahil etmem lazım
    become: true
    command: kubeadm token create --print-join-command  # TOKEN alacak komut, bunun sonucunda çıkanı (bir komut çıkacak) worker'da girince cluster'a dahil olacak
    register: join_command_for_workers # yukarıdaki komut çıktısını register ile bir değişkene (join_command_for_workers) atıyorum

  - debug: msg='{{ join_command_for_workers.stdout.strip() }}' # yukarıdaki değişkene atadığım çıktıyı görmek için, sağını solunu da kırptım

  - name: register join command for workers # yazdığım bu play role_master için, yani burada yukarıda elde ettiğim ve worker'lara göndermem gereken komut çıktısını role_worker makinaya gönderemem.
    add_host:   # Bu modülü kullanarak "kube_master" isminde bir host ekliyorum ve aşağıda bu host için bir değişken tanımlıyorum. Bu modül yerine "set_fact" kullansan da olur.
      name: "kube_master" # "kube_master" isminde bir host ekliyorum
      worker_join: "{{ join_command_for_workers.stdout.strip() }}" # "kube_master" ismindeki host için " worker_join" isminde bir değişken tanımlıyorum ki --->>> tüm playbook içerisinde kullanabileceğim bir değişken haline gelmiş oluyor, aslında ansible fact haline dönüşmüş gibi oluyor. Dolayısıyla aşağıdaki "role_worker" için yazılan play içerisinde de geçerli olacak ve çağırabileceğim.

  - name: install Helm # kube-master'da helm kullanacağım, Kubernetes ortamında uygulamayı çalıştıran bütün servislerimize ait manifesto dosyalarımızı helm ile paketleyeceğiz, S3'e göndereceğim ve buradan da helm ile chart'larımızı çekeceğiz.
    shell: |
      cd /home/ubuntu
      curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
      chmod 777 get_helm.sh
      ./get_helm.sh

- hosts: role_worker  # worker'ları cluster'a dahil ediyoruz.
  become: true
  tasks:

  - name: Join workers to cluster # yukarıda değişkene atayarak elde ettiğim komut çıktısını çağırıp koşturuyorum.
    shell: "{{ hostvars['kube_master']['worker_join'] }}"
    register: result_of_joining   # Bu task sonucunu register ile bir değişkene atıyorum

  - debug: msg='{{ result_of_joining.stdout }}' # Yukarıdaki sonucu console'da görüyorum.
```

```bash
# Commit the change, then push the ansible playbooks to the remote repo.
git add .
git commit -m 'added ansible playbooks for dev environment'
git push
```

* Configure `test-creating-qa-automation-infrastructure` job and replace the existing script with the one below in order to test the playbooks to create a Kubernetes cluster. (Click `Configure`)
* Ansible playbook ile cluster kurmak için job çalıştır.

```bash
APP_NAME="Petclinic"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
PATH="$PATH:/usr/local/bin"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
# k8s setup
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml ./ansible/playbooks/k8s_setup.yaml
```

Click `Save`--->>> `Build Now`


master'a ssh ile bağlan
```bash
kubectl get no
kubectl get po -A  # sistem podlarını görmek için
kubectl get po --all-namespaces # namespace farketmeksizin tüm pod'ları listeler.
kubectl get po -n kube-system # kube-system isimli namespace içerisindeki podları gösterir.
```

# yukarıda job'lar ile yaptıklarımızı geri alalım

* job'u aşağıdaki gibi configure edelim : Oluşturulan kaynakları silelim ki sonraki büyük pipeline ile yeniden oluşacak kaynaklar ile çakışmasın.

```bash
cd infrastructure/dev-k8s-terraform
terraform destroy -auto-approve -no-color
```

Click `Save`--->>> `Build Now`

* After running the job above, replace the script with the one below in order to test deleting existing key pair using AWS CLI with following script. (Click `Configure`) job'u aşağıdaki gibi configure edelim :

```bash
PATH="$PATH:/usr/local/bin"
ANS_KEYPAIR="petclinic-ansible-test-dev.key"
AWS_REGION="us-east-1"
aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR}
rm -rf ${ANS_KEYPAIR}
```

Click `Save`--->>> `Build Now`

* Bu job'lar içerisindeki komutları script haline getirelim ve arşiv yapalım.

* Create a script to create QA Automation infrastructure and save it as `create-qa-automation-environment.sh` under `infrastructure` folder. (This script shouldn't be used in one time. It should be applied step by step like above)

```bash
# Environment variables
PATH="$PATH:/usr/local/bin"
APP_NAME="Petclinic"
ANS_KEYPAIR="petclinic-$APP_NAME-dev-${BUILD_NUMBER}.key"
AWS_REGION="us-east-1"
export ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
export ANSIBLE_HOST_KEY_CHECKING=False
# Create key pair for Ansible
aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR} --query "KeyMaterial" --output text > ${ANS_KEYPAIR}
chmod 400 ${ANS_KEYPAIR}
# Create infrastructure for kubernetes
cd infrastructure/dev-k8s-terraform
terraform init
terraform apply -auto-approve -no-color
# Install k8s cluster on the infrastructure
ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml ./ansible/playbooks/k8s_setup.yaml
# Build, Deploy, Test the application
# Tear down the k8s infrastructure
cd infrastructure/dev-k8s-terraform
terraform destroy -auto-approve -no-color
# Delete key pair
aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR}
rm -rf ${ANS_KEYPAIR}
```

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added scripts for qa automation environment'
git push
git checkout dev
git merge feature/msp-16
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 17 - Prepare Petlinic Kubernetes YAML Files***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-17` branch from `dev`.
git checkout dev
git branch feature/msp-17
git checkout feature/msp-17
```

* Create a folder with name of `k8s` under `petclinic-microservices-with-db` folder for keeping the manifest files of Petclinic App on Kubernetes cluster.
* Create a `docker-compose.yml` under `k8s` folder with the following content as to be used in conversion the k8s files.

* Install [conversion tool](https://kompose.io/installation/) named `Kompose` on your Jenkins Server. [User Guide](https://kompose.io/user-guide/#user-guide)

# Uygulamayı çalıştırmak için bize manifesto dosyaları (deployment ve service yaml dosyaları gibi) lazım. `kompose` ismindeki tool, elimizde bulunan docker-compose.yml dosyasını kullanarak kubernetes object'lerini oluşturmamız için gerekli yaml dosyalarını oluşturmamıza olanak sağlar

# <https://kompose.io/>

# <https://kompose.io/conversion/>

# <https://kompose.io/user-guide/>  (Labels)

# Herşeyi dönüştüremiyor. Aşağıdaki komut girdikten sonra oluşan yaml dosyalarına eklemeler girmek gerekebiliyor. `https://kompose.io/conversion/`

# Docker-compose.yml  ile kubernetes object'lerini oluşturan yml dosyaları tamamen örtüşmeyebiliyor, kullandığımız docker-compose.yml dosyasına ekleme yapmamız gerekebilir. `https://kompose.io/user-guide/`  (Labels)

```bash
# Bu komut ile dönüşüm yapılıyor.
kompose convert -f docker-compose.yml
```

# kompose anlatmak için repodan project_203 -> solutions-files içindeki docker-compose.yml'ı al ve kullan, ana dizinde test adında klasör oluştur ve içinde çalış. `https://kompose.io/user-guide/`  (Labels)
  - aşağıdaki örneği ``` kompose convert -f docker-compose.yml ``` komutu ile çalıştırdıktan sonra `networkpolicy.yaml` da oluşuyor, bunun ile sadece belirlediğimiz podların kendi aralarında görüşmelerini sağlayabiliriz, normalde kubernetes'de sınırlama yok her pod diğeriyle görüşebilir.

```yml (pwd:test) (docker-compose.yml)
labels: # myapp : altında ports: ile aynı hizada
  kompose.service.type: "nodeport"
  kompose.service.nodeport.port: "30001"  # Bu değişikliği yaptıktan sonra tekrar convert edersen eğer oluşur.
  kompose.service.expose: "devopsarrow.com" # bunu da eklersen ingress de oluşur
```


```yaml (docker-compose.yml)
version: '3'
services:
  config-server:
    image: "{{ .Values.IMAGE_TAG_CONFIG_SERVER }}"  # helm kullandığımızda, oluşan chart (phonebook-chart  gibi) klasörü içerisinde "values.yaml" dosyamızda değişken tanımlaması yapıyoruz ki güncelleme yapmak istediğimizde her defasında aynı chart klasörü altında oluşan "templates" klasörü içerisindeki "deployment.yml" dosyası içindeki "image ismi" değiştirmekle uğraşmayalım. Bunun yerine "values.yaml" dosyası içerisindeki ismi değiştirelim. 
    # values.yml dosyası içinde     -->> resultserver_image: alparslanu6347/result-server_app
    # deployment.yml dosyası içinde -->> - image: {{ .Values.resultserver_image }}
    ports:
     - 8888:8888
    labels:   # https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#registry-secret-existing-credentials
      kompose.image-pull-secret: "regcred"  # register credentials , Her bir pod ECR repodan/repoya image push/pull edecek, ama AWS ECR özel bir repo olduğundan podların erişim yetkisi yok, AWS ECR login komutu kullanarak docker log in yapıyorduk, bize .docker isimli bir dosyanın içinde credentials oluşturuyordu, o credentials ile log in olduktan sonra image push/pull edebiliyorduk. İŞTE O CREDENTIALS İÇİN "regcred" İSİMLİ BİR SECRET OLUŞTURACAĞIM DAHA SONRA, BURADA İSE POD'A OLUŞTURDUĞUM (SONRAKİ ADIMLARDA OLUŞTURACAĞIM) O SECRET'I KULLAN DİYORUM.
  discovery-server:
    image: "{{ .Values.IMAGE_TAG_DISCOVERY_SERVER }}"
    ports:
     - 8761:8761
    labels:
      kompose.image-pull-secret: "regcred"
  customers-service:
    image: "{{ .Values.IMAGE_TAG_CUSTOMERS_SERVICE }}"
    deploy:
      replicas: 2
    ports:
    - 8081:8081
    labels:
      kompose.image-pull-secret: "regcred"
  visits-service:
    image: "{{ .Values.IMAGE_TAG_VISITS_SERVICE }}"
    deploy:
      replicas: 2
    ports:
     - 8082:8082
    labels:
      kompose.image-pull-secret: "regcred"
  vets-service:
    image: "{{ .Values.IMAGE_TAG_VETS_SERVICE }}"
    deploy:
      replicas: 2
    ports:
     - 8083:8083
    labels:
      kompose.image-pull-secret: "regcred"
  api-gateway:    # BU service'de diğerlerine göre farklılık var : UYGULAMANIN GELDİĞİ ANA SAYFA BURASI, selenium testler de bu service üzerinden test yapabilir
    image: "{{ .Values.IMAGE_TAG_API_GATEWAY }}"
    deploy:
      replicas: 1
    ports:
     - 8080:8080
    labels:
      kompose.image-pull-secret: "regcred"
      kompose.service.expose: "{{ .Values.DNS_NAME }}"  # ingress dosyası oluşturuyor.
      kompose.service.type: "nodeport"        # service tipini nodeport verdim.
      kompose.service.nodeport.port: "30001"  # service portunu belirledim.
  tracing-server:
    image: openzipkin/zipkin
    ports:
     - 9411:9411
  admin-server:
    image: "{{ .Values.IMAGE_TAG_ADMIN_SERVER }}"
    ports:
     - 9090:9090
    labels:
      kompose.image-pull-secret: "regcred"
  hystrix-dashboard:
    image: "{{ .Values.IMAGE_TAG_HYSTRIX_DASHBOARD }}"
    ports:
     - 7979:7979
    labels:
      kompose.image-pull-secret: "regcred"
  grafana-server:
    image: "{{ .Values.IMAGE_TAG_GRAFANA_SERVICE }}"
    ports:
    - 3000:3000
    labels:
      kompose.image-pull-secret: "regcred"
  prometheus-server:
    image: "{{ .Values.IMAGE_TAG_PROMETHEUS_SERVICE }}"
    ports:
    - 9091:9090
    labels:
      kompose.image-pull-secret: "regcred"

  mysql-server:
    image: mysql:5.7.8
    environment: 
      MYSQL_ROOT_PASSWORD: petclinic
      MYSQL_DATABASE: petclinic
    ports:
    - 3306:3306
```

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de  ; farketmez)
# kompose kuralım
curl -L https://github.com/kubernetes/kompose/releases/download/v1.28.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
kompose version
```

* Install Helm [version 3+](https://github.com/helm/helm/releases) on Jenkins Server. [Introduction to Helm](https://helm.sh/docs/intro/). [Helm Installation](https://helm.sh/docs/intro/install/).

# Uygulamayı cluster'a `helm` kullanarak deploy edeceğiz, bu yüzden helm kuruyoruz

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de  ; farketmez)
# helm kuralım
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm version
```

* Create an helm chart named `petclinic_chart` under `k8s` folder.

```bash (pwd : home/ec2-user/petclinic-microservices-with-db )
cd k8s
helm create petclinic_chart 
# Bu komut ile chart oluşturacağız -->> komutu koştuğumuz k8s klasörü altında "petclinic_chart" klasörü oluştu

ls petclinic_chart
# Chart.yaml  charts  templates  values.yaml
# charts klasörü : bu chart'ın ihtiyaç duyduğu başka chart'lar veya dependency'ler olabilir, işte bunların içerisinde bulunacağı klasör
# templates klasörü   : kubernetes manifesto dosyalarının bulunduğu klasör
# Chart.yaml dosyası  : bu chart ile ilgili bilgiler bulunur (chart versiyonu, appVersion gibi)
# values.yaml dosyası : manifesto dosyalarında spesifik ifadeler yazıp her değişiklik gerektiğinde bütün yaml dosyalarında değişiklikleri yapmak yerine, manifesto dosyalarında değişken olarak kullandığın ifadeleri  values.yaml dosyasından al diyoruz. Değişiklik yapmak gerektiğinde sadece bu dosyada değişiklik yapmak yetiyor, mesela image ismi.

rm -r petclinic_chart/templates/*
# "petclinic_chart/templates" dizini içerisindeki dosyaları sil ve içerisine kubernetes object'lerini oluşturmamız için gerekli yaml dosyalarını koy.
# kubernetes object'lerini oluşturmamız için gerekli yaml dosyaları --NASIL OLUŞTU-->> kompose tool kullanarak docker-compose.yml ile
```

* Convert the `docker-compose.yml` into k8s/petclinic_chart/templates objects and save under `k8s/petclinic_chart` folder.

```bash
# yaml dosyasının ismi "docker-compose.yml" ise ve bu dosyayı gören dizinde komutu koşuyorsak  ==>> "-f docker-compose.yml" 'A GEREK YOK
# "-o petclinic_chart/templates"  ==>> oluşacak manifesto dosyalarının nerede oluşacağını söylüyorum, bir şey yazmazsan komutu koştuğun dizinde oluşur.
kompose convert -f docker-compose.yml -o petclinic_chart/templates

# tüm servisler için deployment ve service dosyaları oluştu
# ilave olarak api-gateway için `ingress` objesi de oluşturdu ==>> "labels" kısmında " kompose.service.expose: "{{ .Values.DNS_NAME }}" " yazdığım için
# ilave olarak k8s-default-policy.yaml network (kompose tool'un son versiyonu ile bu özellik geldi) oluştu : Bir cluster içinde başka proje/uygulama da çalışıyor olabilir, bu policy sayesinde gerekli parametreleri de girerek podlar arası erişim-iletişimi düzenleyebilirsiniz 
```

# service'lerin aynı anda ayağa kalkmaması gerekiyor, önce `config-server` sırasıyla `discovery-server` ve diğerlerinin ayağa kalkması gerekiyor. Bunun için docker-compose'da `dockerize` kullanmıştık. Kubernetes'da bunu `busybox` image kullanarak `initContainers` ile yapıyoruz

# Kompose ile hazırlanan yaml dosyalarda bu özellik oluşmadığından `DEPLOYMENT`  dosyalarına `spec` içerisine `containers` ile aynı hizada olacak şekilde ekleme yapacağız

  * https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

# command: ['sh', '-c', 'until nc -z discovery-server:8761; do echo waiting for discovery-server; sleep 2; done;'] AÇIKLAMASI

'sh'        : shell'de çalıştır
'-c'        : standart girdi yerine bu komut satırında geçen komutu çalıştır
nc (natcat) : TCP/UDP bağlantılarıyla bir kontrol yapacaksan
-z          : Parametre olarak "-z" seçeneği kullanıldığında, nc komutu sadece bağlantı kurma denemesi yapar ve herhangi bir veri alışverişi gerçekleştirmez.Bağlantı başarılı olursa, nc komutu çıktıda "succeeded" (başarılı) mesajını gösterir. Bağlantı başarısız olursa, hata mesajları veya hiçbir çıktı gösterilmez.
nc -z config-server:8888 : config-server olarak adlandırılan bir sunucunun 8888 numaralı bağlantı noktasının açık olup olmadığını kontrol etmek için kullanılır. Eğer çıktıda "succeeded" mesajı görünürse, bağlantı noktası açıktır; aksi takdirde, bağlantı noktası kapalı veya erişilemez olabilir. bir ağ taraması yapmak için kullanılan bir komuttur. Bu komut, "nc" (netcat) adlı bir aracı çalıştırır ve "config-server" olarak adlandırılan bir hedef sunucuya TCP üzerinden 8888 numaralı bir bağlantı kurmaya çalışır.

```yaml
# `DEPLOYMENT`  DOSYALARINDA YAPILACAK EKLEME (`spec` içerisine `containers` ile aynı hizada olacak)  :
# for discovery server
      initContainers:
        - name: init-config-server
          image: busybox
          command: ['sh', '-c', 'until nc -z config-server:8888; do echo waiting for config-server; sleep 2; done;']
# for admin-server, api-gateway, customers-service, hystrix-dashboard, vets-service and visits service
      initContainers:
        - name: init-discovery-server
          image: busybox
          command: ['sh', '-c', 'until nc -z discovery-server:8761; do echo waiting for discovery-server; sleep 2; done;']
```

# `Kompose` ile hazırlanan yaml dosyalarında bazı özellikler oluşmadığından veya hatalı oluştuğundan `api-gateway-ingress.yaml`  içerisinde `spec` değiştireceğiz

```yaml
spec:
  ingressClassName: nginx
  rules:
    - host: '{{ .Values.DNS_NAME }}'
      ...
```

* `k8s/petclinic_chart` dizininde `values-template.yaml` isminde dosya oluştur, içeriği aşağıda.
* son satırı değiştirmeyi unutma `domain name`. Diğer adımlarda `EKS` kullanırken `load balancer` için gerekli olacak

# Bu `values-template.yaml` dosyası içerisindeki değişkenleri dinamik olarak çekeceğim ---nerede--->>> Jenkinsfile içerisinde

# Bu `values-template.yaml` dosyasını ---- `envsubst`  komutunu kullanarak ---->>> `values.yaml` dosyasına dönüştüreceğiz

# Bu yeni oluşan `values.yaml` dosyasındaki image ismi ilgili deployment içerisine gidecek
  
```yaml
IMAGE_TAG_CONFIG_SERVER: "${IMAGE_TAG_CONFIG_SERVER}"
IMAGE_TAG_DISCOVERY_SERVER: "${IMAGE_TAG_DISCOVERY_SERVER}"
IMAGE_TAG_CUSTOMERS_SERVICE: "${IMAGE_TAG_CUSTOMERS_SERVICE}"
IMAGE_TAG_VISITS_SERVICE: "${IMAGE_TAG_VISITS_SERVICE}"
IMAGE_TAG_VETS_SERVICE: "${IMAGE_TAG_VETS_SERVICE}"
IMAGE_TAG_API_GATEWAY: "${IMAGE_TAG_API_GATEWAY}"
IMAGE_TAG_ADMIN_SERVER: "${IMAGE_TAG_ADMIN_SERVER}"
IMAGE_TAG_HYSTRIX_DASHBOARD: "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
IMAGE_TAG_GRAFANA_SERVICE: "${IMAGE_TAG_GRAFANA_SERVICE}"
IMAGE_TAG_PROMETHEUS_SERVICE: "${IMAGE_TAG_PROMETHEUS_SERVICE}"
DNS_NAME: "petclinic.devopsalparslanugurer.com"    # "DNS Name of your application"
```

# Helm repo için (Helm chart'ları için) s3 bucket oluşturalım (içinde ``stable/myapp`` dizini de oluşturacak)

```bash
# aws s3api create-bucket --bucket petclinic-helm-charts-<put-your-name> --region us-east-1
aws s3api create-bucket --bucket petclinic-helm-charts-arrow --region us-east-1
# aws s3api put-object --bucket petclinic-helm-charts-<put-your-name> --key stable/myapp/
aws s3api put-object --bucket petclinic-helm-charts-arrow --key stable/myapp/
```

# Helm kurunca direkt olarak s3 kullanamıyorum, bunun için plugin yüklemeliyim ``helm-s3 plugin``

  - https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/set-up-a-helm-v3-chart-repository-in-amazon-s3.html

  - https://helm.sh/    -->> Charts  : https://artifacthub.io/  -->> browse all packages : https://artifacthub.io/packages/search  --->> https://artifacthub.io/packages/helm-plugin/s3/s3

```bash
helm plugin install https://github.com/hypnoglow/helm-s3.git
```

# Bu plugin'i kurduk ama `ec2-user` görebiliyor, `jenkins user` görmüyor, jenkins user'a geçip orada da kuracağız

``` bash
# sudo su -s /bin/bash jenkins
sudo su - jenkins
export PATH=$PATH:/usr/local/bin  # helm'in çalıştığı path'i tanıtıyorum.
helm version  # helm kurulumunu görüyor.
helm plugin install https://github.com/hypnoglow/helm-s3.git
exit
```

# s3 bucket'ı Helm repo haline getirmek için `init` komutunu çalıştırmalısın. Repo halile gelmesi için `init` edilmeli ve init edilince `index.yaml` oluşmalı

```bash
AWS_REGION=us-east-1 helm s3 init s3://petclinic-helm-charts-arrow/stable/myapp # Bu komutu girince "index.yaml" oluştu
aws s3 ls s3://petclinic-helm-charts-arrow/stable/myapp/ # "index.yaml" gördüm
```

# S3 bucket içindeki `remote` Helm repoyu `local'e --yani--> jenkins-server makinaya` ekleyeceğim

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
helm repo list
helm repo ls  # lokalde repo henüz yok, error verir.
AWS_REGION=us-east-1 helm repo add stable-petclinicapp s3://petclinic-helm-charts-arrow/stable/myapp/
# lokaldeki ismi "stable-petclinicapp" olacak. EKLEME YAPARKEN LOKALDEKİ İSMİNİ BELİRTEBİLİRİM. 
### Lokaldeki "stable-petclinicapp"  isimli repom "s3://petclinic-helm-charts-arrow/stable/myapp/" URL'si ile bağlandı

helm repo ls
# stable-petclinicapp     s3://petclinic-helm-charts-arrow/stable/myapp         (REPOM  VE  BAĞLI OLDUĞU URL)
helm search repo stable-petclinicapp  # BURADA LİSTELENEN S3 BUCKET İÇİNDEKİ HELM REPO BİLGİLERİ OLUYOR.
```

* Update `version` and `appVersion` field of `k8s/petclinic_chart/Chart.yaml` file as below for testing.

```yaml
version: 0.0.1
appVersion: 0.1.0
```

# Local Helm Chart'ı repoyu  ``Package`` paketleyeceğiz.  `templates` klasörü içeriğini zipliyor. Sonrasında istersek `install` komutu ile  `release` oluşturabilirim , yani kubernetes object'ler oluşur ve uygulamayı ayağa kaldırmış olurum

```bash
cd k8s
helm package petclinic_chart/   # petclinic_chart-0.0.1.tgz OLUŞTU , Chart.yaml dosyasında version ne yazıyorsa (version: 0.0.1) o versiyon numarasıyla paket oluştu.
```

# Store the local package in the Amazon S3 Helm repository. Oluşan `artifact : petclinic_chart-0.0.1.tgz` s3'e gönder

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de/k8s)
HELM_S3_MODE=3 AWS_REGION=us-east-1 helm s3 push ./petclinic_chart-0.0.1.tgz stable-petclinicapp
# "stable-petclinicapp"  ZATEN "s3://petclinic-helm-charts-arrow/stable/myapp"E BAĞLI, YANİ PUSH EDİNCE S3'E GİTTİ.
```

# Search for the Helm chart. AŞAĞIDAKİ 2 KOMUT İLE DE GÖREBİLİRİM

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de/k8s)
helm search repo stable-petclinicapp  # BURADA LİSTELENEN S3 BUCKET İÇİNDEKİ HELM REPO BİLGİLERİ OLUYOR.
aws s3 ls s3://petclinic-helm-charts-arrow/stable/myapp/
```

* ``Chart.yaml``içerisinde `version` ifadesini `0.0.2` olarak değiştirin, yeniden chart'ı paketle, yeni versiyonu s3 helm repoya gönder,

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de/k8s)
helm package petclinic_chart/

HELM_S3_MODE=3 AWS_REGION=us-east-1 helm s3 push ./petclinic_chart-0.0.2.tgz stable-petclinicapp

helm repo update  # Bu komut ile bilgileri güncellemek gerekiyor, s3 yerine github kullanınca bu komutu girmeden aşağıdaki komutu girersen hata verebiliyor.
helm search repo stable-petclinicapp

      NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                
      stable-petclinicapp/petclinic_chart     0.0.2           0.1.0           A Helm chart for Kubernetes
```

* Tüm versiyonlarını görmek için aşağıdaki komut çalıştır.

```bash
helm search repo stable-petclinicapp --versions
# helm search repo stable-petclinicapp -l   (bu komut da olur)
    NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                
    stable-petclinicapp/petclinic_chart     0.0.2           0.1.0           A Helm chart for Kubernetes
    stable-petclinicapp/petclinic_chart     0.0.1           0.1.0           A Helm chart for Kubernetes
```

# ``Chart.yaml`` dosyasında `version` değerini `HELM_VERSION` olarak değiştir. `Jenkins Pipeline` ile dinamik olarak bu ifade çekilecek, her job build ettiğimde değişecek

``` bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added Configuration YAML Files for Kubernetes Deployment'
git push --set-upstream origin feature/msp-17
git checkout dev
git merge feature/msp-17
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 18 - Prepare a QA Automation Pipeline for Nightly Builds***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Developer uygulamada güncelleme yaptı github'a gönderdi
* Jenkins pipeline için scm (source code management) github, Jenkinsfile ile mvn clean package çalıştırıp github'dan çektiğim dosya ve klasörler ile jar dosyası üretiyorum,
* Jar dosyasını dockerfile'da kullanıyorum, image oluşturuyorum,
* image'ları ECR'a gönderiyorum,
* Deployment yaml dosyalarına image'ların son halini çekiyorum,
* helm package ile son halini paketliyorum ve s3'e gönderiyorum,
* helm install ile uygulamanın son hali ayağa kalkacak.

```bash
# Create `feature/msp-18` branch from `dev`.
git checkout dev
git branch feature/msp-18
git checkout feature/msp-18
```

- Maven ile oluşturacağım jar paketleri için `jenkins` klasörü içinde `package-with-maven-container.sh` isminde bir script oluşturalım

```bash (package-with-maven-container.sh)
docker run --rm -v $HOME/.m2:/root/.m2 -v $WORKSPACE:/app -w /app maven:3.6-openjdk-11 mvn clean package
```

- ECR repoya göndereceğim image'ları ``ECR tags`` tag'lemek için `jenkins` klasörü içinde `prepare-tags-ecr-for-dev-docker-images.sh` isminde bir script oluşturalım

### AŞAĞIDAKİ 2 KOMUT HER BİR SERVER IMAGE ETİKETLENMESİ İÇİN YAPILACAK, AÇIKLANMASI

    MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
    export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"

  # MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)

  ". test.txt = source test.txt = . ./test.txt = source ./test.txt = chmod +x test.txt +++ ./test.txt " BU 5'i DE AYNI ŞEY , echo $version komutu ile versiyonu görürsün;
  ./  -->> KOMUTU KOŞTUĞUN DİZİNDE ÇALIŞTIR DEMEK, ama "." ve "source" ile birlikte kullanmana gerek yok ;
  KOMUTU KOŞTUĞUM YERDEN BAŞKA BİR DİZİNDEKİ DOSYA İÇİN PATH BELİRTEBİLİRİM  ./alp/arslan/test.txt GİBİ   ;
  /alp  olursa  `root` dizinden itibaren absolute path gösteriyorsun (nokta kullanmadım en başta)

  pipeline içerisinde `package-with-maven-container.sh` çalıştığında, bu script içerisindeki `mvn clean package` komutu girilmiş olacak ve bu paketleme işi volume olarak bağladığım `workspace/pipeline-ismi` klasörü altında olacak, her bir service için `target/maven-archiver/` dizinlerinde `pom.properties` oluşacak, bu dosya içerisinde versiyon bilgisi bulunur  --->>> `workspace/pipeline-ismi/spring-petclinic-admin-server/target/maven-archiver/pom.properties`
  $WORKSPACE : jenkins'te tanımlı değişken : workspace/pipeline-ismi
                                                          --->>> `${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties`

  Yani bu komut sonucunda `2.1.2` gibi versiyon bilgisi gelecek
  
# export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
  
  admin-server-v${MVN_VERSION}-b${BUILD_NUMBER} : bu kısım image'ın tag kısmı, latest olarak değil de versiyon ve build numaraları içeren bir tag

  ${MVN_VERSION} : 2.1.2    (Yukarıdaki açıklamadan)

  BUILD_NUMBER : Jenkins'teki tanımlı değişkenlerden  --->> b${BUILD_NUMBER}  --- 2. build olsun --->> b2

  admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}  --->>> admin-server-v2.1.2-b2

  Bu adım sonundaki `Jenkinsfile` içerisinde :
                                APP_NAME="petclinic"
                                APP_REPO_NAME="clarusway-repo/${APP_NAME}-app-dev" --->>>
                                APP_REPO_NAME="clarusway-repo/petclinic-app-dev"

                                AWS_REGION="us-east-1"
                                AWS_ACCOUNT_ID=sh(script:'aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
                                  - Bu komut sonucu = benim AWS Account ID : 377346128947
                                ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                                ECR_REGISTRY="377346128947.dkr.ecr.us-east-1.amazonaws.com"

                                {ECR_REGISTRY}/${APP_REPO_NAME} --->>> 377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-dev

EN SON ALACAĞI İFADE :
          export IMAGE_TAG_ADMIN_SERVER="377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-dev:admin-server-v2.1.2-b2"

```bash (prepare-tags-ecr-for-dev-docker-images.sh)
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

# `IMAGE_TAG_ADMIN_SERVER` yukarıda dinamik olarak çekilen tag'li bu image isimleri  --->>> `values-template.yaml`  içerisine dinamik olarak gidecek oradan da `envsubst` komutu ile --->>> `values.yaml` içerisine --- oradan da --->>> deployment yaml dosyaları içerisine gidecek

# Prepare a script to build the dev docker images tagged for ECR registry and save it as `build-dev-docker-images-for-ecr.sh` and save it under `jenkins` folder

```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```

# Prepare a script to push the dev docker images to the ECR repo and save it as `push-dev-docker-images-to-ecr.sh` and save it under `jenkins` folder

```bash
# Provide credentials for Docker to login the AWS ECR and push the images
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY} 
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```

```bash
- Commit the change, then push the scripts to the remote repo.
# 
git add .
git commit -m 'added scripts for qa automation environment'
git push --set-upstream origin feature/msp-18
```

* Yukarıda oluşturduğumuz script'leri deneyeceğimiz bir job

```yml
- job name: test-msp-18-scripts
- job type: Freestyle project
- GitHub project: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */feature/msp-18
- Build:
      Add build step: Execute Shell
      Command:
```

```bash
# Bu komutlarda daha aynı isimde repo var mı diye kontrol kısmı yok, bu yüzden bu job çalışmadan önce ECR'daki aynı isimli repoyu console üzerinden silmelisin.
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="clarusway-repo/petclinic-app-dev" # Write your own repo name
AWS_REGION="us-east-1" #Update this line if you work on another region
ECR_REGISTRY="377346128947.dkr.ecr.us-east-1.amazonaws.com" # Replace this line with your ECR name , sadece account ID değiştirden de olur.
aws ecr create-repository \
    --repository-name ${APP_REPO_NAME} \
    --image-scanning-configuration scanOnPush=false \
    --image-tag-mutability MUTABLE \
    --region ${AWS_REGION}
. ./jenkins/package-with-maven-container.sh
. ./jenkins/prepare-tags-ecr-for-dev-docker-images.sh
. ./jenkins/build-dev-docker-images-for-ecr.sh
. ./jenkins/push-dev-docker-images-to-ecr.sh
```

Click `Save`--->>> `Build Now`

* `ansible/playbooks` klasörü altında  `dev-petclinic-deploy-template` isminde playbook için template dosyası oluştur.

# daha sonra `envsubst` komutu ile `ansible/playbooks/dev-petclinic-deploy.yaml` haline gelecek. Neden template : bazı değişken değerlerini sonradan alacak/çekecek -->> `$JENKINS_HOME` ve `${BUILD_NUMBER}`(bu değişkenler jenkins'de default tanımlı değişkenlerden),  ve `$AWS_REGION` .

# config.json dosyasının açıklaması:
  - `.docker`   isminde dosya sistemi/dizini var ne zaman oluştu bu ==>> `push-dev-docker-images-to-ecr.sh`  isimli script içerisinde `docker login` komutu var, biz job çalıştırınca bu script çalıştı ve "docker login" komutu çalıştı --> bu komut bir `credential` oluşturuyor ve komutun çalıştığı makinanın `home` dizininde `.docker` oluşuyor ve bu credential'ı `home` dizinde oluşan .docker altına koyuyor. ``ECR repodan/repoya image pull/push`` edebilmesi için repoya girme yetkisi veren credentials.
  - Yani bu job'ı jenkins user çalıştırdığına göre, jenkins user'ın home dizini `/var/lib/jenkins içerisinde` `.docker` oluşuyor --->>> `/var/lib/jenkins/.docker` altında da `config.json` oluşuyor ve credential bunun içerisinde.

```bash
sudo su - jenkins
pwd   # /var/lib/jenkins
ls -a   
cd .docker
ls
cat config.json
```

# name: deploy petclinic application :
  - https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets
  - Master ve Worker node'ların ECR repodan image çekmesi için repoya girme yetkisi olan credentials'a ihtiyacı var ===>>> jenkins içindeki `config.json` dosyasını -----copy modülü ile gönder----->>> master node içine `config.json` geldi -----shell modülü ile master node içerisindeki `config.json` dosyasından `regcred` isminde `secret` oluştur----->>> Bu regcred` ismindeki secret `deployment.yaml` dosyalarında tanımlanmıştı ===>>> Yani node'lar artık recgred'e dolayısıyla credentials'a sahipler ve böylece ECR repodan image çekebilecekler

# --version ${BUILD_NUMBER}  açıklaması :
- `Chart.yaml` içerisinde `version: HELM_VERSION` yazmıştık, sonrasında jenkinsfile içerisinde `sed` komutu ile değiştirip bunun yerine `Build` numarasını yaz/değiştir diyeceğiz, yani bizim bu işlemleri yapmadan önce son göndereceğimiz paketlenmiş zipli dosyaların adı `petclinic_chart-0.0.1.tgz` YERİNE `petclinic_chart-1.tgz` olacak. İşte `--version ${BUILD_NUMBER}`'deki numara da oradan gelecek

```bash
helm repo ls    
      NAME                          URL
      stable-petclinicapp     s3://petclinic-helm-charts-arrow/stable/myapp

helm search repo stable-petclnicapp
    NAME                                    CHART VERSION       APP VERSION      DESCRIPTION
    stable-petclinicapp/petclinic_chart     0.0.2               0.1.0            A Helm Chart of Kubernetes
  
helm search repo stable-petclnicapp -l
    NAME                                    CHART VERSION       APP VERSION      DESCRIPTION
    stable-petclinicapp/petclinic_chart     0.0.2               0.1.0            A Helm Chart of Kubernetes
    stable-petclinicapp/petclinic_chart     0.0.1               0.1.0            A Helm Chart of Kubernetes
```


```yaml (dev-petclinic-deploy-template)
- hosts: role_master  # sadece master makinada
  tasks:

  - name: Create .docker folder
    file:   # dosya veya dosya sistemi oluşturma veya da dizin oluşturma
      path: /home/ubuntu/.docker  # home dizininde .docker isminde dizin oluşturacak
      state: directory
      mode: '0755'    # rwx rx rx

  - name: copy the docker config file   # içerisinde credential bulunan config.json isimli dosya kopyalanacak
    become: yes
    copy: 
      src: $JENKINS_HOME/.docker/config.json  # $JENKINS_HOME : var/lib/jenkins , job'un çalıştığı jenkis-server ec2-instance home
      dest: /home/ubuntu/.docker/config.json  # credential  master makinanın home dizinine gönderiyorum

  - name: deploy petclinic application  # Bu kısmın açıklaması yukarıda
  # helm plugin install https://github.com/hypnoglow/helm-s3.git   komutu ile `master'a` `s3 plugin` kuruyoruz, `ansible/playbooks` içerisindeki `k8s_setup.yaml` içerisinde helm kurmuştuk
    shell: |
      helm plugin install https://github.com/hypnoglow/helm-s3.git  
      kubectl create ns petclinic-dev
      kubectl delete secret regcred -n petclinic-dev || true
      kubectl create secret generic regcred -n petclinic-dev \
        --from-file=.dockerconfigjson=/home/ubuntu/.docker/config.json \
        --type=kubernetes.io/dockerconfigjson
      AWS_REGION=$AWS_REGION helm repo add stable-petclinic s3://petclinic-helm-charts-arrow/stable/myapp/  
    # bucket ismi değiştir
      AWS_REGION=$AWS_REGION helm repo update
      AWS_REGION=$AWS_REGION helm upgrade --install \ 
        petclinic-app-release stable-petclinic/petclinic_chart --version ${BUILD_NUMBER} \
        --namespace petclinic-dev
# helm upgrade --install : böyle bir release varsa upgrade yap yoksa install et demek
```

* Deneme amaçlı dummy Selenium testi yapacağız ve bunun için `selenium-jobs` klasörü altında `dummy_selenium_test_headless.py` isimli test dosyamızı oluşturuyoruz.

```python
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("headless")
chrome_options.add_argument("no-sandbox")
chrome_options.add_argument("disable-dev-shm-usage")
driver = webdriver.Chrome(options=chrome_options)

base_url = "https://www.google.com/"
driver.get(base_url)
source = driver.page_source

if "I'm Feeling Lucky" in source:
  print("Test passed")
else:
  print("Test failed")
driver.close()
```

* Deneme amaçlı yapacağımız dummy Selenium testi için `ansible/playbooks` klasörü altında `pb_run_dummy_selenium_job.yaml` isminde Ansible playbook yazıyoruz.

# ASLINDA DENEDİĞİMİZ ŞEY PLAYBOOK VE TEST; SONRASINDA `jenkinsfile-petclinic-nightly` İSİMLİ Jenkinsfile İÇİNDEKİ `stage('Run QA Automation Tests')` KISMINDA `pb_run_selenium_jobs.yaml` İSİMLİ PLAYBOOK ÇALIŞTIRILACAK VE BÖYLECE TÜM TEST DOSYALARI KULLANILARAK TESTLER `Jenkins server` İÇERİSİNDE YANİ `127.0.0.1`'DE  YAPILMIŞ OLACAK

# `{{ workspace }}` ansible içerisinde bir değişken ve ben bu değişkeni henüz tanımlamadım, ansible ile değişken tanımlama yöntemlerinden bir tanesi de `ansible-playbook` komutu girerken tanımlamaktır. Ben bu komutu `jenkinsfile` içerisinde girince --->>> hem ansible için değişken tanımlaması yapmış olacağım hem de aynı komutla jenkins içindeki `WORKSPACE` değişkenine atayarak jenkins'in de bu değişkeni tanımasını sağlamış olacağım. Yani jenkins içerisinde workspace dizini altında oluşan job klasörü

```yaml
- hosts: all
  tasks:
  - name: run dummy selenium job
    shell: "docker run --rm -v {{ workspace }}:{{ workspace }} -w {{ workspace }} clarusway/selenium-py-chrome:latest python {{ item }}"
    with_fileglob: "{{ workspace }}/selenium-jobs/dummy*.py"  # https://docs.ansible.com/ansible/2.8/plugins/lookup/fileglob.html : BİRDEN FAZLA DOSYA KOPYALAMAK GEREKTİĞİNDE HER DEFASINDA  src VE dest YAZMAK YERİNE `with_fileglob` İLE DOSYALARIN TAMAMINI LİSTELEMİŞ OLUYORUZ, YUKARIDAKİ `shell` KOMUTU İÇİNDEKİ {{ item }} BU LİSTEDEKİ HER BİR DOSYA OLUR.  
    register: output  # test sonucunu yazılı görmek için yukarıdaki komutaların çıktısını (register) output isminde değişkene atıyorum, aşağıda da debug modülü ile bu çıktıları okuyabilirim, yani test sonuçlarını görebilirim.
  
  - name: show results
    debug: msg="{{ item.stdout }}"
    with_items: "{{ output.results }}"
```

* Bu playbook (Jenkins Server'daki dummy selenium job çalışması için) çalışması için `ansible/scripts` klasörü altında `run_dummy_selenium_job.sh` isminde  bir script oluşturalım. 

```bash
# --connection=local --inventory 127.0.0.1  YAZMAMIZIN SEBEBİ PLAYBOOK İÇERİSİNDE hosts: all  YERİNE jenkins İÇİNDE ÇALIŞMASI İÇİN
PATH="$PATH:/usr/local/bin"
ansible-playbook --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE}" ./ansible/playbooks/pb_run_dummy_selenium_job.yaml
```

* Deneme amaçlı yazdığımız `dummy_selenium_test_headless.py` isimli selenium test dosyasını `petclinic-microservices-with-db` içerisinde komut girerek çalıştıralım: Terminalde komutu gireceğiz.

```bash
# --connection=local --inventory 127.0.0.1: Bu makina 
# "workspace=$(pwd)"                      : Komutu bu makinada girdiğim için komutu koştuğum dizin olacak  home/ec2-user/petclinic-microservices-with-db
cd petclinic-microservices-with-db/
ansible-playbook --connection=local --inventory 127.0.0.1, --extra-vars "workspace=$(pwd)" ./ansible/playbooks/pb_run_dummy_selenium_job.yaml
```

```bash
# Commit the change, then push the scripts for dummy selenium job to the remote repo.
git add .
git commit -m 'added scripts for running dummy selenium job'
git push --set-upstream origin feature/msp-18
```

* Deneme amaçlı yazdığımız `dummy_selenium_test_headless.py` isimli selenium test dosyasını `test-running-dummy-selenium-job` isimli jenkins job ile çalıştıralım

```yml
- job name: test-running-dummy-selenium-job
- job type: Freestyle project
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */feature/msp-18
- Build:
      Add build step: Execute Shell
      Command:
```

```bash
ansible-playbook --connection=local --inventory 127.0.0.1, --extra-vars "workspace=$(pwd)" ./ansible/playbooks/pb_run_dummy_selenium_job.yaml
```

# Şimdi `selenium-jobs` klasörü atındaki tüm selenium testleri yapacak `asıl` dosyamızı `ansible/playbooks` klasörü altında `pb_run_selenium_jobs.yaml` isimli Ansible playbook olarak hazırlayalım

  *** BU TESTLERİ YAPMAK İÇİN PLAYBOOK HAZIRLAMAMIN SEBEBİ: EĞER BEN Playbook İÇİNDEKİ docker run KOMUTUNU Jenkinsfile İÇERİSİNDE Playbook KULLANMADAN DİREKT YAZSAYDIM --->>> `TESTLERDEN BİRİSİ HATA VERSE Jenkinsfile DURURDU`. FAKAT Playbook İLE ÇALIŞTIRIRSAM TESTLER HATA VERSE BİLE SONUCU YAZAR VE Jenkinsfile Job SONUNA KADAR DEVAM EDER.***

```yaml
# --rm : işi bitince container siler
# Jenkinsfile içerisinde Playbook'lar çalışmadan önce " stage('Test the Application Deployment') " KISMINDA master makina IP : MASTER_PUBLIC_IP  alacağız --->>> Bu bilgi test içerisinde kullanılacak
# clarusway/selenium-py-chrome:latest : İçerisinde Selenium yüklü test yapmak amaçlı yapılmış bir image
# --env MASTER_PUBLIC_IP={{ master_public_ip }}  -->> jenkinsfile içerisinde bu playbook'lar çalışmadan önce bu değişken tanımlanmış ve bilgi çekilmiş olacak, bana master makinanın Public IP'si lazım çünkü uygulamaya api-gateway 30001 portundan bağlanıp test edeceğim
- hosts: all
  tasks:
  - name: run all selenium jobs
    shell: "docker run --rm --env MASTER_PUBLIC_IP={{ master_public_ip }} -v {{ workspace }}:{{ workspace }} -w {{ workspace }} clarusway/selenium-py-chrome:latest python {{ item }}"
    register: output
    with_fileglob: "{{ workspace }}/selenium-jobs/test*.py"
  
  - name: show results
    debug: msg="{{ item.stdout }}"
    with_items: "{{ output.results }}"
```

# Test dosyalarında `test_owners_all_headless.py (18. ve 19. satır), test_owners_register_headless.py (16. ve 17. satır) ve test_veterinarians_headless.py (18. ve 19. satır)`  aşağıdaki değişiklikleri yapalım. Zaten nodeport service ve api-gateway service 30001 portlarını açmıştım, uygulama buradan selenium teste tabi tutulacak

```py
APP_IP = os.environ['MASTER_PUBLIC_IP']
url = "http://"+APP_IP.strip()+":30001/"
```

* Jenkins Server (localhost) içinde çalıştırarak `Master Node`a tüm selenium testleri yapmak maksadıyla playbook'u çalıştırmak için `ansible/scripts` klasörü altında `run_selenium_jobs.sh` isimli bir script oluşturalım.

# --connection=local --inventory 127.0.0.1 : lokal yani Jenkins server'da testi çalıştıracak ===>>> TESTİ KİME YAPACAK `MASTER NODE`A YAPACAK

```bash
# -vvv  : ilave bilgi getir
# --connection=local --inventory 127.0.0.1 : lokal yani Jenkins server'da testi çalıştıracak ===>>> TESTİ KİME YAPACAK "MASTER NODE"A YAPACAK
PATH="$PATH:/usr/local/bin"
ansible-playbook -vvv --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE} master_public_ip=${MASTER_PUBLIC_IP}" ./ansible/playbooks/pb_run_selenium_jobs.yaml
```

* `jenkins` klasörü altında `jenkinsfile-petclinic-nightly` isimli bir Jenkinsfile (`petclinic-nightly`) oluşturalım.

## jenkinsfile-petclinic-nightly

  - jenkinsfile 109. satır repo ismini bucket ismini değiştirmeyi unutma !!!

```groovy
pipeline {
    agent any
    environment { // Stage'lerde kullanacağım environment variable tanımlamaları
        APP_NAME="petclinic"
        APP_REPO_NAME="clarusway-repo/${APP_NAME}-app-dev"
        AWS_ACCOUNT_ID=sh(script:'aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
// https://www.jenkins.io/doc/pipeline/steps/workflow-durable-task-step/ , https://www.baeldung.com/ops/jenkins-pipeline-output-shell-command-variable
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ANS_KEYPAIR="${APP_NAME}-dev-${BUILD_NUMBER}.key"
        ANSIBLE_PRIVATE_KEY_FILE="${WORKSPACE}/${ANS_KEYPAIR}"
        ANSIBLE_HOST_KEY_CHECKING="False"
    }
    stages {
        stage('Create ECR Repo') {  // Böyle bir repo varsa tanımla repo yoksa create et.
            steps {
                echo "Creating ECR Repo for ${APP_NAME} app"
                sh '''
                aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                         aws ecr create-repository \
                         --repository-name ${APP_REPO_NAME} \
                         --image-scanning-configuration scanOnPush=true \
                         --image-tag-mutability MUTABLE \
                         --region ${AWS_REGION}
                '''
            }
        }
        stage('Package Application') {  // jar dosyalarını oluşturan sonunda mvn clean package komutu olan docker run komutunu içeren script
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Docker Images') { // env.IMAGE_TAG_ADMIN_SERVER ile yani env. ile başlayan değişken tanımlamasıyla değişken tüm stage'lerde geçerli oluyor.
            steps {
                echo 'Preparing Tags for Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Docker Images') { 
            steps {
                echo 'Building App Dev Images'
                sh ". ./jenkins/build-dev-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-dev-docker-images-to-ecr.sh"
            }
        }
        stage('Create Key Pair for Ansible') {  // Terraform ve Ansible'da kullanacağım key-pair oluşturuyorum.
            steps {
                echo "Creating Key Pair for ${APP_NAME} App"
                sh "aws ec2 create-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR} --query KeyMaterial --output text > ${ANS_KEYPAIR}"
                sh "chmod 400 ${ANS_KEYPAIR}"
            }
        }

        stage('Create QA Automation Infrastructure') {  // Terraform dosyası içinde key-pair: clarus yazıyor, sed komutu ile değiştiriyorum, yukarıdaki stage ile oluşan ${ANS_KEYPAIR}'i üzerine yazıyorum.
            steps {
                echo 'Creating QA Automation Infrastructure for Dev Environment'
                sh """
                    cd infrastructure/dev-k8s-terraform
                    sed -i "s/clarus/$ANS_KEYPAIR/g" main.tf 
                    terraform init
                    terraform apply -auto-approve -no-color
                """
                script {  // makinaların "Status check : 2/2 checks passed" olmasını bekletiyor. https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/wait/index.html    ;  https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/wait/instance-status-ok.html
                    echo "Kubernetes Master is not UP and running yet."
                    env.id = sh(script: 'aws ec2 describe-instances --filters Name=tag-value,Values=master Name=tag-value,Values=tera-kube-ans Name=instance-state-name,Values=running --query Reservations[*].Instances[*].[InstanceId] --output text',  returnStdout:true).trim()
                    sh 'aws ec2 wait instance-status-ok --instance-ids $id'  // master node'un instance id'sini çekip burada kullanıyorum
                }
            }
        }


        stage('Create Kubernetes Cluster for QA Automation Build') {
            steps {
                echo "Setup Kubernetes cluster for ${APP_NAME} App"
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml ./ansible/playbooks/k8s_setup.yaml"
            }
        }

        stage('Deploy App on Kubernetes cluster'){   
            steps { // Jenkins artık oluşturulan image tag'lerini tanıdığı için values-template.yaml dosyası içindeki değişkenler tanımlanmış oluyor, envsubst komutu ile values.yaml'a dönüştürüyorum. Deployment içerisindeki - image:'{{ .Values.IMAGE_TAG_ADMIN_SERVER }}' kısmınada image bilgileri gelmiş oluyor.
                    // Yaml dosyalarını chart haline getireceğim helm package komutu ile, ama bunu yapmadan önce chart'ın versiyon bilgisinin geldiği Chart.yaml dosyası içerisindeki version:HELM_VERSION bilgisini BUILD_NUMBER bilgisi ile değiştir diyorum sed komutu kullanarak, yani paket ismi "petclinic_chart-0.0.2.tgz" yerine "petclinic_chart-2.tgz" diye gelecek.
                    // helm repo add  komutu ile Jenkins server içine repomu ekliyorum.
                    // helm package ile paketliyorum ve artık chart hazır
                    // --force : aynı isimde s3'te paket varsa normalde hata verir ama bu şekilde overwrite yapar.
                    // dev-petclinic-deploy-template dosyası içerisindeki değişkenler artık tanımlanmış oluyor, envsubst komutu ile bunu dev-petclinic-deploy.yaml dosyasına dönüştürüyorum
                echo 'Deploying App on Kubernetes' 
                sh "envsubst < k8s/petclinic_chart/values-template.yaml > k8s/petclinic_chart/values.yaml"
                sh "sed -i s/HELM_VERSION/${BUILD_NUMBER}/ k8s/petclinic_chart/Chart.yaml"
                sh "helm repo add stable-petclinic s3://petclinic-helm-charts-arrow/stable/myapp/" // s3'teki repo ismini değiştirmeyi unutma
                sh "helm package k8s/petclinic_chart"
                sh "helm s3 push --force petclinic_chart-${BUILD_NUMBER}.tgz stable-petclinic"
                sh "envsubst < ansible/playbooks/dev-petclinic-deploy-template > ansible/playbooks/dev-petclinic-deploy.yaml"
                sh "sleep 60"  // daha önce denendi belli bir süre beklemek gerekiyor.
                sh "ansible-playbook -i ./ansible/inventory/dev_stack_dynamic_inventory_aws_ec2.yaml ./ansible/playbooks/dev-petclinic-deploy.yaml"
            }
        }     // Uygulamanın ayağa kalkması zaman alacak, master node'a 30001 portundan yapacağım selenium testler öncesi uygulamanın ayağa kalktığını ve master node'un 30001 portundan yayın yaptığını kontrol ediyorum. Bu kontrol işlemi için groovy dilinde while döngüsü yazıyorum. Uygulamanın hazır olup olmadığını sürekli olarak kontrol etmek istiyoruz. Döngü içinde, curl komutu kullanılarak belirtilen IP adresi ve port üzerinde uygulamaya bir istek gönderilir (curl -s ${MASTER_PUBLIC_IP}:30001). Eğer istek başarılı bir şekilde tamamlanırsa, yani uygulama yanıt veriyorsa, ${APP_NAME} app is successfully deployed. mesajı yazdırılır ve döngüden çıkılır (break). Eğer istek başarısız olursa, yani uygulama yanıt vermiyorsa, "Could not connect to ${APP_NAME} app" mesajı yazdırılır ve 5 saniye boyunca bekleme yapılır (sleep(5)). Döngü, uygulama yanıt verene kadar veya başka bir hata oluşana kadar devam eder.

        stage('Test the Application Deployment'){
            steps {
                echo "Check if the ${APP_NAME} app is ready or not"
                script {
                    env.MASTER_PUBLIC_IP = sh(script:"aws ec2 describe-instances --region ${AWS_REGION} --filters Name=tag-value,Values=master Name=tag-value,Values=tera-kube-ans Name=instance-state-name,Values=running --query Reservations[*].Instances[*].[PublicIpAddress] --output text", returnStdout:true).trim()
                    while(true) {
                        try{
                            sh "curl -s ${MASTER_PUBLIC_IP}:30001"
                            echo "${APP_NAME} app is successfully deployed."
                            break
                        }
                        catch(Exception){
                            echo "Could not connect to ${APP_NAME} app"
                            sleep(5)
                        }
                    }
                }
            }
        }

        stage('Run QA Automation Tests'){ 
            steps {
                echo "Run the Selenium Functional Test on QA Environment"
                sh 'ansible-playbook -vvv --connection=local --inventory 127.0.0.1, --extra-vars "workspace=${WORKSPACE} master_public_ip=${MASTER_PUBLIC_IP}" ./ansible/playbooks/pb_run_selenium_jobs.yaml'
            }
        }
 }

    post {  // Testleri yaptığıma ve raporu aldığıma göre cluster'ın ayakta durmasına gerek yok
        always {  // Hata da olsa tüm işlemler başarılı da olsa aşağıdaki işlemleri yap
            echo 'Deleting all local images'
            sh 'docker image prune -af'
            echo 'Delete the Image Repository on ECR'
            sh """
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                """
            echo 'Tear down the Kubernetes Cluster'
            sh """
            cd infrastructure/dev-k8s-terraform
            terraform destroy -auto-approve -no-color
            """
            echo "Delete existing key pair using AWS CLI"
            sh "aws ec2 delete-key-pair --region ${AWS_REGION} --key-name ${ANS_KEYPAIR}"
            sh "rm -rf ${ANS_KEYPAIR}"
        }
    }
}
```

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added qa automation pipeline for dev'
git push
git checkout dev
git merge feature/msp-18
git push origin dev
```

* Create a Jenkins pipeline with the name of `petclinic-nightly` with the following script to run QA automation tests and configure a `cron job` to trigger the pipeline every night at midnight (`0 0 * * *`) on the `dev` branch. Input `jenkins/jenkinsfile-petclinic-nightly` to the `Script Path` field. Petclinic nightly build pipeline should be built on a temporary QA automation environment.

```yml
- job name: petclinic-nightly
- job type: Pipeline
- Discard old builds: 3  ,   3
- Build Triggers :
  - Build periodically:
    - Schedule: 0 0 * * *
- Pipeline:
  - Definition: Pipeline scriptp from SCM
    - SCM: Git
      - Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
      - Branches to build:
      Branch Specifier (blank for 'any'): */dev
    - Script Path: jenkins/jenkinsfile-petclinic-nightly
```

Click `Save`--->>> `Build Now`

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 19 - Create a QA Environment on EKS Cluster***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Create `feature/msp-19` branch from `dev`.

```bash
git checkout dev
git branch feature/msp-19
git checkout feature/msp-19
```

* Tester'lar manuel test yapacaklar: uygulama sayfasına girip uygulamanın tüm fonksiyonlarını çalışıyormu deneyecekler.
* Cluster kurarken `EKS` kullanacağız.
* Biz istersek eksctl kullanıp komutlarla cluster kurabiliriz (Kubernetes 8. Hands-on), ama böylesi uzun komut girmek pek uygun değildir. Böyle uzun komut girmemiz gerekiyorsa mutlaka `ClusterConfig` yani Konfigürasyon dosyası vardır `https://eksctl.io/usage/creating-and-managing-clusters/` , bu `ClusterConfig` dosyasına neleri ekleyebilirim dersen `https://eksctl.io/usage/schema/` . 

* `infrastructure` klasörü altında EKS cluster setup için `qa-eks-cluster` isminde klasör oluştur. `infrastructure/qa-eks-cluster` klasörü altında `cluster.yaml` isimli dosya oluştur. Sadece kayıt/arşiv amaçlı bu dosyayı kullanmayacağız.

```yaml (cluster.yaml)
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: petclinic-cluster
  region: us-east-1
availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]
managedNodeGroups:
  - name: ng-1
    instanceType: t3a.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 3
    volumeSize: 8
```

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added cluster.yaml file'
git push --set-upstream origin feature/msp-19
git checkout dev
git merge feature/msp-19
git push origin dev
```

* ### Install eksctl (aws eks'deki cluster'ları yönetmek için kullandığımız CLI)

```bash (pwd : home/ec2-user)
# Download and extract the latest release of eksctl with the following command.
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

# - Move the extracted binary to /usr/local/bin.
sudo mv /tmp/eksctl /usr/local/bin

# Test that your installation was successful with the following command.
eksctl version
```

### Install kubectl (cluster'ı yönetmek için CLI)

```bash (pwd : home/ec2-user)
# Download the Amazon EKS vended kubectl binary.
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl

# Apply execute permissions to the binary.
chmod +x ./kubectl

# Move the kubectl binary to /usr/local/bin. Her kullanıcı ulaşabilsin
sudo mv kubectl /usr/local/bin

# After you install kubectl , you can verify its version with the following command
kubectl version --short --client
```

*** `eksctl create cluster` ile başlayan komutla cluster oluşturduğunda hangi kullanıcıyla/user'dayken cluster oluşturduysam doğrudan o kullanıcının/user'ın cluster'a erişme yetkisi oluyor, ilave konfigürasyon ayarları ile uğraşmak zorunda kalmıyorum. Burada jenkins arayüzü ile cluster'a ulaşmak istiyorum, bu yüzden cluster oluşturma işlemini `jenkins user` altında yapacağız.**
  - `eksctl create cluster -f cluster.yaml` komutu ile birlikte ``/var/lib/jenkins`` altında `.kube` klasörü ve bunun altında da ``cache ve config`` dosyaları oluşur.
*** `kubectl` komutu nereye bakar `.kube` klasörünün içindeki `config` yani konfigürasyon dosyasına bakar. Cluster ayağa kalktıktan sonra  `/var/lib/jenkins` altında `ls -a` yaptığında `.kube` klasörünü , `.kube` klasörü içinde `ls` komutu ile `cache ve config` dosyalarını görürsün.**

* `/var/lib/jenkins` klasörü altında `cluster.yaml` isminde dosya oluştur.

```bash (pwd : home/ec2-user)
# jenkins kullanıcısına geç   # pwd : /var/lib/jenkins
sudo su - jenkins
vi cluster.yaml # aşağıdakini içerik olarak kopyala-yapıştır.
```

```yaml (cluster.yaml)
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: petclinic-cluster
  region: us-east-1
availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"] # AZ yazmazsan kendisi ayrı bir VPC create eder ve içindeki AZ'leri kullanır 
managedNodeGroups:
  - name: ng-1
    instanceType: t3a.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 3
    volumeSize: 8
```

* Create an EKS cluster via `eksctl`. It will take a while.

```bash (pwd : /var/lib/jenkins)
ls -a
# .kube klasörü yok , aşağıdaki komut ile .kube dizini oluşacak, oluşan bu .kube içerisindeki "config" üzerinden ben cluster'a bağlanabileceğim
eksctl create cluster -f cluster.yaml
ls -a
# .kube  klasörü geldi
```
# Cluster oluşması 15-20 dakika sürüyor. BU İŞLEM SÜRERKEN YENİ BİR TERMİNAL AÇ VE ec2-user'a geç, pwd : home/ec2-user OLACAK, AŞAĞIDAKİ KOMUTU CLUSTER AYAĞA KALKTIKTAN SONRA GİRECEKSİN. BURADA MSP 20 ADIMINA GEÇELİM ve diğer adımlara devam edelim.
* Cluster `ayağa kalkınca` default olarak gelmeyen `ingress controller`u manuel kurmak için aşağıdaki komutu çalıştır.

```bash (pwd : /var/lib/jenkins)
export PATH=$PATH:$HOME/bin # ESKİDEN JENKİNS USER EKSCTL KOMUTUNU GÖRMÜYORDU, ARTIK GÖRÜYOR, BU SATIRA GEREK YOK.
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 20 - Prepare Build Scripts for QA Environment***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash
# Create `feature/msp-20` branch from `dev`.
git checkout dev
git branch feature/msp-20
git checkout feature/msp-20
```

* AWS ECR repo oluşturmak için ``Jenkins Job`` oluşturalım.

```yml
- job name: create-ecr-docker-registry-for-petclinic-qa
- job type: Freestyle project
- Build:
      Add build step: Execute Shell
      Command:
```

```bash
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="clarusway-repo/petclinic-app-qa"
AWS_REGION="us-east-1"

aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
aws ecr create-repository \
 --repository-name ${APP_REPO_NAME} \
 --image-scanning-configuration scanOnPush=false \
 --image-tag-mutability MUTABLE \
 --region ${AWS_REGION}
```

Click `Save`--->>> `Build Now`

-`jenkins` klasörü altında docker image'lerine dinamik olarak ECR tag'leri atamak için `prepare-tags-ecr-for-qa-docker-images.sh` isimli bir script oluşturalım.

```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

-`jenkins` klasörü altında dinamik olarak ECR tag'leri atanmış docker image'lerini build etmek için  `build-qa-docker-images-for-ecr.sh` isimli bir script oluşturalım.

```bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```

-`jenkins` klasörü altında dinamik olarak ECR tag'leri atanmış ve build edilmiş olan docker image'lerini ECR Repoya push etmek için  `push-qa-docker-images-to-ecr.sh` isimli bir script oluşturalım.

```bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```

-`jenkins` klasörü altında uygulamayı deploy etmek için  `deploy_app_on_qa_environment.sh` isimli bir script oluşturalım.

```bash
# 4. satırda s3 helm chart repo ismini değiştirmeyi unutma
echo 'Deploying App on Kubernetes'
envsubst < k8s/petclinic_chart/values-template.yaml > k8s/petclinic_chart/values.yaml
sed -i s/HELM_VERSION/${BUILD_NUMBER}/ k8s/petclinic_chart/Chart.yaml
AWS_REGION=$AWS_REGION helm repo add stable-petclinic s3://petclinic-helm-charts-arrow/stable/myapp/ || echo "repository name already exists"
AWS_REGION=$AWS_REGION helm repo update
helm package k8s/petclinic_chart
AWS_REGION=$AWS_REGION helm s3 push --force petclinic_chart-${BUILD_NUMBER}.tgz stable-petclinic
kubectl create ns petclinic-qa || echo "namespace petclinic-qa already exists"
kubectl delete secret regcred -n petclinic-qa || echo "there is no regcred secret in petclinic-qa namespace"
kubectl create secret generic regcred -n petclinic-qa \
    --from-file=.dockerconfigjson=/var/lib/jenkins/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
AWS_REGION=$AWS_REGION helm repo update
AWS_REGION=$AWS_REGION helm upgrade --install \
    petclinic-app-release stable-petclinic/petclinic_chart --version ${BUILD_NUMBER} \
    --namespace petclinic-qa
```

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added build scripts for QA Environment'
git push --set-upstream origin feature/msp-20
git checkout dev
git merge feature/msp-20
git push origin dev
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 21 - Build and Deploy App on QA Environment Manually***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash
# Create `feature/msp-21` branch from `dev`.
git checkout dev
git branch feature/msp-21
git checkout feature/msp-21
```

# Cluster kurmuştuk şimdi manuel test yapacak tester'lara ortam hazırlıyorum, uygulamayı yükleyeceğim.

- `jenkins` klasörü altında uygulamayı deploy etmek için  `build-and-deploy-petclinic-on-qa-env-manually.sh` isimli bir script oluşturalım.
- Aşağıdaki script içerisinde adı geçen .sh dosyaları yukarıda oluşturduğumuz scriptlerdir.
* Create a ``Jenkins Job`` with name of `build-and-deploy-petclinic-on-qa-env` to build and deploy the app on `QA environment` manually on `release` branch using following script

# Build işlemini aşağıdaki push ve merge komutlarundan sonra yap !!!!!!!!!!

```yml
- job name: build-and-deploy-petclinic-on-qa-env  
- job type: Freestyle project
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */release
- Build:
      Add build step: Execute Shell
      Command:
```

```bash (build-and-deploy-petclinic-on-qa-env-manually.sh)
PATH="$PATH:/usr/local/bin:$HOME/bin"
APP_NAME="petclinic"
APP_REPO_NAME="clarusway-repo/petclinic-app-qa"
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION="us-east-1"
ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
echo 'Packaging the App into Jars with Maven'
. ./jenkins/package-with-maven-container.sh
echo 'Preparing QA Tags for Docker Images'
. ./jenkins/prepare-tags-ecr-for-qa-docker-images.sh
echo 'Building App QA Images'
. ./jenkins/build-qa-docker-images-for-ecr.sh
echo "Pushing App QA Images to ECR Repo"
. ./jenkins/push-qa-docker-images-to-ecr.sh
echo 'Deploying App on Kubernetes Cluster'
. ./jenkins/deploy_app_on_qa_environment.sh
echo 'Deleting all local images'
docker image prune -af
```

# Build işlemini aşağıdaki push ve merge komutlarundan sonra yap

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added script for jenkins job to build and deploy app on QA environment'
git push --set-upstream origin feature/msp-21
git checkout dev
git merge feature/msp-21
git push origin dev
```

```bash
# Merge `dev` into `release` branch, then run `build-and-deploy-petclinic-on-qa-env` job to build and deploy the app on `QA environment` manually.
git checkout release
git merge dev
git push origin release
```

# Click `Save`--->>> `Build Now`


# Build işleminden sonra çalışan job klasörüne git `var/lib/jenkind/workspace/build-and-deploy-petclinic-on-qa-env` -->> içindeki `k8s/petclinic_chart` klasörüne git `ls` ile listele : `Chart.yaml   templates   values-template.yaml   values.yaml`  , `values.yaml` dosyasının içine bak-incele.
  - `values.yaml` :
      IMAGE_TAG_CONFIG_SERVER: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:config-server-qa-v2.1.2-b1"
      IMAGE_TAG_DISCOVERY_SERVER: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:discovery-server-qa-v2.1.2-b1"
      IMAGE_TAG_CUSTOMERS_SERVICE: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:customers-service-qa-v2.1.2-b1"
      IMAGE_TAG_VISITS_SERVICE: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:visits-service-qa-v2.1.2-b1"
      IMAGE_TAG_VETS_SERVICE: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:vets-service-qa-v2.1.2-b1"
      IMAGE_TAG_API_GATEWAY: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:api-gateway-qa-v2.1.2-b1"
      IMAGE_TAG_ADMIN_SERVER: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:admin-server-qa-v2.1.2-b1"
      IMAGE_TAG_HYSTRIX_DASHBOARD: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:hystrix-dashboard-qa-v2.1.2-b1"
      IMAGE_TAG_GRAFANA_SERVICE: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:grafana-service"
      IMAGE_TAG_PROMETHEUS_SERVICE: "377346128947.dkr.ecr.us-east-1.amazonaws.com/clarusway-repo/petclinic-app-qa:prometheus-service


# MSP-19 sonunda `ingress controller` kurmuştuk, `EKS` cluster ayağa kaldırıp uygulamayı deploy edince `cloud` desteği aldığımızdan `load balancer` oluşacak ve dış dünyadan uygulamaya erişim bu load balancer üzerinden olacak.  AWS Console `Load Balancers` aç ve classic load balancer olduğunu gör.

```bash (pwd : /var/lib/jenkins , jenkins-user)
kubectl get ns
kubectl get po -n petclinic-qa
kubectl get ingress -n petclinic-qa # api-gateway-ingress.yaml  -->> values-template.yaml
#    NAME          CLASS   HOSTS                                 ADDRESS                                                                  PORTS   AGE
#    api-gateway   nginx   petclinic.devopsalparslanugurer.com   a711dfa0ce80748b18cd088cd61d7f83-781346454.us-east-1.elb.amazonaws.com   80      8m9s

kubectl edit ingress -n petclinic-qa
```


# `api-gateway-ingress.yaml` içindeki host name'i `Route53` servisine bağla : ingress'i route53'e bağlıyoruz

* `Route53` servisine git -->> Hosted zones -->> devopsalparslanugurer.com  -->> Create record -->>
                                Record name: petclinic
                                Record type: A
                                Value/Route traffic to -->> Alias to Application and Classic Load Balancer
                                                            US East(N. Virginia) [us-east-1]
                                                            Load Balancer seç
                                Create record

* Sayfayı görmek için browser'a `petclinic.devopsalparslanugurer.com` yaz ve uygulamayı gör

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 22 - Prepare a QA Pipeline***
# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash
# Create `feature/msp-22` branch from `dev`.
git checkout dev
git branch feature/msp-22
git checkout feature/msp-22
```

-`jenkins` klasörü altında `petclinic-weekly-qa` build'leri için  `jenkinsfile-petclinic-weekly-qa` isimli bir Jenkinsfile oluşturalım.

# Pipeline içerisinde 1 stage hariç scriptleri çalıştırdık, sebebi o stage içerisinde `script` ve `env.` ile değişkenleri tanımlayınca tüm stage'lerde geçerli olacağından diğer stagelerde o değişkenleri artık kullanabiliriz

```groovy
pipeline {
    agent any
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin:$HOME/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="clarusway-repo/petclinic-app-qa"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Docker Images') {
            steps {
                echo 'Preparing Tags for Docker Images'
                script {  // env. kullanmamızın amacı buradaki değişkenler tüm stage'lerde tanımlı hale gelmiş olacak
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-qa-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Docker Images') {
            steps {
                echo 'Building App Dev Images'
                sh ". ./jenkins/build-qa-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-qa-docker-images-to-ecr.sh"
            }
        }
        stage('Deploy App on Kubernetes Cluster'){
            steps {
                echo 'Deploying App on Kubernetes Cluster'
                sh '. ./jenkins/deploy_app_on_qa_environment.sh'
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

```bash
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added jenkinsfile petclinic-weekly-qa for release branch'
git push --set-upstream origin feature/msp-22
git checkout dev
git merge feature/msp-22
git push origin dev
```

```bash
# Merge `dev` into `release` branch to build and deploy the app on `QA environment` with pipeline.
git checkout release
git merge dev
git push origin release
```

* Jenkins server'da `petclinic-weekly-qa` isminde ``Pipeline``   hazırlayalım ve `release` branşında her pazar gece yarısı (`59 23 * * 0`) çalışacak şekilde bir `cron job` trigger (Build Triggers -->> Build periodically -->> Schedule : 59 23 ** 0) edelim. Petclinic weekly build pipeline should be built on permanent QA environment.

```yml
- job name: petclinic-weekly-qa
- job type: pipeline
- Pipeline:
  - Definition: Pipeline script from SCM
    - SCM: Git
        Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */release
- Pipeline:
      Script Path: jenkins/jenkinsfile-petclinic-weekly-qa
```

- Click `Save`--->>> `Build Now`

## ingress'e neden `host` ekliyoruz : `k8s/petclinic_chart/templates/api-gateway-ingress.yaml` içerisinde `host: host name` kısmına host name yazıyoruz -->> böylelikle `hostname (petclinic.devopsalparslanugurer.com)` isminde ingress object oluşuyor. ingress controller hostname'i loadbalancer'a  yönlendiriyor. Güvenlik sağlanmış oluyor sadece hostname üzerinden uygulamaya ulaşılabiliyor

## Zaten route53 ile bağladım, `api-gateway-ingress.yaml` içerisindeki `- host: ''{{ .Values.DNS_NAME }}` kısmına host yazmayım desem, ben Loadbalancer isminden DE uygulamaya ulaşabilirdim. Fakat güvensiz olur

```bash
sudo su - jenkins
kubectl get ingress -n petclinic-qa
  NAME           CLASS   HOSTS                                  ADDRESS           PORTS   AGE
  api-gateway    nginx   petclinic.devopsalparslanugurer.com    LoadBalancerİSMİ  80      44m

curl petclinic.devopsalparslanugurer.com # loadbalancer'ı Route53'e yönlendiriyor
curl loadbalancerismi # curl yapmıyor , izin vermiyor. NEDEN host:petclinic.devopsalparslanugurer.com girdiğim için.

# Eğer `k8s/petclinic_chart/templates/api-gateway-ingress.yaml` içinde host kısmı olmasaydı aşağıdaki komut çalışırdı.
kubectl edit ingress -n petclinic-qa  # ingress içerisindeki host kısmını sil ve http ile başlayan satır önüne - koy ve save et çık.
curl loadbalancerismi # şimdi geliyor
```

* EKS cluster'ı `eksctl` ile silmek için aşağıdaki komutu koş, terminal akmaya devam edecek ve zaman alacaktır, AWS Console üzerinde silindiğini gördükten sonra ec2'ları stop et.

```bash (jenkins user olarak /var/lib/jenkins içerisinde bu komutu koşmalısın)
sudo su - jenkins
eksctl delete cluster -f cluster.yaml
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

*** MSP 23 - Prepare High-availability RKE Kubernetes Cluster on AWS EC2***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

``` bash
# Create `feature/msp-23` branch from `release`.
git checkout release
git branch feature/msp-23
git checkout feature/msp-23
```

# `Rancher server` isminde bir makina (Ubuntu 20.04) ayağa kaldıracağız. Rancher server cluster'ı yöneteceğine göre IAM Policy'lere ihtiyaç duyacak, AWS'de service'lere erişim yetkisi gerekecek, bunun için 2 tane policy oluşturacağız. Rancher'ın içinde bir uygulama çalışıyor ve kendisi de bir kubernetes cluster'ında çalışıyor. Normalde de en az 3 makina ayağa kaldırıp cluster oluşturup rancher'ı o cluster'ın üzerine deploy ediyoruz, ama biz maliyetli olmasın diye bu eğitimde 1 makina kullanacağız.
# Rancker için cluster kurma işini `RKE` (Rancher Kubernetes Engine) isminde tool ile yapacağız ama isterseniz EKS ile (eksctl create cluster) de kurulum yapabilirsiniz ve bu bilgiye dokümantasyondan erişebilirsiniz.
# Bugün 1 adet `Ubuntu 20.04` ec2 ayağa kaldıracağız, AWS service'lerine ulaşabilmesi için `role (petclinic-rke-role)` atayacağız, bu role için de `controlplane (control-node)` ve `etcd - worker` görevlerini yapabilmesi için 2 `policy` atayacağız. Tek makina kullanacağımız için tüm policy'leri tek bir role atayıp makinaya role'ü bağlayacağız. Normally in the market en az 3 makina var, 1'ine `controlplane` policy'li role'ü atarsın diğer 2'sine `etcd - worker` policy'li role'ü atarsın.
# `Load balancer` oluşturacağız ki kullanıcı önce loadbalancer'a oradanda `dns name` üzerinden rancher'a ulaşsın, bunun için `dns name` ve `certificate` ayarlayacağız. `certificate` olmadan cluster'ı yönetmemize izin vermiyor, kullanıcının `secure -->> https` ile ulaşması gerekiyor.
# Rancher ve loadbalancer için toplam 2 adet `security group` oluşturacağız. `rancher-cluster-sec-gr` `rancher-alb-sec-gr`
# Rancker için cluster kurma işini `RKE` isminde tool ile yapacağız demiştik, bu tool'u `jenkins-server` içine install edeceğia çünkü rancher cluster kurulumunu ve yönetim işini buradan yapacağız. Jenkins-server içinde RKE komutu ile bir yaml dosyası kullanarak Rancher için cluster kuracağız, yani rancher uygulamasını çalıştıracak olan cluster hazırlandı. Bu cluster üzerine rancher uygulamasını deploy edeceğim, cluster üzerine uygulamayı nereden yükleyeceğiz `HELM`'de bulunan artifact repodan.

# Rancher : <https://rancher.com/docs/rancher/v2.x/en/overview/architecture/>

- <https://ranchermanager.docs.rancher.com/pages-for-subheaders/quick-start-guides>
* <https://ranchermanager.docs.rancher.com/pages-for-subheaders/installation-and-upgrade>
* <https://ranchermanager.docs.rancher.com/pages-for-subheaders/deploy-rancher-manager>

# Policy : <https://rke.docs.rancher.com/config-options/cloud-providers>   --->>> AWS   


* Rancher'ın EC2 resource'larına erişebilmesini sağlamak maksadıyla `Control Plane` görevi için `infrastructure` klasörü altında `petclinic-rke-controlplane-policy.json` isimli IAM Policy dosyası oluşturalım.
* Aynı zamanda AWS Console IAM Service'e git ve policy oluştur.

``` json
{
"Version": "2012-10-17",
"Statement": [
  {
    "Effect": "Allow",
    "Action": [
      "autoscaling:DescribeAutoScalingGroups",
      "autoscaling:DescribeLaunchConfigurations",
      "autoscaling:DescribeTags",
      "ec2:DescribeInstances",
      "ec2:DescribeRegions",
      "ec2:DescribeRouteTables",
      "ec2:DescribeSecurityGroups",
      "ec2:DescribeSubnets",
      "ec2:DescribeVolumes",
      "ec2:CreateSecurityGroup",
      "ec2:CreateTags",
      "ec2:CreateVolume",
      "ec2:ModifyInstanceAttribute",
      "ec2:ModifyVolume",
      "ec2:AttachVolume",
      "ec2:AuthorizeSecurityGroupIngress",
      "ec2:CreateRoute",
      "ec2:DeleteRoute",
      "ec2:DeleteSecurityGroup",
      "ec2:DeleteVolume",
      "ec2:DetachVolume",
      "ec2:RevokeSecurityGroupIngress",
      "ec2:DescribeVpcs",
      "elasticloadbalancing:AddTags",
      "elasticloadbalancing:AttachLoadBalancerToSubnets",
      "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
      "elasticloadbalancing:CreateLoadBalancer",
      "elasticloadbalancing:CreateLoadBalancerPolicy",
      "elasticloadbalancing:CreateLoadBalancerListeners",
      "elasticloadbalancing:ConfigureHealthCheck",
      "elasticloadbalancing:DeleteLoadBalancer",
      "elasticloadbalancing:DeleteLoadBalancerListeners",
      "elasticloadbalancing:DescribeLoadBalancers",
      "elasticloadbalancing:DescribeLoadBalancerAttributes",
      "elasticloadbalancing:DetachLoadBalancerFromSubnets",
      "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
      "elasticloadbalancing:ModifyLoadBalancerAttributes",
      "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
      "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
      "elasticloadbalancing:AddTags",
      "elasticloadbalancing:CreateListener",
      "elasticloadbalancing:CreateTargetGroup",
      "elasticloadbalancing:DeleteListener",
      "elasticloadbalancing:DeleteTargetGroup",
      "elasticloadbalancing:DescribeListeners",
      "elasticloadbalancing:DescribeLoadBalancerPolicies",
      "elasticloadbalancing:DescribeTargetGroups",
      "elasticloadbalancing:DescribeTargetHealth",
      "elasticloadbalancing:ModifyListener",
      "elasticloadbalancing:ModifyTargetGroup",
      "elasticloadbalancing:RegisterTargets",
      "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
      "iam:CreateServiceLinkedRole",
      "kms:DescribeKey"
    ],
    "Resource": [
      "*"
    ]
  }
]
}
```

* Rancher'ın EC2 resource'larınıa erişebilmesini sağlamak maksadıyla `etcd` veya `worker` görevleri için `infrastructure` klasörü altında `petclinic-rke-etcd-worker-policy.json` isimli IAM Policy dosyası oluşturalım.
* Aynı zamanda AWS Console IAM Service'e git ve policy oluştur.

```json
{
"Version": "2012-10-17",
"Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "ec2:DescribeInstances",
            "ec2:DescribeRegions",
            "ecr:GetAuthorizationToken",
            "ecr:BatchCheckLayerAvailability",
            "ecr:GetDownloadUrlForLayer",
            "ecr:GetRepositoryPolicy",
            "ecr:DescribeRepositories",
            "ecr:ListImages",
            "ecr:BatchGetImage"
        ],
        "Resource": "*"
    }
]
}
```

* yukarıdaki policy'lerden `petclinic-rke-role` isminde IAM Role oluştur.

# security groups : <https://ranchermanager.docs.rancher.com/v2.5/getting-started/installation-and-upgrade/installation-requirements/port-requirements> - https://ranchermanager.docs.rancher.com/v2.5/getting-started/installation-and-upgrade/installation-requirements/port-requirements#ports-for-rancher-server-nodes-on-rke

* AWS Console -- Security Groups -->> ``Application Load Balancer`` için  ``petclinic-rke-alb-sg`` isminde security group oluştur:
  * Inbound : `HTTP (Port 80) and HTTPS (Port 443) anywhere`
  * Outbound : Aynı kalsın
# create security group

* AWS Console --> Security Groups -->> ``RKE Kubernetes Cluster`` için  ``petclinic-rke-cluster-sg`` isminde security group oluştur:
  
  * Inbound :   - `HTTP (Port 80)`,          `Source: petclinic-rke-alb-sg` 80 portundan kimse uygulamaya erişemesin sadece HTTPS(443)'den ulaşsın diye, yani  SADECE Application Load Balancer security group'dan geçen HTTP(80)'e izin ver
                - `HTTPS (Port 443)`,        `Source: Anywhere` Rancher UI ve API için kurulumda ihtiyaç duyulan paketlerin  bazıları https sayfalardan/linklerden inecek
                - `Custom TCP (Port 6443)`,  `Source: Anywhere` Jenkins-server'dan kubectl ile Rancher-server ile etkileşime geçip işlemler yaptırabilsin, yani cluster'ı yönetebilsin.
                - `SSH (Port 22)` ,          `Source: Anywhere` Jenkins-server'a Docker kurayım ki sonrasında RKE tool'u ile kubernetes cluster kurabileyim

  * Outbound :  - `SSH (Port 22)` ,           `Source: Anywhere` to any node IP from a node created using Node Driver.
                - `HTTP (Port 80)` ,          `Source: Anywhere` to all IP for getting updates.
                - `HTTPS (Port 443)`,         `Source: 35.160.43.145/32` for catalogs of `git.rancher.io`.
                - `HTTPS (Port 443)`,         `Source: 35.167.242.46/32` for catalogs of `git.rancher.io`.
                - `HTTPS (Port 443)`,         `Source: 52.33.59.17/32` for catalogs of `git.rancher.io`.
                - `Custom TCP (Port 2376)`,   `Source: Anywhere` Rancher'ın docker engine ile iletişime geçmesi için gerekli
                - `All traffic`,              `Source: Anywhere` Normalde dökümanda böyle bir port açılması yok, ama docker kurulumu için gerekli belge/paketlerin curl komutu ile indirilmesinde oluşan sorunu aşmak için ekledik.
# create security group

# Oluştuktan sonra `Edit inbound rules` : `All TCP`, `Source: petclinic-rke-cluster-sg` (KENDİ ID'sini GİRİYORUM). Cluster içerisindeki node'lar birbiri ile görüşsün diye yaptım, dökümanda yazılı portları `Rules for traffic between Rancher nodes` teker teker yazmak yerine

* RKE tool , Rancher'a kubernetes cluster kurma işlemini ssh bağlantı üzerinden key.pem kullanarak yapıyor.
* Jenkins Server'da Rancher Server için AWS CLI kullanarak  `petclinic-rancher.pem` isminde bir key-pair oluştur.

```bash (pwd :  farketmez)
aws ec2 create-key-pair --region us-east-1 --key-name petclinic-rancher --query KeyMaterial --output text > ~/.ssh/petclinic-rancher.pem
chmod 400 ~/.ssh/petclinic-rancher.pem
```

* 1 adet EC2 ayağa kaldır : 
                            - Tag               : `Name:Petclinic-Rancher-Cluster-Instance`
                            - Image             : `Ubuntu Server 20.04 LTS (HVM) (64-bit x86)`
                            - Instance type     : `t3a.medium`
                            - key-pair          : `petclinic-rancher.pem`
                            - security group    : `petclinic-rke-cluster-sg`
                            - Configure storage : 16 GB gp2
                            - Advanced Details  :
                                                  - IAM Role : `petclinic-rke-role`
  - Launch instance.

# Tags : <https://ranchermanager.docs.rancher.com/pages-for-subheaders/use-existing-nodes#3-amazon-only-tag-resources>

- Key=kubernetes.io/cluster/<CLUSTERID>, Value=owned
* EC2'nun ayağa kalktığı `subnets`'e, `nodes (intances)`'a ve `security group`'a aşağıdaki tag'i ata:
                            `Key = kubernetes.io/cluster/Petclinic-Rancher`
                            `Value = owned`
  
* Jenkins-server içinden ssh ile `Petclinic-Rancher-Cluster-Instance`'a bağlan (Log into `Petclinic-Rancher-Cluster-Instance` from Jenkins Server (Bastion host) and install Docker using the following script.) :
  * .ssh klasörü üzerinde sağa tıkla `Open in Integrated Terminal` ve ayrı bir terminal ile aç, AWS Console üzerinden `Petclinic-Rancher-Cluster-Instance`'e connect , ssh ile bağlanma komutunu kopyala-yapıştır.

```bash (ec2-user@jenkins-server# .ssh:$)
ssh -i "petclinic-rancher.pem" ubuntu@ec2-54-89-114-22.compute-1.amazonaws.com
# Set hostname of instance
sudo hostnamectl set-hostname rancher-instance-1
# Update OS 
sudo apt-get update -y
sudo apt-get upgrade -y
# Update the apt package index and install packages to allow apt to use a repository over HTTPS
sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
sudo mkdir -p /etc/apt/keyrings
# security group'ta ALL TCP yapmamızın nedeni bu komut :
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg  
# Use the following command to set up the stable repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash (ec2-user@jenkins-server# .ssh:$)
# Update packages
sudo apt-get update

# Install and start Docker  (SON VERSİYONU KURMUYORUM ÇÜNKÜ UYUMSUZLUK VAR)
# RKE is not compatible with the current Docker version (v23 hence we need to install an earlier version of Docker
sudo apt-get install docker-ce=5:20.10.23~3-0~ubuntu-focal docker-ce-cli=5:20.10.23~3-0~ubuntu-focal containerd.io docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker

# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu
newgrp docker

docker --version    # Docker version 20.10.23, build 7155243
docker ps
```

* AWS Console üzerinden `Target group` oluşturma:

```yml
Create target group: Enter

Target group name   : petclinic-rancher-http-80-tg
Target type         : instance
Protocol            : HTTP
Port                : 80
VPC                 : default-vpc
Protocol version    : HTTP1

<!-- Health Checks Settings -->
Protocol            : HTTP
Health check path   : /healthz
<!-- Advanced Health Checks Settings -->
Port                : Traffic port
Healthy threshold   : 3
Unhealthy threshold : 3
Timeout             : 5 seconds
Interval            : 10 seoconds
Success             : 200

Next : Enter

<!-- Register targets -->
Petclinic-Rancher-Cluster-Instance : SEÇ
Include as pending below : ENTER

Create target group : ENTER
```

* AWS Console üzerinden `Application Load Balancer` oluşturma:  `petclinic-rke-alb-sg` security group ve `petclinic-rancher-http-80-tg` target group kullanarak.

```yml
Application Load Balancer  : SEÇ
Create : ENTER
Load balancer name : petclinic-rancher-alb  

Scheme              : internet-facing
IP address type     : ipv4

<!-- Network mapping-->
VPC : default-vpc
Mappings : Rancher'ın olduğu subnet ve ilave 2 subnet daha SEÇ

<!-- Security groups-->
Security groups : petclinic-rke-alb-sg (EKRANA GELEN DEFAULT security group SİL)

<!-- Listeners and routing-->
Protocol            : HTTP
Port                : 80
Default action      : `petclinic-rancher-http-80-tg`


<!-- Add Listener-->
Protocol            : HTTPS   # Bunu seçince altta policy ve certificate kısmı çıktı
Port                : 443
Default action      : `petclinic-rancher-http-80-tg`


<!-- Secure listener settings -->
<!-- Default SSL/TLS certificate -->
From ACM            : *.devopsalparslanugurer.com   # change with your dns name

Create load balance : ENTER
```

* AWS Console'da oluşturduğumuz Load balancer'a git
          - Listener -->> HTTP 80 -->> Actions -->> Edit Listener
          - Önce Remove -->> Default actions -->> Redirect -->> Protocol : HTTPS   Port : 443   ---->>>> Save changes
          *** BURASI DEĞİŞMİŞ    <Redirect to URL>  <HTTP 443>

* AWS Console Route53 :  Hosted zones -->> devopsalparslanugurer.com -->> Create record -->>  Record name : rancher
                                                                                              Record type : A
                                                                                              Alias : AÇ
                                                                                              Route traffic to: Alias to Application and Classic Load Balancer
                                                                                              Choose Region : US East(N. Virginia) [us-east-1]
                                                                                              Choose load balancer : petclinic-rancher-alb
                                                                              Create records : ENTER

* Install RKE, the Rancher Kubernetes Engine, [Kubernetes distribution and command-line tool] on Jenkins Server.
# jenkins-server içerisinde rke tool'u kuracağız

# <https://rancher.com/docs/rke/latest/en/installation/>

```bash (jenkins-server'da   home/ec2-user)
curl -SsL "https://github.com/rancher/rke/releases/download/v1.4.5/rke_linux-amd64" -o "rke_linux-amd64"
sudo mv rke_linux-amd64 /usr/local/bin/rke
chmod +x /usr/local/bin/rke
rke --version   # rke version v1.4.5
```

* `infrastructure` klasörü altında RKE Kubernetes Cluster'ını konfigüre etmek/kurmak için `rancher-cluster.yml` isminde bir dosya oluşturalım.

# <https://rke.docs.rancher.com/config-options>

# <https://rke.docs.rancher.com/example-yamls>

# <https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-references/helm-chart-options#configuring-ingress-for-external-tls-when-using-nginx-v025>

```yml
# Biz tek makina kullandık, ama normalde en az 3 tane kullanılır.
nodes:
  - address: 172.31.40.33            # Change with the Private Ip of rancher server ; Normalde Public IP girilir. Çünkü bu makinalar hiç stop edilmez.
    internal_address: 172.31.40.33   # Change with the Private Ip of rancher server ; Normalde Private IP girilir.
    user: ubuntu
    role:
      - controlplane
      - etcd
      - worker

# ignore_docker_version: true   KULLANDIĞIMIZ DOCKER VERSİYONU GÖZARDI ET DİYEBİLİRİZ, AMA BİZ ZATEN SON VERSİYON YERİNE UYUMLU OLAN VERSİYONU KULLANDIK VE İHTİYACIMIZ KALMADIĞINDAN BU SATIRI YORUMA ALDI,

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

ssh_key_path: ~/.ssh/petclinic-rancher.pem  # Rancher server'a ssh ile ulaşıyor

# Required for external TLS termination with
# ingress-nginx v0.22+
# https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-references/helm-chart-options#configuring-ingress-for-external-tls-when-using-nginx-v025
ingress:                # Normalde Rancher'ın kendi kurulumunda "Letsencrypt"'den sertifika aldırabiliyor, dahili sertifika kullanma durumu var. Biz burada AWS'nin kendi sertifikasını yani "external" bir sertifika kullanıyoruz.
  provider: nginx
  options:
    use-forwarded-headers: "true"
```

* Run `rke` command to setup RKE Kubernetes cluster on EC2 Rancher instance *`Warning:` You should add rule to cluster sec group for Jenkins Server using its `IP/32` from SSH (22) and TCP(6443) before running `rke` command, because it is giving connection error.*

```bash (pwd : petclinic-microservices-with-db)
cd infrastructure
rke up --config ./rancher-cluster.yml
```

# Yukarıdaki komut ile terminal 2-3 dakika akacak. Bu komuttan sonra infrastructure klasörü altında `rancher-cluster.rkestate` isminde bir dosya oluştu. Terraform.state dosyası gibi. Ayrıca `kube_config_rancher-cluster.yml` isminde bir dosya daha oluşuyor cluster'ın `config` dosyasıdır bu.

# Yukarıda oluştu dediğim `rancher-cluster.rkestate` ve `kube_config_rancher-cluster.yml` isimli dosyaları `.kube` klasörü altına göndereceğim. Böylelikle kubectl ile podlarımı görmeye başlayacağım.

```bash (pwd : infrastructure  klasörü altında )
mkdir -p ~/.kube
mv ./kube_config_rancher-cluster.yml $HOME/.kube/config
mv ./rancher-cluster.rkestate $HOME/.kube/rancher-cluster.rkestate
chmod 400 ~/.kube/config
kubectl get nodes
kubectl get pods --all-namespaces
```

```bash (pwd : petclinic-microservices-with-db)
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added rancher setup files'
git push --set-upstream origin feature/msp-23
git checkout release
git merge feature/msp-23
git push origin release
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 24 - Install Rancher App on RKE Kubernetes Cluster ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

# Şimdiye kadar sadece cluster kurduk, daha rancher yüklemedik. Bu kısımda rancher yükleyeceğiz.

# https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster#1-add-the-helm-chart-repository
# https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-references/helm-chart-options#health-checks

```bash (pwd : petclinic-microservices-with-db)
# Add ``helm chart repositories`` of Rancher.
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo list
```

* Create a ``namespace`` for Rancher.

```bash (pwd : petclinic-microservices-with-db)
kubectl create namespace cattle-system
```

* Install Rancher on RKE Kubernetes Cluster using Helm.

```bash (pwd : petclinic-microservices-with-db)
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.devopsalparslanugurer.com \
  --set tls=external \
  --set replicas=1 \
  --set global.cattle.psp.enabled=false
  
# Change DNS name (hostname)
# namespace cattle-system  : buradaki namespace ismi dökümandan (artifact.hub) geliyor
# set tls=external :  harici olarak AWS'den aldığım sertifikamı kullanıyorum
# set replicas=1   : default 3 , rancher normalde 3 makinada çalışıyor, ama biz 1 tane kullanacağız
# set global.cattle.psp.enabled=false  : port security policy (psp) isminde bir component rancher'ın port güvenlik politikasını devre dışı bırakıyor. Podlara ulaşmayı engellemeyi bırakıyor.
```

* Check if the Rancher Server is deployed successfully.
  
```bash
kubectl -n cattle-system get deploy rancher
kubectl -n cattle-system get pods
```

* Browser'a : <https://rancher.devopsalparslanugurer.com>
  * Ekranda çıkan komutu kopyala-yapıştır, çalıştırınca şifre verecek
  ** Bu komut yukarıdaki helm komutundan sonra da terminalde çıkmıştı, istersen terminalden - istersen ekrandan kopyala.

```bash (pwd : petclinic-microservices-with-db)
# ekranda çıkan komut:
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
```

* Aldığın şifreyi gir -->> Log in with Local User -->> Kendi şifreni oluştur (Alttaki 2 kutuyu da işaretle) -->> Continue ==>> Dashboard Geldi.
  ``user     : admin``
  ``password : petclinic123456``

* If bootstrap pod is not initialized or you forget your admin password you can use the below command to reset your password.

# Rancher admin şifreni unuttuysan aşağıdaki komutlar ile reset atıp tekrar şifre alabilirsin

```bash
export KUBECONFIG=~/.kube/config
kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- reset-password
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 25 - Create Staging and Production Environment with Rancher ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Rancher-server ayağa kaldır, arayüze bağlan, Dashboard aç.
  ``rancher.devopsalparslanugurer.com``
  ``user     : admin``
  ``password : petclinic123456``

# Cluster ayağa kaldırmadan önce Rancher'ın cloud service'lerine erişimi için `Cloud Credentials` oluşturmalıyız:
* To provide access of Rancher to the cloud resources, create a `Cloud Credentials` on `Cluster Management` segment for AWS on Rancher and name it as `Petclinic-AWS-Training-Account`. As region select `us-east-1`.

  * Cluster Management -- Cloud Credentials -- Create -- Amazon -->>
                                                                      -   Name           : Petclinic-AWS-Training-Account
                                                                      -   Description    : Petclinic-AWS-Training-Account
                                                                      -   Access Key     : *******AWS Access Key*******
                                                                      -   Secret Key     : *******AWS Secret Key*******
                                                                      -   Default Region : us-east-1
                                                                    Create

# Cluster ayağa kaldır. Cluster ayağa kalkması biraz vakit alıyor, cluster ayağa kalkarken biz de MSP 26 adımına geçer ve nexus'daki ayar/değişiklikleri yaparız.
* Create a Kubernetes cluster using Rancher (Cluster Management --> Clusters) with RKE and new nodes in AWS  and name it as `petclinic-cluster-staging`.

```yml
Region            : us-east-1
Security group    : create new sg (rancher-nodes)
Instance Type     : t3a.medium
Root Disk Size    : 16 GB
AMI (RancherOS)   : ami-02fe87f853d560d52
SSH User          : rancher
Label             : os=rancheros
```
# Hamburger Menu -->> Cluster Management --> Cluster - Create -->> Amazon EC2 -->>  Keep `RKE2/K3s` selected
                                                        - Cluster Name        : petclinic     (petclinic-cluster-staging)
                                                        - Cluster Description : petclinic project
                                                        - Cloud Credential    : There is only one and it appeared automatically.
                                                        - Pool Name           : pool1
                                                        - Machine Count       : 3 ([raft.github.io](https://raft.github.io/) -> raft consensus algorithm)
                                                        - Zone                : a
                                                        - Roles               : etcd , Control Plane and Worker (select all)
                                                        - Region              : us-east-1
                                                        - Root Disk Size      : Default 16
                                                        - Instance Type       : t3a.medium
                                                        - VPC/Subnet          : default-1a
                                                    Create

# Cluster ayağa kalkarken nexus'a gidelim

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 26 - Prepare Nexus Server ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

# Bu MSP-26 diğer adımlardan tamamen bağımsız, sadece Nexus repo ile bir proje nasıl kullanılır görmek için.

```bash (pwd : petclinic-microservices-with-db)
# Create `feature/msp-26` branch from `release`.
git checkout release
git branch feature/msp-26
git checkout feature/msp-26
```

# Nexus-server ayağa kaldırmak için ya proje klasöründeki `terraform` dosyalarını lokalden kullanacağız ya da `manuel` olarak kuracağız. Nexus'u `docker` ile 1 ``container`` içresinde kuracağım.

* Proje klasöründe `msp-26-nexus-server-template` içindeki terraform dosyalarını kullanarak aşağıdaki özelliklerde 1 adet `Nexus Server` set up edelim. `nexus.auto.tfvars` dosyasındaki key değiştirmeyi unutma. Sonra remote ssh ile bağlan.
      - t3a.medium (Nexus needs 8 GB of RAM)
      - Amazon Linux 2023 AMI
      - security group allowing `SSH (22)` and `Nexus Port (8081)` connections

* Terraform dosyası kullanmadan Nexus server set up etmek için : yukarıdaki özelliklerde bir ec2 ayağa kaldır, remote ssh ile bağlan ve aşağıdaki komutları çalıştır.
<!-- - Aynı terraform dosyasını `nexus-server.tf` ismi ile `infrastructure` klasörü altına kaydet.  -->

``` bash  (manuel kurulum komutları) (nexus server) (/home/ec2-user)
#! /bin/bash
# update os
sudo yum update -y
# set server hostname as jenkins-server
sudo hostnamectl set-hostname nexus-server
# install docker
sudo yum install docker -y
# start docker
sudo systemctl start docker
# enable docker service
sudo systemctl enable docker
# add ec2-user to docker group
sudo usermod -aG docker ec2-user
newgrp docker
# create a docker volume for nexus persistent data
sudo docker volume create --name nexus-data
# run the nexus container  
# hub.docker.com -->> nexus3  arattır  --->> sonatype/nexus3 (en üstte çıkan)
sudo docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3
```

``` bash (nexus server)
exit
docker container ps   # sonatype/nexus3   İSİMLİ image gör, container ismi de nexus
docker volume ls      # nexus-data
```

* Browser'da `http://<nexus-server-petclinic public IP>:8081` yaz ve  `Sign in` ol.
      - Username : `admin`
      - Password : sign in kutucuğundaki `/nexus-data/admin.password` ifadesini kopyala

# password almak için cnxus'un çalıştığı container içindeki dosyayı okumamız gerekiyor
``` bash (nexus server)
docker container ps   # sonatype/nexus3   İSİMLİ image gör, container ismi de nexus
docker exec nexus pwd   # opt/sonatype   # çalışan nexus container'ı içinde komut çalıştırıyorum
docker exec nexus cat /nexus-data/admin.password  # ilk password verir , BU KOMUTU AL nexus sign in sayfasına yaz
```

- Yukarıdaki komut çıktısı şifreyi yapıştır  -->> Sign in -->> Next -->> Yeni Şifre gir: 123456 -->> Disable anonymous access -->> Next -->> Finish : Nexus Hazır.

### Configure the app for Nexus Operation

* Nexus : Artifact repo, son ürünlerin saklandığı repo, Deploy edilmeye hazır dosyalarımızı sakladığımız repo.

* Maven için lokalde central repo neresi : .m2 -->> Doğrudan Maven Central repodan çekiyor -->> Araya Proxy (Nexus) repo ekleyeceğiz ki doğrudan Maven Central repodan değil de cache'leme yaptığı proxy repodan çeksin ==>>> Yani tekrar ./mvnw clean komutunu çalıştırdığımda bu proxy/ara repodan çekecek.
  - maven (central) repo --->> .m2 (local) Repo
  - maven (central) repo --->> Nexus (Proxy) Repo -->> --->> .m2 (local) Repo

* `/home/ec2-user/.m2` klasörü altında ``settings.xml``isimli bir dosya oluştur:

# Nexus-server Private IP değiştir , Nexus Şifre ve Kullanıcı adını değiştir

```xml  (settings.xml)
<settings>
  <mirrors>
    <mirror>
      <!--This sends everything else to /public -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>  <!-- * Yani her hangi bir repoyu bu nexus repo'ya cache'le diyoruz, buraya * işareti yerine sadece cache'lenmesini istediğim repo ismini de yazabilirim. -->
      <url>http://<AWS private IP>:8081/repository/maven-public/</url>  <!--Nexus-server Private IP değiştir -->
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
<activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>          <!--Nexus Şifre ve Kullanıcı adını değiştir : admin -->
      <password>your-password</password>  <!--Nexus Şifre ve Kullanıcı adını değiştir : 123456 -->
    </server>
  </servers>
</settings>
```

# `/home/ec2-user/.m2` altındaki `repository` klasörünü sil, aşağıdaki `./mvnw clean` komutunu gir ve dependency'lerin Nexus-server'dan indiğini gör.
# Aynı zamanda `komutu girmeden önce` browser'da Nexus-server'a git, Browse -->> maven-central/public/release  içlerinin boş olduğunu göster.

``` bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
./mvnw clean
```

* Üretmiş olduğum jar dosyaları nexus repo'ya yani `maven-release`e gönderirim, ihtiyacım olduğunda da oradan çekerim. 

* Peki jar dosyalarımı nasıl gönderirim :

# Petclinic-microservices-with-db  klasörü (proje ana dizini) içinde bulunan `pom.xml` dosyası içine `</dependencyManagement>`'den sonra 77. satırdan itibaren aşağıdaki kısmı ekle. `Nexus-server Private IP değiştir`

```xml
<distributionManagement>
  <repository>
    <id>nexus</id>
    <name>maven-releases</name> <!--Çalıştığın branch releases ise, yani kod hazır ise buraya gönder-->
    <url>http://<AWS private IP>:8081/repository/maven-releases/</url> <!--Nexus-server Private IP değiştir-->
  </repository>
  <snapshotRepository>
    <id>nexus</id>
    <name>maven-snapshots</name>  <!--Çalıştığın branch snapshots ise, yani halen üzerinde çalışıyorsan buraya gönder-->
    <url>http://<AWS private IP>:8081/repository/maven-snapshots/</url> <!--Nexus-server Private IP değiştir-->
  </snapshotRepository>
</distributionManagement>
```

# Aşağıdaki komutu çalıştırın, Create edilen artifact'ler nexus-releases reposunda depolanacak.
  - <https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#a-build-lifecycle-is-made-up-of-phases> :
  * validate -> validate the project is correct and all necessary information is available
  * compile  -> compile the source code of the project
  * test     -> test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
  * package  -> take the compiled code and package it in its distributable format, such as a JAR.
  * verify   -> run any checks on results of integration tests to ensure quality criteria are met
  * install  -> install the package into the local repository, for use as a dependency in other projects locally `lokaldeki .m2 klasörüne koyar`
  * deploy   -> done in the build environment, copies the final package to the remote repository for sharing with other developers and projects. `Remote olan nexus-server'a koyar`

```bash (pwd : /home/ec2-user/petclinic-microservices-with-db-de)
./mvnw clean deploy
```

- Yukarıdaki komut sonunda success alınca browser'da nexus repoyu aç, Browse -- maven-releases -- GELMİŞ OLDUĞUNU GÖR !

# Aynı komutu `./mvnw clean deploy` tekrar girersen hata alırsın, Çünkü `releases` repoya tekrar göndereceksen `pom.xml` içerisindeki `version` bilgisini değiştirmelisin ki yeni bir release versiyonu olarak gitsin. Versiyon değiştirmediğin için ve releases repo üzerine overwrite yapmadığından hata aldın

# Note: Eğer releases repoya aynı artifacat'i redeploy etmek istiyorum derseniz ==>> Çark simgesi(server administration and configuration) -->> Repositories -->> maven-releases -->> `Deployment policy : "Allow redeploy"` olmalı. TAVSİYE EDİLEN BİR DAVRANIŞ DEĞİLDİR !!!

(nexus server --> server configuration --> repositories --> maven releases --> Deployment policy : ``Allow redeploy``)

```bash (pwd : petclinic-microservices-with-db)
git add .
git commit -m 'added Nexus server terraform files'
git push --set-upstream origin feature/msp-26
git checkout release
git merge feature/msp-26
git push origin release
```

* Nexus server terminate et.

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 27 - Prepare a Staging Pipeline ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash (pwd : petclinic-microservices-with-db)
# Create `feature/msp-27` branch from `release`.
git checkout release
git branch feature/msp-27
git checkout feature/msp-27
```

```bash (pwd : petclinic-microservices-with-db)
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="clarusway-repo/petclinic-app-staging"
AWS_REGION="us-east-1"

aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \
  --image-scanning-configuration scanOnPush=false \
  --image-tag-mutability MUTABLE \
  --region ${AWS_REGION}
```

* Prepare a script to create ECR tags for the staging docker images and name it as `prepare-tags-ecr-for-staging-docker-images.sh` and save it under `jenkins` folder.

``` bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

* Prepare a script to build the staging docker images tagged for ECR registry and name it as `build-staging-docker-images-for-ecr.sh` and save it under `jenkins` folder.

``` bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```

* Prepare a script to push the staging docker images to the ECR repo and name it as `push-staging-docker-images-to-ecr.sh` and save it under `jenkins` folder.

``` bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```

# Rancher Arayüz kullanımı nasıl olur :
* Rancher Arayüz aç : Hamburger/3 çizgili'ye tıkla -- petclinic -- Workloads -- Pods --> Create:
                                                                                      - Namespace : default
                                                                                      - Name            : mynginx
                                                                                      - Container Mame  : nginx
                                                                                      - Container Image : nginx
                                                                                      - Networking - Add Port or Service
                                                                                      - Name            : ng
                                                                                      - Private Container Port : 80
                                                                                      - Service Type           : NodePort
                                                                                    Create
                                                        Services -->> mynginx --->> Yan tarafta 3 nokta üst üste tıkla : Edit YAML
                                                        YAML dosyasında değişiklik yap (type: NodePort) -->> Save -->> 31386/TCP
                                                        AWS Console - EC2 - petlinic-pool ec2'lardan birinin Public IP al -- browser'a yaz  <http://3.17.58.237:31386> ==>> nginx sayfası geldi

# Repoda  ilave hands-on var -->> ``kubernetes-17-daemonsets-jobs-cronjobs``

# Yukarıdakinin yerine jenkins-server üzerinden `Rancher CLI` kurup rancher'ı CLI ile yönetebilirim:

```bash (pwd : home/ec2-user)
# Install `Rancher CLI` on Jenkins Server.
curl -SsL "https://github.com/rancher/cli/releases/download/v2.7.0/rancher-linux-amd64-v2.7.0.tar.gz" -o "rancher-cli.tar.gz"
tar -zxvf rancher-cli.tar.gz
sudo mv ./rancher*/rancher /usr/local/bin/rancher
chmod +x /usr/local/bin/rancher
rancher --version   # v2.7.0
```

# Jenkins'ın Rancher'ı CLI ile yönetebilmesi için Rancher'dan `credentials` alıp Jenkins'da tanımlamalıyız

  * Rancher API Key : <https://rancher.com/docs/rancher/v2.x/en/user-settings/api-keys/#creating-an-api-key>

* Rancher Arayüz aç -- Avatar resmine tıkla (sağ üst köşe) -- Account & API Keys -- Create API Key -->>
                                                                                    - Descripton: jenkins --> Scope : No Scope --> Automatically expire : Never
                                                                                  Create
                                                                              `Çıkan Sayfadaki Şifreleri Bir yere Kaydet Kaybetme`
                                                                                Done
------>>>
# ``token-86rbc:c7j76lrp4zwf6ccx878gsd2vgc29wmctbgsdxdvgd2twjnlgnpxq2g`` : Rancher'dan aldık aşağıda Jenkins'e gireceğiz

# Jenkins'e bağlan http://50.16.37.97:8080/ , user : petclinic  , password : aU063470
  * jenkins server Dashboard, select ***Manage Jenkins --> Credentials --> System -->   Global credentials (unrestricted) --> Add Credentials***.

```yml
- Kind    : Username with password
- username: Access Key -->  token-86rbc
- password: Secret Key -->  c7j76lrp4zwf6ccx878gsd2vgc29wmctbgsdxdvgd2twjnlgnpxq2g
- id      : rancher-petclinic-credentials
Create
```

# Uygulama için Rancher dashboard üzerinde ayrı bir  ``namespace`` oluşturacağız. 
* Create `petclinic-staging-ns` namespace on `petclinic (petclinic-cluster-staging)` with Rancher

  - Project:Default -->> Create Namespace --> Name : `petclinic-staging-ns` --->> Create

  - projects/namespaces ---> Project:Default --->> petclinic-staging-ns namespace ---> sağ tarafta 3 nokta (şiş kebap menu) ---> Edit yaml  --->     copy projectId  ===>>>>   c-m-p7wjr9t2:p-xzw6s (c-m-p7wjr9t2 = cluster name  ; p-xzw6s = project/namespace name )


* Prepare a Jenkinsfile for `petclinic-staging` pipeline and save it as `jenkinsfile-petclinic-staging` under `jenkins` folder.

``` groovy
pipeline {
    agent any
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="clarusway-repo/petclinic-app-staging"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        RANCHER_URL="https://rancher.devopsalparslanugurer.com"     // DEĞİŞTİRMEYİ UNUTMA
        // Get the project-id from Rancher UI (projects/namespaces --> petclinic-staging-ns namespace --> Edit yaml --> copy projectId )
        RANCHER_CONTEXT="c-m-p7wjr9t2:p-xzw6s"  // DEĞİŞTİRMEYİ UNUTMA
       //First part of projectID
        CLUSTERID="c-m-p7wjr9t2"  // Yukarıdaki RANCHER_CONTEXT'in :'den önceki kısmını al  DEĞİŞTİRMEYİ UNUTMA
        RANCHER_CREDS=credentials('rancher-petclinic-credentials')  // jenkins'te verdiğim isim
    }
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Staging Docker Images') {
            steps {
                echo 'Preparing Tags for Staging Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-staging-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Staging Docker Images') {
            steps {
                echo 'Building App Staging Images'
                sh ". ./jenkins/build-staging-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-staging-docker-images-to-ecr.sh"
            }
        }
        stage('Deploy App on Petclinic Kubernetes Cluster'){  // Bu adım weekly pipeline'dan farklı
            steps {
                echo 'Deploying App on K8s Cluster'
                sh "rancher login $RANCHER_URL --context $RANCHER_CONTEXT --token $RANCHER_CREDS_USR:$RANCHER_CREDS_PSW"  // Jenkins'ten Rancher'a login oluyorum, artık rancher komutlarını kullanabilirim.
                sh "envsubst < k8s/petclinic_chart/values-template.yaml > k8s/petclinic_chart/values.yaml"
                sh "sed -i s/HELM_VERSION/${BUILD_NUMBER}/ k8s/petclinic_chart/Chart.yaml"
                sh "rancher kubectl delete secret regcred -n petclinic-staging-ns || true"  // regcred varsa siliyorum
// config.json dosyasının açıklaması:
//                                    - `.docker`   isminde dosya sistemi/dizini var ne zaman oluştu bu ==>> `push-dev-docker-images-to-ecr.sh`  isimli script içerisinde `docker login` komutu var, biz job çalıştırınca bu script çalıştı ve "docker login" komutu çalıştı --> bu komut bir `credential` oluşturuyor ve komutun çalıştığı makinanın `home` dizininde `.docker` oluşuyor ve bu credential'ı `home` dizinde oluşan .docker altına koyuyor. ``ECR repodan/repoya image pull/push`` edebilmesi için repoya girme yetkisi veren credentials.
//                                   - Yani bu job'ı jenkins user çalıştırdığına göre, jenkins user'ın home dizini `/var/lib/jenkins içerisinde` `.docker` oluşuyor --->>> `/var/lib/jenkins/.docker` altında da `config.json` oluşuyor ve credential bunun içerisinde.

// - Rancher petclinic-staging-ns isimli namespace içindeki petclinic cluster'daki node'ların ECR repodan image çekmesi için repoya girme yetkisi olan credentials'a ihtiyacı var ===>>> jenkins içindeki `config.json` dosyasından `regcred` isminde `secret` oluştur----->>> Bu regcred` ismindeki secret `deployment.yaml` dosyalarında da tanımlanmıştı ===>>> Yani petclinic cluster'daki node'lar artık recgred'e dolayısıyla credentials'a sahipler ve böylece ECR repodan/repoya image pull/push yapabilecekler.
                sh """
                rancher kubectl create secret generic regcred -n petclinic-staging-ns \
                --from-file=.dockerconfigjson=$JENKINS_HOME/.docker/config.json \
                --type=kubernetes.io/dockerconfigjson
                """
                sh "rm -f k8s/config"
                sh "rancher cluster kf $CLUSTERID > k8s/config" // rancher'ın config dosyası içeriğini  k8s/config'e aktarıyorum
                sh "chmod 400 k8s/config"
                sh "helm repo add stable-petclinic s3://petclinic-helm-charts-arrow/stable/myapp/"  // DEĞİŞTİRMEYİ UNUTMA
                sh "helm package k8s/petclinic_chart"
                sh "helm s3 push --force petclinic_chart-${BUILD_NUMBER}.tgz stable-petclinic"
                sh "helm repo update"
                sh "AWS_REGION=$AWS_REGION helm upgrade --install petclinic-app-release stable-petclinic/petclinic_chart --version ${BUILD_NUMBER} --namespace petclinic-staging-ns --kubeconfig k8s/config"
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

# `rancher login` açıklaması :
```bash (pwd : petclinic-microservices-with-db)
# RANCHER_CONTEXT="c-m-p7wjr9t2:p-xzw6s"   ,  (c-m-p7wjr9t2 = cluster name  ; p-xzw6s = project/namespace name )
# Jenkins'ın Rancher'ı CLI ile yönetebilmesi için Rancher'dan `credentials` alıp Jenkins'da tanımladık : 
# password: Secret Key -->token-86rbc:c7j76lrp4zwf6ccx878gsd2vgc29wmctbgsdxdvgd2twjnlgnpxq2g
# id      : rancher-petclinic-credentials

rancher login https://rancher.devopsalparslanugurer.com --context c-m-p7wjr9t2:p-xzw6s --token token-86rbc:c7j76lrp4zwf6ccx878gsd2vgc29wmctbgsdxdvgd2twjnlgnpxq2g
# jenkins'ten rancher'a bağlandım login oldum , artık rancher kubectl ile cluster'ımı yönetebilirim.

rancher kubectl get node
      petclinic-pool1-a0688060-ll8vk   Ready    control-plane,etcd,master,worker   3h17m   v1.26.8+rke2r1
      petclinic-pool1-a0688060-mmx69   Ready    control-plane,etcd,master,worker   3h13m   v1.26.8+rke2r1
      petclinic-pool1-a0688060-vdjqx   Ready    control-plane,etcd,master,worker   3h12m   v1.26.8+rke2r1

rancher kubectl get svc
      NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   3h20m
```

# `sh "rancher cluster kf $CLUSTERID > k8s/config"` açıklaması : CLUSTERID="c-m-p7wjr9t2"
  
  - `rancher cluster kf $CLUSTERID > k8s/config` ile  rancher'ın config dosyası içeriğini  k8s/config'e aktarıyorum ki rancher'da oluşturduğum 3 node'luk petclinic isimli cluster'ıma erişebileyim

```bash (pwd : /home/ec2-user)
kubectl get node # BU KOMUT .kube altındaki config'e bakar ;# BU KOMUT İLE DE rancher cluster'A ULAŞIYORUM, YANİ 1 NODE BİLGİSİ GELİYOR
        NAME           STATUS   ROLES                      AGE   VERSION
        172.31.40.33   Ready    controlplane,etcd,worker   24h   v1.25.9

# rancher'dan cluster'ımın config dosyasını almam lazım ==>> Bunu rancher dashboard üzerinden de yapabilirim, BURADA KOMUTLA YAPIYORUM
rancher cluster kf c-m-p7wjr9t2 > config # rancher'daki petclinic isimli cluster'ıma ait config dosyamın içeriğini aldım ve kaydettim
ls    # config  petclinic-microservices-with-db-de  rancher-cli.tar.gz  rancher-v2.7.0
kubectl get node --kubeconfig config  # BU KOMUT İLE RANCHER İÇERİSİNDE OLUŞTURDUĞUM petclinic İSİMLİ cluster'A ULAŞIYORUM VE 3 TANE NODE LİSTELİYOR.
            NAME                             STATUS   ROLES                              AGE     VERSION
            petclinic-pool1-a0688060-ll8vk   Ready    control-plane,etcd,master,worker   3h36m   v1.26.8+rke2r1
            petclinic-pool1-a0688060-mmx69   Ready    control-plane,etcd,master,worker   3h32m   v1.26.8+rke2r1
            petclinic-pool1-a0688060-vdjqx   Ready    control-plane,etcd,master,worker   3h31m   v1.26.8+rke2r1
```

```bash (pwd : petclinic-microservices-with-db)
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added jenkinsfile petclinic-staging for release branch'
git push --set-upstream origin feature/msp-27
git checkout release
git merge feature/msp-27
git push origin release
```

* Create a Staging ``Pipeline`` on Jenkins with name of `petclinic-staging` with following script and configure a `cron job` to trigger the pipeline every Sundays at midnight (`59 23 * * 0`) on `release` branch. `Petclinic staging pipeline` should be deployed on permanent staging-environment on `petclinic-cluster` Kubernetes cluster under `petclinic-staging-ns` namespace.

```yml
- job name: petclinic-staging
- job type: pipeline
- Build Triggers:
      Build periodically: 59 23 * * 0   # every Sundays at midnight
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */release
- Pipeline:
      Script Path: jenkins/jenkinsfile-petclinic-staging
```

- Click `save` -->> Click `Build Now`

```bash (pwd : petclinic-microservices-with-db)
# Rancher Arayüz'den de görebilirsin bu şekilde komutlarla da
rancher kubectl get no
rancher kubectl get svc
```

* Önce AWS Console EC2 -->> petclinic-pool ismiyle baŞlayan cluster ec2'lardan bir tanesinin Public IP'sini kopyala
* AWS Console Route53 -->> devopsalparslanugurer.com -->> Create record -->> Record Name: `petclinic`
                                                                             Record type: `A`
                                                                             Value/Route traffic to : Public IP (yukarıda aldığın)
                                                                             Create records

* Browser'da : petclinic.devopsalparslanugurer.com  ---->> UYGULAMA GELMEDİ ==>> AŞAĞIDAKİ İŞLEMİ YAPARSAN GELECEK

# ingress Edit:
* Rancher Arayüz aç : hamburger menu/3 çizgili'ye tıkla -- petclinic -- Network Policies -- Namespace:petclinic-staging-ns
                                                                             k8s-default  sağ tarafta  3 nokta üst üste --- Tıkla --- Edit YAML --- yaml dosyasında : ingress kısmı aşağıdaki gibi olacak
                                                                             Save
```yaml
# Network policy object ile sadece aynı labels'a sahip podlara erişim izni vererek her pod'un her pod'a erişimini kısıtlayarak güvenliği sağlamış oluyor.
# ingress altındaki podSelector:matchLabels: kısmı ile alt tarafta bulunan podSelector:matchLabels: kısmı aynı olan pod'lar birbirine erişebilir demek
# Ama dış dünyadan erişemiyoruz, erişmek için ilave portlar açmak lazım.

# Aşağıdaki gibi yaparak ilave port açmadan dış dünyadan ulaşabilelim diye, Aslında her yerden ulaşıma açtık -->> ingress aşağıdaki gibi olacak
  ingress:
    - {}

# SON HALİ :
spec:
  ingress:
    - {}
  podSelector:
    matchLabels:
      io.kompose.network/k8s-default: 'true'
  policyTypes:
    - Ingress
status: {}
```

# Browser'da : petclinic.devopsalparslanugurer.com  ---->> UYGULAMA GELDİ

* Rancher Arayüz bağlan ve petclinic cluster delete et.

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 28 - Prepare a Production Pipeline ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

* Create `feature/msp-28` branch from `release`.

``` bash
git checkout release
git branch feature/msp-28
git checkout feature/msp-28
```

# İlave olarak `RDS` kullanımı ve `http` -->> `https` haline nasıl getiririz, sertifika nasıl alırız.

* Switch user to jenkins for creating eks cluster. Execute following commands as `jenkins` user.

```bash (home/ec2-user  jenkins-server)
sudo su - jenkins
pwd    # /var/lib/jenkins
cat cluster.yaml    # cluster.yaml  gözüktü  ,yoksa dosyayı içeriği aşağıdaki gibi olacak şekilde yeniden oluştur.
```

```yaml (cluster.yaml)
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: petclinic-cluster
  region: us-east-1
availabilityZones: ["us-east-1a", "us-east-1b", "us-east-1c"]
managedNodeGroups:
  - name: ng-1
    instanceType: t3a.medium
    desiredCapacity: 2
    minSize: 2
    maxSize: 3
    volumeSize: 8
```

# EKS cluster'ı `eksctl`aracılığıyla oluşturalım. Bu komut biraz zaman alacak, terminal akmaya devam etsin, yeni bir terminal aç.

```bash (pwd : /var/lib/jenkins)
eksctl create cluster -f cluster.yaml
# bu komut ile birlikte   /var/lib/jenkins  dizini altında  .kube  dizini oluştu ---içindeki dosyalar--->> cache   config   ==>> işte jenkins bu "config" dosyası üzerinden cluster'ı görüp yönetebiliyor.
```

# Yukarıdaki komutun çalışması ve EKS Cluster oluşması biraz zaman alacak, bu işlem sürerken diğer işlemleri yapalım. ECR Repomuzu oluşturalım.
  * Job ile yapmaya gerek yok direkt komut ile ECR repo oluştur.

* Create a ``Jenkins Job`` and name it as `create-ecr-docker-registry-for-petclinic-prod` to create Docker Registry for `Production` manually on AWS ECR.

```yml
- job name: create-ecr-docker-registry-for-petclinic-prod
- job type: Freestyle project
- Build:
      Add build step: Execute Shell
      Command:
```

```bash (pwd : petclinic-microservices-with-db)
PATH="$PATH:/usr/local/bin"
APP_REPO_NAME="clarusway-repo/petclinic-app-prod"
AWS_REGION="us-east-1"

aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
aws ecr create-repository \
  --repository-name ${APP_REPO_NAME} \
  --image-scanning-configuration scanOnPush=false \
  --image-tag-mutability MUTABLE \
  --region ${AWS_REGION}
```

* `jenkins` klasörü altında `production docker image`'lere ECR tags oluşturmak için `prepare-tags-ecr-for-prod-docker-images.sh` isminde script oluştur.

``` bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version)
export IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
export IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
export IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
```

* `jenkins` klasörü altında `production docker image`'leri build etmek için `build-prod-docker-images-for-ecr.sh` isminde script oluştur.

``` bash
docker build --force-rm -t "${IMAGE_TAG_ADMIN_SERVER}" "${WORKSPACE}/spring-petclinic-admin-server"
docker build --force-rm -t "${IMAGE_TAG_API_GATEWAY}" "${WORKSPACE}/spring-petclinic-api-gateway"
docker build --force-rm -t "${IMAGE_TAG_CONFIG_SERVER}" "${WORKSPACE}/spring-petclinic-config-server"
docker build --force-rm -t "${IMAGE_TAG_CUSTOMERS_SERVICE}" "${WORKSPACE}/spring-petclinic-customers-service"
docker build --force-rm -t "${IMAGE_TAG_DISCOVERY_SERVER}" "${WORKSPACE}/spring-petclinic-discovery-server"
docker build --force-rm -t "${IMAGE_TAG_HYSTRIX_DASHBOARD}" "${WORKSPACE}/spring-petclinic-hystrix-dashboard"
docker build --force-rm -t "${IMAGE_TAG_VETS_SERVICE}" "${WORKSPACE}/spring-petclinic-vets-service"
docker build --force-rm -t "${IMAGE_TAG_VISITS_SERVICE}" "${WORKSPACE}/spring-petclinic-visits-service"
docker build --force-rm -t "${IMAGE_TAG_GRAFANA_SERVICE}" "${WORKSPACE}/docker/grafana"
docker build --force-rm -t "${IMAGE_TAG_PROMETHEUS_SERVICE}" "${WORKSPACE}/docker/prometheus"
```

* `jenkins` klasörü altında `production docker image`'leri ECR repo'ya push etmek için `push-prod-docker-images-to-ecr.sh` isminde script oluştur.

``` bash
aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker push "${IMAGE_TAG_ADMIN_SERVER}"
docker push "${IMAGE_TAG_API_GATEWAY}"
docker push "${IMAGE_TAG_CONFIG_SERVER}"
docker push "${IMAGE_TAG_CUSTOMERS_SERVICE}"
docker push "${IMAGE_TAG_DISCOVERY_SERVER}"
docker push "${IMAGE_TAG_HYSTRIX_DASHBOARD}"
docker push "${IMAGE_TAG_VETS_SERVICE}"
docker push "${IMAGE_TAG_VISITS_SERVICE}"
docker push "${IMAGE_TAG_GRAFANA_SERVICE}"
docker push "${IMAGE_TAG_PROMETHEUS_SERVICE}"
```

* `jenkins` klasörü altında QA environment'da uygulamayı deploy etmek için `deploy_app_on_prod_environment.sh` isminde script oluştur.

```bash
# 4.satır :  chart repo ismini değiştirmeyi unutma "s3://petclinic-helm-charts-arrow"
echo 'Deploying App on Kubernetes'
envsubst < k8s/petclinic_chart/values-template.yaml > k8s/petclinic_chart/values.yaml
sed -i s/HELM_VERSION/${BUILD_NUMBER}/ k8s/petclinic_chart/Chart.yaml
AWS_REGION=$AWS_REGION helm repo add stable-petclinic s3://petclinic-helm-charts-arrow/stable/myapp/ || echo "repository name already exists"
AWS_REGION=$AWS_REGION helm repo update
helm package k8s/petclinic_chart
AWS_REGION=$AWS_REGION helm s3 push --force petclinic_chart-${BUILD_NUMBER}.tgz stable-petclinic
kubectl create ns petclinic-prod-ns || echo "namespace petclinic-prod-ns already exists"
kubectl delete secret regcred -n petclinic-prod-ns || echo "there is no regcred secret in petclinic-prod-ns namespace"
kubectl create secret generic regcred -n petclinic-prod-ns \
    --from-file=.dockerconfigjson=/var/lib/jenkins/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
AWS_REGION=$AWS_REGION helm repo update
AWS_REGION=$AWS_REGION helm upgrade --install \
    petclinic-app-release stable-petclinic/petclinic_chart --version ${BUILD_NUMBER} \
    --namespace petclinic-prod-ns
```

* Uygulama database'i için ``mysql`` kullandık, ama cluster'da database container'da muhafaza edilir, yani güvenilir değil. Bu aşamada mysql pod and service'leri yerine ``Amazon RDS`` kullanacağız.
* AWS Console üzerinde AWS RDS ayağa kaldıracağız:

```yml
  - Database:
  - Create Database:
  - Choose a database creation method: Standart create
  - Engine options: MySQL
  - Engine Version : 5.7.39
  - Templates: Free tier
  - Settings:
  - DB instance identifier: petclinic
  - Master username: root   # developer kodu bu şekilde yazmış
  - Master password: petclinic  # developer kodu bu şekilde yazmış
  - Confirm master password: petclinic  # developer kodu bu şekilde yazmış
  - Public access: Yes
  - Existing VPC security groups: default   #### DEFAULT SECURITY GROUP ===>>> 3306 port açın
  - Additional configuration:
    - Initial database name: petclinic
Create database
```

* EKS cluster'ı ayağa kalkınca (terminal akışı durunca) jenkins-user olarak `ingress controller` kuracağız.
* - After the cluster is up, run the following command to install `ingress controller`.


```bash (pwd : /var/lib/jenkins   , jenkins-user)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
kubectl get no    # 2 tane node/instance gördüm
      NAME                              STATUS    ROLES     AGE     VERSION
      ip-192-168-23-238.ec2.internal    Ready     <none>    7m41s   v1.25.9-eks-0a21954
      ip-192-168-66-9.ec2.internal      Ready     <none>    7m48s   v1.25.9-eks-0a21954

```

# `RDS` kullanacağımız için mysql deployment ve service'e ihtiyaç yok, object'lerin yaml dosyalarını silelim/update ededim

- Kubernetes'te 4 tane `Service tye` var : ClusterIP(default) , NodePort , LoadBalancer , ExternalName. Bizim burada kullanacağımız service tipi `ExternalName`

* Delete `mysql-server-deployment.yaml` file from `k8s/petclinic_chart/templates` folder.

* Update `k8s/petclinic_chart/templates/mysql-server-service.yaml` aşağıdaki gibi:  `spec'ten sonrası değişecek -> RDS Endpoint`

```yaml (mysql-server-service.yaml)
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert -f docker-compose-local-db.yml
    kompose.version: 1.28.0 (c4137012e)
  labels:
    io.kompose.service: mysql-server
  name: mysql-server
spec:
  type: ExternalName
  externalName: petclinic.cbanmzptkrzf.us-east-1.rds.amazonaws.com # Change this line with the endpoint of your RDS.
```

* `jenkins` klasörü altında `petclinic-prod` pipeline'ı için `jenkinsfile-petclinic-prod` isminde bir Jenkinsfile oluştur.

```groovy
pipeline {
    agent any
    environment {
        PATH=sh(script:"echo $PATH:/usr/local/bin", returnStdout:true).trim()
        APP_NAME="petclinic"
        APP_REPO_NAME="clarusway-repo/petclinic-app-prod"
        AWS_ACCOUNT_ID=sh(script:'export PATH="$PATH:/usr/local/bin" && aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh ". ./jenkins/package-with-maven-container.sh"
            }
        }
        stage('Prepare Tags for Production Docker Images') {
            steps {
                echo 'Preparing Tags for Production Docker Images'
                script {
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_ADMIN_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:admin-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-api-gateway/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_API_GATEWAY="${ECR_REGISTRY}/${APP_REPO_NAME}:api-gateway-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-config-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CONFIG_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:config-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-customers-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_CUSTOMERS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:customers-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-discovery-server/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_DISCOVERY_SERVER="${ECR_REGISTRY}/${APP_REPO_NAME}:discovery-server-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-hystrix-dashboard/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_HYSTRIX_DASHBOARD="${ECR_REGISTRY}/${APP_REPO_NAME}:hystrix-dashboard-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-vets-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VETS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:vets-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    MVN_VERSION=sh(script:'. ${WORKSPACE}/spring-petclinic-visits-service/target/maven-archiver/pom.properties && echo $version', returnStdout:true).trim()
                    env.IMAGE_TAG_VISITS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:visits-service-v${MVN_VERSION}-b${BUILD_NUMBER}"
                    env.IMAGE_TAG_GRAFANA_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:grafana-service"
                    env.IMAGE_TAG_PROMETHEUS_SERVICE="${ECR_REGISTRY}/${APP_REPO_NAME}:prometheus-service"
                }
            }
        }
        stage('Build App Production Docker Images') {
            steps {
                echo 'Building App Production Images'
                sh ". ./jenkins/build-prod-docker-images-for-ecr.sh"
                sh 'docker image ls'
            }
        }
        stage('Push Images to ECR Repo') {
            steps {
                echo "Pushing ${APP_NAME} App Images to ECR Repo"
                sh ". ./jenkins/push-prod-docker-images-to-ecr.sh"
            }
        }
        stage('Deploy App on Petclinic Kubernetes Cluster'){
            steps {
                echo 'Deploying App on K8s Cluster'
                sh ". ./jenkins/deploy_app_on_prod_environment.sh"
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

```bash (pwd : petclinic-microservices-with-db)
# Commit the change, then push the script to the remote repo.
git add .
git commit -m 'added jenkinsfile petclinic-production for main branch'
git push --set-upstream origin feature/msp-28
git checkout release
git merge feature/msp-28
git push origin release
```

```bash (pwd : petclinic-microservices-with-db)
# Merge `release` into `main` branch to build and deploy the app on `Production environment` with pipeline.
git checkout main
git merge release
git push origin main
```

* Jenkins'de `petclinic-prod` isminde `Production Pipeline` oluşturacağız ve `main` branch'ına her `commit` yapıldığında pipeline'ı trigger edecek şekilde `github-webhook` ekleyeceğiz.
* `Petclinic production pipeline`, `petclinic-prod-ns` namespace'i altında  `petclinic-cluster` isimli Kubernetes cluster'ındaki prod-environment'a deploy edilecek

```yml
- job name: petclinic-prod
- job type: pipeline
- Source Code Management: Git
      Repository URL: https://github.com/alparslanu6347/petclinic-microservices-with-db-de.git
- Branches to build:
      Branch Specifier (blank for 'any'): */main
# - Build triggers: GitHub hook trigger for GITScm polling  --->> Burası olacaksa => Repoya gidip webhook eklemek gerekir ==>> http://[jenkins-server-hostname]:8080/github-webhook/
- Pipeline:
      Script Path: jenkins/jenkinsfile-petclinic-prod
```

- Click `save` -->> Click `Build Now`

# `k8s/petclinic_hart/templates/` klasörü altındaki `api-gateway-ingress.yaml` sebebiyle AWS'de 1 adet `Classic Load Balancer` oluştu

```bash (pwd : /var/lib/jenkins   , jenkins-user)
kubectl get ns  # petclinic-prod-ns  gör
kubectl get ingress -n petclinic-prod-ns
# NAME          CLASS   HOSTS                                   ADDRESS                                                               PORTS     AGE
# api-gateway   nginx   petclinic.devopsalparslanugurer.com     ae416b46a98794305bccb01f1137519f-192718.us-east-1.elb.amazonaws.com   80        2m41s
```
# Route53  record varsa yeniden create etmene gerek yok  edit yap.
* AWS Console Route53 :  Hosted zones -->> devopsalparslanugurer.com -->>
                                  Create record -->>  Record name : petclinic
                                                      Record type : A
                                                      Alias : `AÇ`
                                                      Route traffic to: Alias to Application and Classic Load Balancer
                                                      Choose Region : US East(N. Virginia) [us-east-1]
                                                      Choose load balancer : dualstackae416b46a98794305bccb01f1137519f-192718us-east-1elbamazonawscom
                                                  Create records : ENTER

* Browser'da `petclinic.devopsalparslanugurer.com` yaz ve uygulamayı gör.
* Veya  `curl http://petclinic.devopsalparslanugurer.com` komutu ile sayfanın geldiğini kontrol et.

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 29 - Setting Domain Name and TLS for Production Pipeline with Route 53 ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

```bash (pwd : petclinic-microservices-with-db)
# Create `feature/msp-29` branch from `main`
git checkout main
git branch feature/msp-29
git checkout feature/msp-29
```

# How Does A TLS Certificate Work? 

- browser'da ``rancher``ın   adresinin yazılı olduğu yerdeki kilide yıkla -->> Connection is secure -->> Certificate is valid --TIKLA-->> İNCELE
- browser'da ``shopify.com``  adresinin yazılı olduğu yerdeki kilide yıkla -->> Connection is secure -->> Certificate is valid --TIKLA-->> İNCELE

* We request a CSR (Certificate Signing Request) from any CA (Certificate Authority).
* CA validates our request.
* CA sends signed certificate to us.

# Kubernetes'de bu işlemler nasıl gerçekleşiyor?

- <https://cert-manager.io/docs/>
* Sertifika almak için `letsencrypt-prod`u kullanacağız.
* <https://cert-manager.io/docs/installation/helm/>

* petclinic cluster'a `cert-manager`ı install edelim:

```bash (pwd : /var/lib/jenkins   , jenkins-user)
# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Add the Jetstack Helm repository.
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository.
helm repo update

# SERTİFİKA ALIRKEN AYRI object'LERE İLAVE resource'LARA İHTİYACIMIZ VAR. AŞAĞIDAKİ KOMUTU GİRİNCE SERTİFİKA İŞLEMLERİ İÇİN YANİ CERT MANAGER'IN İHTİYAÇ DUYDUĞU object'LERİ YÜKLEYECEĞİZ
# Install the `Custom Resource Definition` resources separately
# https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml
```

```bash (pwd : /var/lib/jenkins   , jenkins-user)
# Install the cert-manager Helm chart
helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--version v1.12.0
```

```bash (pwd : /var/lib/jenkins   , jenkins-user)
# Verify that the cert-manager is deployed correctly.
kubectl get pods --namespace cert-manager -o wide
```

* `ClusterIssuer` object'i oluşturacağız ve onunla sertifika alacağız:
* `Let's Encrypt ACME` (Automated Certificate Management Environment) üzerinden production certificate almak için `tls-cluster-issuer-prod.yml` isimli dosya ile `ClusterIssuer` object'i oluştur.
* Aynı zamanda bu dosyayı `k8s` klasörü altına da kaydedelim.

```yaml (tls-cluster-issuer-prod.yml)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: alparslanu6347@gmail.com       # DEĞİŞTİRMEYİ UNUTMA
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01: # certificate Authority beni validate ederken, yapmamı istediği bir takım task/challenge olur, bunun için 2 yöntem var "http01 ve dns01". Biz burada "http01" yöntemini kullanacağız.
        ingress:  # "http01" yöntemiyle doğrulamayı ne üzerinden yapacağız -->> "ingress" object'i üzerinden yapacağız
          class: nginx  # "http01" yöntemiyle üzerinden doğrulama yapacağım "ingress" object'inin class'ı ne "nginx"
```

```bash (pwd : /var/lib/jenkins   , jenkins-user)
vi tls-cluster-issuer-prod.yml

kubectl apply -f tls-cluster-issuer-prod.yml
kubectl get clusterissuers letsencrypt-prod -n cert-manager -o wide   # Check if `ClusterIssuer` resource is created.
```

* Rancher Arayüz aç : 4 çizgili'ye tıkla -- Cluster Management -- Import Existing -- Generic -->>
                                                                              - Cluster Name : petclinic-eks (Rancher bu cluster'ı hangi isimle bilsin)
                                                                              - Cluster Description : petclinic-eks
                                                                        Create

# Create deyince Rancher arayüz ekranında kubectl komutları gözükür, bu komutu cluster'ında girersen rancher'ın ilgili agent'ı oluşacak o da buraya bağlayacak

```bash (pwd : /var/lib/jenkins   , jenkins-user)
kubectl apply -f https://rancher.devopsalparslanugurer.com/v3/import/phk8hnsz7dsqmnssnzn7bcgjj7pdmp42kspr6txwffcjvjvkx6p276h_c-m-6wlkqxjv.yaml
```

- Tekrar Rancher Arayüz aç : 4 çizgili'ye tıkla - petclinic-eks

```bash (pwd : petclinic-microservices-with-db)
# Commit the change, then push the tls script to the remote repo.
git add .
git commit -m 'added tls scripts for petclinic-production'
git push --set-upstream origin feature/msp-29
git checkout main
git merge feature/msp-2p
git push origin main
```

# <https://letsencrypt.org/how-it-works/>

### Rancher üzerinden sertifika alma

* Rancher Arayüz aç : 4 çizgili'ye tıkla -- petclinic-eks -- Service Discovery -->> Ingresses :
                                                                      api-gateway --> nginx --->> Yan tarafta 3 nokta üst üste tıkla : Edit YAML
                                                                      YAML dosyasında değişiklik yap : - spec altına ekleme yapacağız
                                                                                                       - annotation ekleyeceğiz
                                                                  Save.

```yaml
spec: # spec altına ekleme yapacağız , rules ile aynı hizada olacak.
  tls:
  - hosts:
    - petclinic.devopsalparslanugurer.com # DEĞİŞTİRMEYİ UNUTMA
    secretName: petclinic-tls
```

```yaml
metadata:
  name: api-gateway
  annotations:    # annotation ekleyeceğiz
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

* Browser'da açık olan uygulamayı refresh et -->> https   oldu.

### Rancher olmadan Kubernetes'de sertifika alma

```bash (pwd : /var/lib/jenkins   , jenkins-user)
kubectl get ingress -n petclinic-prod-ns
kubectl edit ingress -n petclinic-prod-ns   # yukarıdaki eklemeleri/değişimleri yapacağım
```

# # # # # # # # # # # # # # # # # # # # # # # # # # # #
*** MSP 30 - Monitoring with Prometheus and Grafana ***

# # # # # # # # # # # # # # # # # # # # # # # # # # # #

### Rancher üzerinden Prometheus ve Grafana

* Rancher Arayüz aç : 4 çizgili'ye tıkla -- petclinic-eks -- Service Discovery -->>
                                                                `prometheus-server` --> Yan tarafta 3 nokta üst üste tıkla : Edit YAML
                                                                YAML dosyasında değişiklik/ekleme yap : - port: 9090   (Değişiklik)
                                                                                                        - nodePort: 30002 (Ekleme)
                                                                                                        - type : NodePort (Değişiklik)
                                                                Save.

                                                                `grafana-server` --> Yan tarafta 3 nokta üst üste tıkla : Edit YAML
                                                                YAML dosyasında değişiklik/ekleme yap : - nodePort: 30003 (Ekleme)
                                                                                                        - type : NodePort (Değişiklik)
                                                                Save.                                                                   

### Rancher olmadan Kubernetes'den Prometheus ve Grafana

```bash (pwd : /var/lib/jenkins   , jenkins-user)
kubectl get svc -n petclinic-prod-ns
kubectl edit svc prometheus-server -n petclinic-prod-ns   # yukarıdaki eklemeleri/değişimleri yapacağım
kubectl edit svc grafana-server -n petclinic-prod-ns      # yukarıdaki eklemeleri/değişimleri yapacağım
```

# AWS Console'a git EKS Cluster'da oluşan Node'lardan birisini aç ve security group'un Inbound'una `30002 and 30003  anywhere` ekle.

-  AWS Console'a git EKS Cluster'da oluşan Node'lardan birisinin Public IP al:
                                                                              - browser'da http://2.34.124.152:30002
                                                                              - browser'da http://2.34.124.152:30003

# `petclinic-microservices-with-db/docker/prometheus` klasörü altındaki `prometheus.yml` dosyası içinde `metrics_path: /actuator-prometheus` yazıyor ===>> uygulamanın olduğu browser'a gel ve adres sonuna ekle -->> https://petclinic.devopsalparslanugurer.com/actuator-prometheus


```bash
# Delete  EKS cluster via `eksctl`. It will take a while.
eksctl delete cluster -f cluster.yaml
```
