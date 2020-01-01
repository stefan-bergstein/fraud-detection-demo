# fraud-detection-demo
Open Data Hub demo with the fraud detection use case.

*The repository is under construction!*

This repo is based on the [Open Data Hub Fraud Detection Use Case Tutorial] (https://gitlab.com/opendatahub/fraud-detection-tutorial/).
The data originates from [Kaggle/ULB](https://www.kaggle.com/mlg-ulb/creditcardfraud).

## File and directories

- Jupyter notebook [rauddetection-demo-notebook.ipynb](rauddetection-demo-notebook.ipynb)



# Ceph Nano setup
The training data and model data in this scenario is stored in S3. Here we are using [Ceph Nano](https://github.com/ceph/cn/).


## Install CN
Check that Docker is working. E.g with `docker run hello-world`

```
mkdir cn && cd cn
curl -L https://github.com/ceph/cn/releases/download/v2.3.1/cn-v2.3.1-linux-amd64 -o cn && chmod +x cn
```

## Create a Ceph Cluster and few buckets.
Ensure you have enough disk space in `/data` or use a different directory 

### Create small helper script:

```
cat > setup-cn-odh.sh <<EOF
#!/bin/bash

#CLUSTERS="fraud-demo ai-library test"
CLUSTERS="test"

DATADIR="/data/odh"

for c in $CLUSTERS
do
  echo $c
  mkdir -p $DATADIR/$c
  ./cn cluster start -d $DATADIR/$c "odh-$c-cluster" -s 20GB -f default 
  docker update --restart=always "ceph-nano-odh-$c-cluster"
done
EOF
```

### Create a cluster
```
./setup-cn-odh.sh 
test
2019/12/31 14:07:22 Running cluster odh-test-cluster | image ceph/daemon | flavor default {512MB Memory, 1 CPU} ...

Endpoint: http://<your-cn-ip>:8000
Dashboard: http://<your-cn-ip>:5000
Access key: <your-access-key>
Secret key: <your-secret-key>
Working directory: /data/odh/test
```

### Create a few buckets

```
./cn s3 mb odh-test-cluster fraud-demo
"Bucket 's3://fraud-demo/' created on cluster odh-test-cluster

./cn s3 mb odh-test-cluster ai-library
"Bucket 's3://ai-library/' created on cluster odh-test-cluster

./cn s3 mb odh-test-cluster test
Bucket 's3://test/' created on cluster odh-test-cluster
```

### Test remote access to S3 CN from a different system 
Either install the aws cli or skip this test

```
echo "hi Ceph Nano" > /tmp/hi-cn.txt
aws s3 cp  /tmp/hi-cn.txt s3://test/hi-cn.txt --endpoint-url http://your-cn-ip:8000 --no-verify-ssl
upload: ../../../../../tmp/hi-cn.txt to s3://test/hi-cn.txt     
aws s3 ls s3://test/ --endpoint-url http://your-cn-ip:8000 --no-verify-ssl
2019-12-31 14:31:38         13 hi-cn.txt
```

## Upload fraud-detection data

For example on the CN System:

Download data into tmp directory:

```
mkdir /data/tmp/
curl https://gitlab.com/opendatahub/fraud-detection-tutorial/raw/master/data/creditcard.csv -o /data/tmp/creditcard.csv 
curl https://gitlab.com/opendatahub/fraud-detection-tutorial/raw/master/data/creditcard-sample10k.csv -o /data/tmp/creditcard-sample10k.csv 
```

Put the files data:
```
./cn s3 put odh-test-cluster /data/tmp/creditcard.csv fraud-demo/creditcard.csv
./cn s3 put odh-test-cluster /data/tmp/creditcard-sample10k.csv fraud-demo/creditcard-sample10k.csv

./cn s3 ls odh-test-cluster fraud-demo
2019-12-31 13:49   5467621   s3://fraud-demo/creditcard-sample10k.csv
2019-12-31 13:48 150259138   s3://fraud-demo/creditcard.csv
```


In case you like to copy the data from another system using the aws cli:

```
aws s3 cp  /tmp/creditcard.csv s3://fraud-demo/creditcard.csv --endpoint-url http://<your-cn-ip>:8000 --no-verify-ssl
aws s3 cp  /tmp/creditcard-sample10k.csv s3://fraud-demo/creditcard-sample10k.csv --endpoint-url http://<your-cn-ip>:8000 --no-verify-ssl
```

# Deploy Open Data Hub
to-be-described

# Deploy the model using Seldon
to-be-described

# Demo flow
to-be-described

