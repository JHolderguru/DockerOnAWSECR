# Creating Docker Container on AWS
#### What we're going to do in this lab is we're going to create an EC2 instance.

We're going to install Docker on that instance where then going to download Ubuntu image and we're going

to make some modifications to it and then we're going to upload that image to the registry and then

deploy a Fargate cluster and actually use our image and deploy a task using our Custom Image.

1. ssh into the machine
2. aws config - enter access key
                      - enter Secret Key
                      - default region name
                      - : enter
3. sudo amazon-linux-extras install docker (b patient take a while)
4. sudo service docker start
5. sudo systemctl enable docker
6. sudo usermod -a -G docker ec2-user
7. sudo shutdown -r now (restart)
8. -ssh -i "jholderguru.pem" ec2-user@132.234.23.344
9. - docker info
10. touch Dockerfile(create the file)
11. edit with nano >> nano Dockerfile

      FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello World!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh

12. docker build -t  jholderguru-container .
13. docker images --filter reference=jholderguru-container #ToTag
14. docker run -t -i -p 80:80 jholderguru-container #ignoreMsg
15. what we What we should find now is if we come back to the ECS management console grab that public IP address

and put it in.

Sure enough this is our container.

So this is the message that we should see says this is a container installed from my jholderguru container

image in the jholderguru ECR repository.

16. control c #stops container running
17.Now we move on to create this in our ECR
aws ecr create-repository --repository-name jholderguru-ecr --region eu-west-1


## if it doesnt work

ssh-add -c jholderguru.pem
chmod 600 jholderguru.pem

##if it doesnt work use the following command:
               nano
                       >> (on your computer)open file and copy the .pem contents
                       >> save (as the same name from computer) and close
18. ```docker tag jholderguru-container aws_account_id.dkr.ecr.region.amazonaws.com/jholderguru-ecr```

expected out put

```
[ec2-user@ip-10-0-1-62 ~]$ aws ecr create-repository --repository-name jholderguru-ecr --region eu-west-1
{
    "repository": {
        "repositoryUri": "591609902341.dkr.ecr.eu-west-1.amazonaws.com/jholderguru-ecr",#useOnNExtCmd

        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        },
        "registryId": "591609902341",
        "imageTagMutability": "MUTABLE",
        "repositoryArn": "arn:aws:ecr:eu-west-1:591609902341:repository/jholderguru-ecr",
        "repositoryName": "jholderguru-ecr",
        "createdAt": 1601908629.0

```
docker tag jholderguru-container 591609902341.dkr.ecr.eu-west-1.amazonaws.com/jholderguru-ecr #numberisFromAbove

18.  aws ecr get-login --no-include-email --region eu-west-1

