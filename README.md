# Cluster
- group of container instances acting as a single compute resource
- Task나 services를 실해하면 cluster에서 돌게끔 스케쥴된다.
- 여러 종류의 instances type을 조합할수있다. 다를 availabilty zone도 사용할수있다.
- 하나의 instance를 여러 클러스터에서 사용할순없다.


```bash
aws ecs create-cluster --cluster-name muchacho
aws ecs list-clusters
aws ecs describe-clusters --cluster muchacho
aws ecs delete-cluster --cluster-name muchacho
```

# Container Agent
- ec2 instance를 cluster에 join 해주는 해주는 tool이다.
- amazon-ecs-agent docker도있으니까 설치하기 쉽다.
- env 나 config file로 설정할수있다.


us-east-1빼고 bucket만들때 specify해야한다.
```bash
aws s3api create-bucket --bucket oiojin831-muchacho --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1
```

ecs.config에 우리는 default cluster를 사용하지 않기 때문에 container agent가 어떤 cluster에 연결할지 알게 하기위해 ECS_CLUSTER 파라미터를 설정해준다.

bucket에 config파일을 업로드한다.
aws s3 cp ecs.config s3://oiojin831-muchacho/ecs.config

