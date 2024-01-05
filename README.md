# Spark submit with K8s cluster

**Prerequisite**: Install local K8s cluster with minikube, Docker Desktop, Orbstack, Rancher Desktop ...

### Step 1: Get the cluster IP

```bash
kubectl cluster-info
```

### Step 2: Create K8s service account

```bash
kubectl create serviceaccount spark

kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
```

### Step 3: Submit job

Sample spark-submit with local file

```bash
spark-submit \
    --master k8s://https://127.0.0.1:32769 \
    --deploy-mode cluster \
    --name spark-pi \
    --executor-memory 1G \
    --num-executors 5 \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=apache/spark \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
    --conf spark.kubernetes.file.upload.path='' \
    local:///opt/spark/examples/jars/spark-examples_2.12-3.5.0.jar 100000
```

Spark-submit with S3

```bash
spark-submit \
    --master k8s://https://127.0.0.1:32769 \
    --deploy-mode cluster \
    --name spark-pi \
    --executor-memory 1G \
    --num-executors 5 \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=apache/spark \
    --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
    --packages org.apache.hadoop:hadoop-aws:3.2.2 \
    --conf spark.kubernetes.file.upload.path=s3a://${BUCKET}/${KEY} \
    --conf spark.hadoop.fs.s3a.access.key=<AWS ACCESS KEY> \
    --conf spark.hadoop.fs.s3a.secret.key=<AWS SECRET KEY> \
    --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
    --conf spark.hadoop.fs.s3a.aws.credentials.provider=org.apache.hadoop.fs.s3a.SimpleAWSCredentialsProvider \
    --conf spark.hadoop.fs.s3a.fast.upload=true \
     --conf spark.driver.extraJavaOptions="-Divy.cache.dir=/tmp -Divy.home=/tmp" \
    file:///path/to/pi.py 10
```