expcted output:
docker login -u AWS -p eyJwYXlsb2FkIjoiRGtGMlhtY1BNWjI2cjNpVVEwNG5SbWlUL0RFeHdCUmI3bEZLWFBVZ1pYdGVnYWsrV0JOVjdadjBqZjNYSTlKMnQ3Qlp5amZVV1Zka21pVzFJcG1CTWVJMmxsQm0xa2ZSNHFHWjJ4YW52MUdySnZqdUZFYXN5U2tNUFpaMElpMTV2Sjh4RVhISVo0dkdBVEp6UFc4OHpqd0M1cEY5TktwMFo3WkE3ZGlnVmhZeW1zZndvQ0RRVzZzR0xEY2VzWkIvUXUxamQzM01yQkhQc3RPdGxsYUQ0UDVENFo0QzdLODZqNm9PT1diZGJsSGRuWTUxOXJRaGtKV3RiOW1BQVdQd0JzbWJUU0JTTk1paExuTEVUSUhIOEhBOXZVR0VxWUVjdzYzZ1FBZ0lyeDBPMXlWWXpYSXg2WlFob1JkbnpqcEQrOGczWUEyWWpXWWhlK21sZWk2U3NVRjdxa2JvUHRiYXdvcjRRcWdjdmo0Z0FyS1FMeDczQUlnWVBFVXlLUkxhYUhqc01WcnVTamhGK3R4RkNOSFlHTmdYRHNWU2JWZHBHbE9SRUx2ZUVubHBzZnNMditvMU1DNGVuRDZDcmpadzdjODdMUGk2eWEwc01SVGxveG9MWGc2empKWG5SdFpIcjBjNXJhaG1WYWRacXpsanR0WXR6emI0b205SnJ2UkF4RUdOTW51Vyt6MWNBV2ppUnVndzhkRDh4VWVXSkZvbHdEbkFjT2dieFpweXkwVE1rK3I1SGNOalNUd1VUdndzb0lMOUUxdUgrR3JUMUcvQ2hLSis5SFV0YnBUWUkva2M4Z2dVYXlXU21mZWlOUWdUanlSSk9zY3BHVXpyQ2tZWVlWTm40dHlFYlR0T1E4Q3dUb0FnamszVXl6TUYwWVlDeklXdTdVcmxjSk1CUnprV1MzVHlVbmpLbHhLdHo3WmVwUVhQZVFiM01iNnV4RTYyNmJhb3hydGxrSGh3cG5UU0lURW5oRWw3Z0ZwMU8xUExxUmpCMjdKWStMWHZ1SFZqdUh4K0hCRS9WSXhvVExtOUxMaFNPaEh2aWt1a3RLQmxWTzNlYTdzMk1jUmVoNHp1TnoyQmlnckdHU0VMQ3o2eE9pNkppdTRWWmFzVEpQMlF5eVF3a3pONVBaMWpiZkNLK1c3R3grYk5FbGpPWWFSenNIdlQvVitKZktwNWdjeXd3VmM4NGt5SDVPRkxpQlA3V1hYNW5xZ0lPSkpJMkhvMHJMZ3ArY1dGSkpSNzVXK1Jxb0h6TU1MbDg1NUJiT01JZGFHZ3RKelN2eUcvT3MyREgwNkp1NXZhb3NDVjVRZUlNUytZdnh6Ym1SRjMveG9VOGE2YU9BUStzZ0NkT2JyU01DT3dOQzlMSHpzSmEvYUtGMFJFbExMZFpYK0tyYUxuSVB3amZTNjBsc2c4OEpKZEFoQVpLUG9lUWpQVWF1aStwdzRXV1hDMlVhQkEwMExiRUZkVVNZTGlDYWZxTHhVWFdCMjdieVIzWFJqc0FqbFRXTFY0cmpNcTRxajZqYjNBY29rZzhQTzFCWGRQdmpMT1BYMytWaFBvLzl1aGxDRGhnT0VHYVFhMSIsImRhdGFrZXkiOiJBUUVCQUhoK2RTK0JsTnUwTnhuWHdvd2JJTHMxMTV5amQrTE5BWmhCTFpzdW5PeGszQUFBQUg0d2ZBWUpLb1pJaHZjTkFRY0dvRzh3YlFJQkFEQm9CZ2txaGtpRzl3MEJCd0V3SGdZSllJWklBV1VEQkFFdU1CRUVEQWxXeWxCQ0VMckU4aWwxQ0FJQkVJQTdZRUVzNWx5OXhKOUkxeUVPUEt4M20vNlQwZEtHTWV1OXR0Nm5PUW85bXZLZm1jK0pncWZqNE9pZm03TWhrM2w5Z2xHeGxzTWs4dTJnUXBnPSIsInZlcnNpb24iOiIyIiwidHlwZSI6IkRBVEFfS0VZIiwiZXhwaXJhdGlvbiI6MTYwMTk1NDE5Mn0= https://591609902341.dkr.ecr.eu-west-1.amazonaws.com

then past it

