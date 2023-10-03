* Prepare server manually on Amazon Linux 2023 (t2.micro) , enabled with `Docker`,  `Docker-Compose`,  `Java 11`,  `Git`.

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
docker compose --version       # v2.17.3
sudo dnf install git -y
git -v                         # git version 2.40.1
sudo dnf install java-11-amazon-corretto -y
java -version
newgrp docker
```

* .bashrc içerisine

```bash
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "  # sarı-mavi-sarı
sudo hostnamectl set-hostname "docker-study"
```

*** COPY ve ADD KULLANIMI ***

- `COPY`: Lokal dosyaları veya dizinleri Docker container'ına kopyalamak için kullanılır.

Kullanım Örnekleri:

a. COPY app.js /app                 # app.js dosyasını yerel makineden /app dizinine kopyalar.
b. COPY . /app                      # Dockerfile'ın bulunduğu dizindeki tüm dosyaları /app dizinine kopyalar.
c. COPY file.txt /app/file.txt      # file.txt dosyasını yerel makineden /app dizinindeki file.txt olarak kopyalar.

- `ADD`: Lokal dosyaları, dizinleri veya `URL`'leri Docker container'ına kopyalamak için kullanılır. Ayrıca, `dosyaları otomatik olarak açabilir` ve `tar arşivlerini de çözebilir`.

Kullanım Örnekleri:

a. ADD app.js /app                              # app.js dosyasını yerel makineden /app dizinine kopyalar.
b. ADD http://example.com/file.txt /file.txt    # Bir URL'den file.txt dosyasını kopyalar.
c. ADD . /app                                   # Dockerfile'ın bulunduğu dizindeki tüm dosyaları /app dizinine kopyalar.

- `COPY` ve `ADD` arasındaki `fark`:

    - ADD komutu COPY ile aynı işi yaparken ek özelliklere sahiptir : URL'lerden dosyaları indirebilir, otomatik olarak dosyaları açabilir ve tar arşivlerini çözebilir.
    - Genel olarak, sadece yerel dosyaları kopyalamak istiyorsanız COPY komutunu tercih etmek daha iyi olabilir.
    - ADD komutunun ek özelliklerine ihtiyaç duyuyorsanız veya Dockerfile'ınızı daha esnek hale getirmek isterseniz ADD komutunu kullanabilirsiniz.



# Hands-on Docker-10 : `exec` form and `shell` form, `CMD` and `ENTRYPOINT` instructions, and `ARG` instruction

Purpose of the this hands-on training is to give the students understanding to difference between:

- `exec` form and `shell` form  

- `CMD` instruction and `ENTRYPOINT` instruction 

- `ARG` instruction

## Learning Outcomes

- At the end of the this hands-on training, students will be able to;

- Learn `exec` form and `shell` form. 

- Explain difference between `exec` form and `shell` form.

- Learn `CMD` instruction and `ENTRYPOINT` instruction.

- Explain difference between `CMD` instruction and `ENTRYPOINT` instruction.

- Use `CMD` instruction as a parameter of `ENTRYPOINT` instruction

- Use `ARG` Instruction

## Outline

- Part 1 - Difference between `exec` form and `shell` form

- Part 2 - Difference between `CMD` instruction and `ENTRYPOINT` instruction

- Part 3 - Use `CMD` instruction as a parameter of `ENTRYPOINT` instruction

- Part 4 - `ARG` instruction

## Part 1 - Difference between `exec` form and `shell` form

- Create a folder and name it shell-exec-cmd-entrypoint.

```bash
mkdir shell-exec-cmd-entrypoint
cd shell-exec-cmd-entrypoint
```

- create a Dockerfile and input following statements.

```Dockerfile
FROM ubuntu
CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "cmd-shell:v1".

```bash
docker build -t cmd-shell:v1 .
```

- Run cmd-shell image.

```bash
docker run cmd-shell:v1     # hello
```

- Change the CMD line to exec form. For exec form, we change to shell script to JSON array form.

```Dockerfile
FROM ubuntu
CMD ["echo", "hello"]
# CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "cmd-exec:v1".

```bash
docker build -t cmd-exec:v1 .
```

- Run cmd-exec image.

```bash
docker run cmd-exec:v1  # hello
```

- It seems the same. What is the difference?

- Change the Dockerfile as below. This time, we input an environment variable instead of hello. 

```Dockerfile
FROM ubuntu
CMD echo $HOME

# CMD ["echo", "hello"]
# CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "cmd-shell:v2".

```bash
docker build -t cmd-shell:v2 .
```

- Run cmd-shell:v2 image.

```bash
docker run cmd-shell:v2     # /root
```

- Change the CMD line to exec form in Dockerfile.

