# terraform-docker-ecr-ecs

## Create Elastic Container Registry
```
cd terraform-ecr
terraform init
terraform apply
# Note the <ecr-URL> returned in output
```

## Create Docker image of Node app and push to above ECR
```
cd docker-node-app
docker build -t <ecr-URL>/myapp:1 .

# aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ecr-URL> 

docker push <ecr-URL>/myapp:1
```

## Create Elastic Container Service to run image in above registry behind an ELB
```
cd terraform-ecs
cp ../terraform-ecr/terraform.tfstate .
terraform init
terraform apply 
# Note <elb-URL> returned in output
```
## Test Node app running on ECS

```
ssh -i mykey <public-ip of ecs-ec2-container> -l ec2-user
# After connecting to container instance
sudo -s
cat /etc/ecs/ecs.config #check user-data execution
ps aux | grep agent # check ecs-agent is running
curl localhost:8000 # check if app is running directly from container
tail /var/log/ecs- # check logs if needed
exit

curl <elb-URL> #Test app from external source
```
#### Destroy all resources once done
```
terraform destroy
```