docker login -u AWS -p eyJwYXlsb2FkIjoiRGtGMlhtY1BNWjI2cjNpVVEwNG5SbWlUL0RFeHdCUmI3bEZLWFBVZ1pYdGVnYWsrV0JOVjdadjBqZjNYSTlKMnQ3Qlp5amZVV1Zka21pVzFJcG1CTWVJMmxsQm0xa2ZSNHFHWjJ4YW52MUdySnZqdUZFYXN5U2tNUFpaMElpMTV2Sjh4RVhISVo0dkdBVEp6UFc4OHpqd0M1cEY5TktwMFo3WkE3ZGlnVmhZeW1zZndvQ0RRVzZzR0xEY2VzWkIvUXUxamQzM01yQkhQc3RPdGxsYUQ0UDVENFo0QzdLODZqNm9PT1diZGJsSGRuWTUxOXJRaGtKV3RiOW1BQVdQd0JzbWJUU0JTTk1paExuTEVUSUhIOEhBOXZVR0VxWUVjdzYzZ1FBZ0lyeDBPMXlWWXpYSXg2WlFob1JkbnpqcEQrOGczWUEyWWpXWWhlK21sZWk2U3NVRjdxa2JvUHRiYXdvcjRRcWdjdmo0Z0FyS1FMeDczQUlnWVBFVXlLUkxhYUhqc01WcnVTamhGK3R4RkNOSFlHTmdYRHNWU2JWZHBHbE9SRUx2ZUVubHBzZnNMditvMU1DNGVuRDZDcmpadzdjODdMUGk2eWEwc01SVGxveG9MWGc2empKWG5SdFpIcjBjNXJhaG1WYWRacXpsanR0WXR6emI0b205SnJ2UkF4RUdOTW51Vyt6MWNBV2ppUnVndzhkRDh4VWVXSkZvbHdEbkFjT2dieFpweXkwVE1rK3I1SGNOalNUd1VUdndzb0lMOUUxdUgrR3JUMUcvQ2hLSis5SFV0YnBUWUkva2M4Z2dVYXlXU21mZWlOUWdUanlSSk9zY3BHVXpyQ2tZWVlWTm40dHlFYlR0T1E4Q3dUb0FnamszVXl6TUYwWVlDeklXdTdVcmxjSk1CUnprV1MzVHlVbmpLbHhLdHo3WmVwUVhQZVFiM01iNnV4RTYyNmJhb3hydGxrSGh3cG5UU0lURW5oRWw3Z0ZwMU8xUExxUmpCMjdKWStMWHZ1SFZqdUh4K0hCRS9WSXhvVExtOUxMaFNPaEh2aWt1a3RLQmxWTzNlYTdzMk1jUmVoNHp1TnoyQmlnckdHU0VMQ3o2eE9pNkppdTRWWmFzVEpQMlF5eVF3a3pONVBaMWpiZkNLK1c3R3grYk5FbGpPWWFSenNIdlQvVitKZktwNWdjeXd3VmM4NGt5SDVPRkxpQlA3V1hYNW5xZ0lPSkpJMkhvMHJMZ3ArY1dGSkpSNzVXK1Jxb0h6TU1MbDg1NUJiT01JZGFHZ3RKelN2eUcvT3MyREgwNkp1NXZhb3NDVjVRZUlNUytZdnh6Ym1SRjMveG9VOGE2YU9BUStzZ0NkT2JyU01DT3dOQzlMSHpzSmEvYUtGMFJFbExMZFpYK0tyYUxuSVB3amZTNjBsc2c4OEpKZEFoQVpLUG9lUWpQVWF1aStwdzRXV1hDMlVhQkEwMExiRUZkVVNZTGlDYWZxTHhVWFdCMjdieVIzWFJqc0FqbFRXTFY0cmpNcTRxajZqYjNBY29rZzhQTzFCWGRQdmpMT1BYMytWaFBvLzl1aGxDRGhnT0VHYVFhMSIsImRhdGFrZXkiOiJBUUVCQUhoK2RTK0JsTnUwTnhuWHdvd2JJTHMxMTV5amQrTE5BWmhCTFpzdW5PeGszQUFBQUg0d2ZBWUpLb1pJaHZjTkFRY0dvRzh3YlFJQkFEQm9CZ2txaGtpRzl3MEJCd0V3SGdZSllJWklBV1VEQkFFdU1CRUVEQWxXeWxCQ0VMckU4aWwxQ0FJQkVJQTdZRUVzNWx5OXhKOUkxeUVPUEt4M20vNlQwZEtHTWV1OXR0Nm5PUW85bXZLZm1jK0pncWZqNE9pZm03TWhrM2w5Z2xHeGxzTWs4dTJnUXBnPSIsInZlcnNpb24iOiIyIiwidHlwZSI6IkRBVEFfS0VZIiwiZXhwaXJhdGlvbiI6MTYwMTk1NDE5Mn0= https://591609902341.dkr.ecr.eu-west-1.amazonaws.com

WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/ec2-user/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded


19. docker push 591609902341.dkr.ecr.eu-west-1.amazonaws.com/jholderguru-ecr
#Image will be pushed to ecr repo

#check Amazon Container Service on the console

20. The image should be created
- copy  from amazon container service
-go to clusters >get started with>configure>container name and put in the cpoied URI to our image>200 for memory 80 for the port CPU units 100>#clicknext #Name ClusterECRTest #it will create a new VPC seeing as we are suing the Cloud Formation

21. Copy Public IP address  from tasks and paste in a new tab to see if it is running

So just to recap what I've done is installed Docker on an EC2 instance.

I've downloaded the ubuntu Image and then we've customized that image uploaded it to the Elastic Container Registry
 which was created from the command line and then launched a Fargate cluster with a task that
pulled that image down.
