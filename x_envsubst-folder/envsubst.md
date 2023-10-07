### Explanation of envsubst

* In jenkinsfile we will use the command below

```bash
sh "envsubst < k8s/petclinic_chart/values-template.yaml > k8s/petclinic_chart/values.yaml"
```

* In Jenkinsfile         ==  In this example 
  - sh                   ==  env-var.sh  (script name)
  - values-template.yaml ==  hello-template.yaml
  - values.yaml          ==  hello.yaml

* Create your script `env-var.sh`  in order to convert values-template.yaml -->> values.yaml
                                                            (hello-template.yaml -->> hello.yaml)
```bash (pwd : test)
export NAME="Arrow-Mentor"
envsubst <hello-template.yaml> ./hello.yaml
```

* Change file permissions of `env-var.sh` , make it executable.

```bash (pwd : test)
chmod +x env-var.sh
```

* Create your `hello-template.yaml`

```bash (pwd : test)
Hello ${NAME}
```

* Finally execute your script `env-var.sh` in the same directory. When you execute the command `hello.yaml` will be created in the same directory.

```bash (pwd : test)
./env-var.sh
ls      # You can see -->>  env-var.sh  hello.yaml  hello-template.yaml
```