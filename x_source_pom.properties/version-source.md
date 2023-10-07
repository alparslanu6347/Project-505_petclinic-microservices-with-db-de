### Explanation of "source & pom.properties & echo $version"

* In jenkinsfile we will use the command below

```bash
MVN_VERSION=$(. ${WORKSPACE}/spring-petclinic-admin-server/target/maven-archiver/pom.properties && echo $version)
```

* In Jenkinsfile           ==  In this example 
  - `pom.properties`       ==  `test.txt`


* After `docker run --rm -v $HOME/.m2:/root/.m2 -v $WORKSPACE:/app -w /app maven:3.6-openjdk-11 mvn clean package ` command our jar package will be created, in the same time our  `target` directory will be created. 

* In the target directories there are `pom.properties` files, normally before every push we change the version of the code in this file and after every push we have new version number. I need to get this new version number dynamically in order to define `MVN_VERSION` variable , and I will use this variable to add tag on my image. So we can now push our image with new version number dynamically got.

* Create `test.txt` file in the test directory

```text
groupId=org.springframework.samples.petclinic.admin
artifactId=spring-petclinic-admin-server
version=2.1.2
```

* Execute and see the version number.

```bash (pwd : test)
source test.txt
# . test.txt (or you can use this command)

echo $version   #  now you can see "2.1.2"
```

NOTES:
- source test.txt = . test.txt = . ./test.txt = source ./test.txt = chmod +x test.txt && ./test.txt
