# Deploying Gaffer on AWS EKS

First follow the [instructions here](../../aws-eks-deployment.md) to provision and configure an [AWS EKS](https://aws.amazon.com/eks/) cluster that the Gaffer Helm Chart can be deployed on.

If you are hosting the container images in your AWS account, using ECR, then run the following commands to configure the Helm Chart to use them:

```bash
ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
[ "${REGION}" = "" ] && REGION=$(aws configure get region)
[ "${REGION}" = "" ] && REGION=$(curl --silent -m 5 http://169.254.169.254/latest/dynamic/instance-identity/document | grep region | cut -d'"' -f 4)
if [ "${REGION}" = "" ]; then
  echo "Unable to detect AWS region, please set \$REGION"
else
  REPO_PREFIX="${ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/gchq"

  EXTRA_HELM_ARGS=""
  EXTRA_HELM_ARGS+="--set hdfs.namenode.repository=${REPO_PREFIX}/hdfs "
  EXTRA_HELM_ARGS+="--set hdfs.datanode.repository=${REPO_PREFIX}/hdfs "
  EXTRA_HELM_ARGS+="--set hdfs.shell.repository=${REPO_PREFIX}/hdfs "
  EXTRA_HELM_ARGS+="--set accumulo.image.repository=${REPO_PREFIX}/gaffer "
  EXTRA_HELM_ARGS+="--set api.image.repository=${REPO_PREFIX}/gaffer-rest "
fi
```

Deploy the Helm Chart with:

```bash
export HADOOP_VERSION=${HADOOP_VERSION:-3.2.1}
export GAFFER_VERSION=${GAFFER_VERSION:-1.11.0}

helm dependency update

helm install gaffer . -f ./values-eks-alb.yaml \
  ${EXTRA_HELM_ARGS} \
  --set hdfs.namenode.tag=${HADOOP_VERSION} \
  --set hdfs.datanode.tag=${HADOOP_VERSION} \
  --set hdfs.shell.tag=${HADOOP_VERSION} \
  --set accumulo.image.tag=${GAFFER_VERSION} \
  --set api.image.tag=${GAFFER_VERSION}

helm test gaffer
```
