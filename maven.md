# Hands-on Maven-01 : Using Maven As a Build Tool

Purpose of the this hands-on training is to teach the students how to use Maven with Java as a build tool.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- install Maven and Java-11 on Amazon Linux 2 EC2 instance

- explain various build phases of a Java Application

- use Maven's clean, compile, package, install and site commands


## Outline

- Part 1 - Launch Amazon Linux 2 EC2 Instance with Cloudformation Template

- Part 2 - Generate a Java application using Maven's Archetype Plugin

- Part 3 - Run Maven Commands


## Part 1 - Launch Amazon Linux 2 EC2 Instance and Connect with SSH

- Launch an EC2 instance using ```maven-java-template.yml``` file located in this folder.
    - This template will create an EC2 instance with Java-11 and Maven.

- Connect to your instance with SSH.

- Check if you can see the Maven's binary directory under ```/home/ec2-user/```.

- Run the command below to check if Java is available.

```bash
java -version
```

- Cat ```~/.bash_profile``` file to check if Maven's path is correctly transferred. If not paste the necessary lines manually.

```
# Append the lines below into ~/.bash_profile file by making necessary changes.
M2_HOME=/home/ec2-user/<apache-maven-directory-with-its-version>
export PATH=$PATH:$M2_HOME/bin
```

- Run the command below to check if mvn commands are available.

```bash
mvn --version
```

- If not, run the command below and wait for the EC2 machine to get ready.

```bash
sudo reboot
```

## Part 2 - Generate a Java application using Maven's Archetype Plugin
# https://maven.apache.org/
# https://maven.apache.org/guides/getting-started/index.html#
# https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html



- Run the command below to produce an outline of a Java project with Maven.
# https://maven.apache.org/archetype/index.html

```bash
mvn archetype:generate -DgroupId=com.clarus.maven -DartifactId=maven-experiment -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

- Cd into the project folder.

- Run the command below to be able to use tree command.

```bash
sudo dnf install -y tree
```

- Run the command below to show the directory structure of the project.

```bash
tree

```

- Go into the folder where App.java resides and ```cat``` the auto generated ```hello world``` application in Java.

- Replace the content of the App.java file with the content of the App.java file in this repo.

- Go into the project's root folder.

- Replace the content of the ```pom.xml``` file with the content of the pom.xml file in this repo.

- Since we've install Java-11 on the EC2 machine, uncomment the ```properties``` tag of the new pom.xml file.

- Explain that the ```maven.compiler.source``` property specifies the version of source code accepted and the ```maven.compiler.target``` generates class files compatible with the specified version of JVM.

- Explain that ```dependencyManagement``` section in the pom file will import multiple dependencies with compatible versions.  


## Part 3 - Run Maven Commands


>### mvn compile
 
- Run the command below.

```bash (pwd : /home/ec2-user/maven-experiment)
mvn compile
```

- Go into the folder ```<project-root>/target/classes/``` and show the class file.

- Run the command below to show how to test a Maven project.


>### mvn clean test

```bash (pwd : /home/ec2-user/maven-experiment)
mvn clean test
mvn help:effective-pom
```

- Show that there is a new folder named ```target``` in the project root. 

- inspect the target folder with tree command.

- Show the content of the file ```<project-root>/target/surefire-report/com.clarus.maven.AppTest.txt``` as the output of the test.


>### mvn package

- Run the command below.

```bash
mvn clean package
```

- Go into the folder ```<project-root>/target/``` and show the ```maven-experiment-1.0-SNAPSHOT.jar``` file as the output of the ```mvn package``` command.

- Run the command below to start the application.

```bash
java -jar maven-experiment-1.0-SNAPSHOT.jar
```

- Explain the error in the standard output. 
    - Maven's jar file is not an executable jar file. The jar file does not have both the ```Main Class``` and the necessary packages to run the application. 

- Add the plugin below to the pom file and run ```mvn clean package``` command again.

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <executions>
          <execution>
              <phase>package</phase>
              <goals>
                  <goal>single</goal>
              </goals>
              <configuration>
                  <archive>
                  <manifest>
                      <mainClass>
                          com.clarus.maven.App
                      </mainClass>
                  </manifest>
                  </archive>
                  <descriptorRefs>
                      <descriptorRef>jar-with-dependencies</descriptorRef>
                  </descriptorRefs>
              </configuration>
          </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

```bash
mvn clean package
```


>### Run The Application

- Now, open up a fresh terminal on your local computer and run the command below.

```bash
scp -i <path-to-your-pem-file> -r <path-to-your-home-directory>/.aws ec2-user@<IP-of-your-instance>:/home/ec2-user/
```

- Check if the the credentials are transferred to EC2 instance.

- Go into your target folder.

- Run the command below to start the application. This time we are running the executable jar file with suffix ```jar-with-dependencies```.

```bash
java -jar maven-experiment-1.0-SNAPSHOT-jar-with-dependencies.jar
```

- Explain what the application does in the background.
    - Note that to be able see the object and the S3 bucket, we should comment the lines 142 and 150.


>### mvn install

- Run the command below to install our own package into .m2 folder.

```bash
mvn install
```

- Go into ```~/.m2/repository``` folder and show where our package is installed.


>### mvn site

- Add two more plugins to run the command ```mvn site```

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-site-plugin</artifactId>
    <version>3.7.1</version>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-project-info-reports-plugin</artifactId>
    <version>3.0.0</version>
</plugin>
```

- Run the command below.

```bash
    mvn clean site
```

- Show the output ```site``` directory under target directory.

- Run the command below to install Apache Server.

```bash (pwd : /home/ec2-user/maven-experiment/target/site)
python3 -m http.server
```
# veya

```bash (pwd : /home/ec2-user/maven-experiment/target)
sudo dnf install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

- Run the command below to copy the contents of the site folder under ```/var/www/html``` folder.

```bash (pwd : /home/ec2-user/maven-experiment/target)
sudo cp -a site/. /var/www/html
```



------------------------------------------------------------------------------------------

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

# pwd : /home/ec2-user/maven-experiment ==>> pom.xml dosyamın da içerisinde olduğu, `mvn clean package` komutunu çalıştırdığım dizin
# `pwd` : /home/ec2-user/maven-experiment   klasörünü volume olarak bağlayacak
# container içerisinde /app diye folder'a bağlanacak, böyle bir folder olmasa da oluşacak

# -w : working directory , docker run ile beraber komut yazabilirim, komutun çalışcağı directory'i gösteriyorum
# -w /app : komut oluşan /app dizini içerisinde çalışacak
# mvn clean package : /app dizini içerisinde çalışacak komut, /app dizinine `pwd` bağlanmıştı, yani /home/ec2-user/maven-experiment dizini bağlanmıştı,
# maven:3.6-openjdk-11 : image ismi 

docker run --rm -v $HOME/.m2:/root/.m2 -v `pwd`:/app -w /app maven:3.6-openjdk-11 mvn clean package

```



