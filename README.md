## key-pair
'KeyMaterial'는 pem으로 안전한 키를 만들게 하는 거임
```bash
mkdir ~/.ssh
aws ec2 create-key-pair --key-name aws-oiojin831 --query 'KeyMaterial'  --output text > ~/.ssh/aws-oiojin831.pem
chmod 400 ~/.ssh/aws-oiojin831.pem
aws ec2 describe-key-pairs --key-name oiojin831_free_micro
```

## security-group
aws ec2 create-security-group --group-name oiojin831_dev_tokyo --description "Security group for oiojin831 on tokyo"
aws ec2 describe-security-groups --group-id sg-17e0fd73
aws ec2 authorize-security-group-ingress --group-id sg-17e0fd73 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-17e0fd73 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-17e0fd73 --protocol tcp --port 5432 --cidr 0.0.0.0/0 --source-group sg-17e0fd73
aws ec2 authorize-security-group-ingress --group-id sg-17e0fd73 --protocol tcp --port 3679 --cidr 0.0.0.0/0 --source-group sg-17e0fd73

## create iam role
do it in web console
ecs ec2 s3 연결해주는 role을 이름을 지어서 만들어준다.


# Amazon ECS
- Cluster - group of instances
- Container Instances - instances that are part of Cluster
- Container Agent - tool that is taking care of instance to register cluster
- Task definitions - how your docker image should be run, docker-compose setreoid
- Scheduler - service나 one-off task가 cluster안에 어떤 isntance 에서 돌아야지 정해주는거
- Services - long runngng task - web 서버같은거, task definition을 base로 만들어짐
- Tasks - 이것도 task definition에서 만들어지는 단발성 임무, batch job같은거 끝나면 다시 시작하지 않는 그런거

## Cluster
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

bucket을 만들어서 나중에 instance만들때 실행할 파일을 저장한다.
us-east-1빼고 bucket만들때 specify해야한다.
```bash
aws s3api create-bucket --bucket oiojin831-muchacho --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1
```
ecs.config에 우리는 default cluster를 사용하지 않기 때문에 container agent가 어떤 cluster에 연결할지 알게 하기위해 ECS_CLUSTER 파라미터를 설정해준다.
```bash
aws s3 cp ecs.config s3://oiojin831-muchacho/ecs.config
```
bucket에 config파일을 업로드한다.
나중에 ec2 instance를 생성할때 자동으로 실행 시킬 shell
copy-ecs-config-to-s3 를 생성한다.

## Container Instances
container instance는 cluster에 등록된 amazon ec2 instance 이다.
container agent를 통해 cluster에 connect한다.

instance를 하나만들어주는데 ami가 tokyo의 ecs용으로 만든다. iam profile은 이름으로 위에서 만든 이름을 지정해주고 key를 지정하고 처음에 실행될 파일을 지정해준다.
```bash
aws ec2 run-instances --image-id ami-096cba68 --count 1 --instance-type t2.micro --iam-instance-profile Name=ecsInstanceRole --key-name dev-machine --security-group-ids sg-17e0fd73 --user-data file://copy-ecs-config-to-s3
aws ec2 describe-instance-status --instance-id i-0c482d784057d2d5a
aws ecs list-container-instances --cluster muchacho
aws ecs describe-container-instances --cluster muchacho --container-instances arn:aws:ecs:ap-northeast-1:303133924573:container-instance/89011c88-ac8d-42fc-a734-9683282d7222
aws ec2 terminate-instances --instance-ids i-0c482d784057d2d5a
```