```Dockerfile
FROM ubuntu
CMD ["echo", "$HOME"]

# CMD echo $HOME

# CMD ["echo", "hello"]
# CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "cmd-exec:v2".

```bash
docker build -t cmd-exec:v2 .
```

- Run cmd-exec:v2 image.

```bash
docker run cmd-exec:v2      # $HOME
```

- This time, the output is changed. We couldn't learn `$HOME` environment variable. The reason is that: <br>
Unlike the shell form, the exec form does not invoke a command shell. This means that normal shell processing does not happen. So we couldn't reach environment variables.

## Part 2 - Difference between `CMD` instruction and `ENTRYPOINT` instruction

- First, run the `cmd-shell:v1` image with ls command.

```Dockerfile   (BU image İÇİN KULLANDIĞIM Dockerfile)
FROM ubuntu
CMD echo "hello"
```

```bash
docker run cmd-shell:v1 ls
```

- Notice that, we see list of `root(/) directory` instead of hello.

- Change the CMD line to ENTRYPOINT line in Dockerfile as below.

```Dockerfile
FROM ubuntu
ENTRYPOINT echo hello

# CMD ["echo", "$HOME"]
# CMD echo $HOME

# CMD ["echo", "hello"]
# CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "entrypoint:v1".

```bash
docker build -t entrypoint:v1 .
```

- Run `entrypoint:v1` image with ls command.

```bash
docker run entrypoint:v1 ls     # hello
```

- Notice that we couldn't execute ls command. Because, unlike `CMD`, we can't override `ENTRYPOINT`.

## Part 3 - Use `CMD` instruction as a parameter of `ENTRYPOINT` instruction

- Change the `ENTRYPOINT` line to exec form and add a `CMD` line.

```Dockerfile
FROM ubuntu
ENTRYPOINT [ "echo", "Hello" ]
CMD [ "Arrow" ]

#ENTRYPOINT echo hello

# CMD ["echo", "$HOME"]
# CMD echo $HOME

# CMD ["echo", "hello"]
# CMD echo "hello"
```

- Build an image from this Dockerfile and tag it as "entrypoint-cmd".

```bash
docker build -t entrypoint-cmd .
```

- Run the `entrypoint-cmd` image.

```bash
docker run entrypoint-cmd   # Hello Arrow
```

- Now, run the `entrypoint-cmd` image as Below.

```bash
docker run entrypoint-cmd Levent    # Hello Levent

docker rm -f $(docker ps -aq)   # Çalışan/Çalışmayan tüm containerları siler
docker image prune -af  # tum imajlari silmek icin
```
- Notice that we can override `CMD` instruction but we can not override `ENTRYPOINT` instruction.


## Part 4 - `ARG` instruction

- The ARG instruction defines a variable that users can pass at `build-time` to the builder with the docker build command using the `--build-arg <varname>=<value>` flag. If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning. 

- An ARG instruction can optionally include a default value. If an ARG instruction has a default value and if there is no value passed at build-time, the builder uses the default.

- Create a folder and name it arg-instruction.

```bash
mkdir arg-instruction
cd arg-instruction
```

- create a `Dockerfile` and input the following statements.

```Dockerfile
FROM alpine
ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-1/main/version.tar /version.tar
RUN tar -xvf version.tar
CMD cat /version/version
```

- Build the docker image.

```bash
docker build -t argtest1 .
```

- Run the container

```bash
docker run argtest1
```

- We get an output like this.

```bash
this is version 1
```

- Let's assume that the application is updated and we get the latest version (version-2) as below. Update the `Dockerfile`.

```Dockerfile
FROM alpine
ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-2/main/version.tar /version.tar
RUN tar -xvf version.tar
CMD cat /version/version
```

- Build and run the image again.

```bash
docker build -t argtest2 .
docker run argtest2
```

- This time the output will be version 2.

```bash
this is version 2
```

- Updating the version with this method is not practical. For this, we will use `ARG` instruction. With `ARG` instructions we will update our image at `build-time`. Update the `Dockerfile` as below.

```Dockerfile
FROM alpine
ARG VERSION=1
ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-${VERSION}/main/version.tar /version.tar
RUN tar -xvf version.tar
CMD cat /version/version
```

- Build and run the image again.

```bash
docker build -t argtest3 .
docker run argtest3
```

- This time the output will be version 1 again.

```bash
this is version 1
```

- Finally, we don't update the Dockerfile but we update the image with `--build-arg <varname>=<value>` flag.

```bash
docker build --build-arg VERSION=2  -t argtest .
```

- Run the container and see that the output will be version 2.

```bash
docker run  argtest4 
```

- The output:

```bash
this is version 2
```