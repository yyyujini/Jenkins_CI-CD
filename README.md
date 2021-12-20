# Getting Started Jenkins CI/CD #1
```
- IaaS - On Premise 방식의 Jenkins 구축
- Continuous Integration Pipeline 구축
```
![image](https://user-images.githubusercontent.com/75412225/146697190-0555c5fd-63cf-45c2-bf2c-a5eca0e1745c.png)

### 사전준비
```
➔	Git Project : https://github.com/yyyujini/Jenkins_CI-CD.git
➔	ECR Full Access - EC2
➔	ECR Repository : 917466992560.dkr.ecr.ap-northeast-2.amazonaws.com/demo
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 917466992560.dkr.ecr.ap-northeast-2.amazonaws.com |
aws ecr create-repository --repository-name demo --region ap-northeast-2
➔	Slack Channel
```

### Pipeline 구축을 위한 인프라 셋팅
```
➔	EC2 Instance (Ubuntu)
➔	Docker
➔	Jenkins & awscli : docker pull yyyujini/jenkins-awscli:4
```
```
-- Docker 설치
curl -fsS: https://get.docker.com/ |sudo sh 		# Docker 설치
sudo usermod -aG docker $USER 			            # 접근 권한 
docker -V						                            # 설치 검증

-- Jenkins 설치
docker pull yyyujini/jenkins-awscli:4				    # Images pull
docker run -v /var/run/docker.sock:/var/run/docker.sock -d -p 8080:8080 
yyyujini /jenkins-awscli:4				           		# Jenkins Container RUN
docker ps -a							                    	# 설치 검증

-- Jenkins 로그인
Jenkins 초기 비밀번호 확인 후 로그인
좌측 메뉴 -> 새로운 Item -> Pipeline 등록

-- Job 설정
Github Project : https://github.com/ yyyujini/Jenkins_CI-CD 
GitHub hook trigger for GITScm polling

-- Pipeline 설정
Definition : Pipeline script from SCM
SCM : Git
Repository URL : https://github.com/ yyyujini/Jenkins_CI-CD 
Branch Path : */master
Script File : Jenkinsfile
저장 -> Build Test
```
#### Jenkinsfile 예시
![image](https://user-images.githubusercontent.com/75412225/146697598-8873e8bb-b5bc-498f-aaee-22e767166df4.png)

#### Github Webhook 설정
![image](https://user-images.githubusercontent.com/75412225/146697654-b88a95db-866b-41f8-9efa-988839ae301c.png)

#### Github Webhook 검증
![image](https://user-images.githubusercontent.com/75412225/146697663-df14a1e5-7394-44a0-b84d-788ac49570dd.png)
![image](https://user-images.githubusercontent.com/75412225/146697669-15ce880c-c9e5-4ee8-b50f-80bfa0d51f7e.png)

#### CI to ECR(ec2 - ecr full access)
##### Jenkinsfile에 Docker Image 빌드 및 ECR PUSH stage 추가
![image](https://user-images.githubusercontent.com/75412225/146697783-4ce8acfd-0334-43fd-a91b-39e4efc3c2ba.png)
![image](https://user-images.githubusercontent.com/75412225/146697791-a61d4148-1e05-4541-a550-8d681a32ffaa.png)

#### Jenkins Console Output
![image](https://user-images.githubusercontent.com/75412225/146697829-53293507-af88-4782-a01c-980ff769e9d6.png)


#### ECR Console 검증
![image](https://user-images.githubusercontent.com/75412225/146697839-70ab50ee-d5bd-4ca9-989c-5eaf8a208c1c.png)

#### ECR Image Pull
```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 612545637826.dkr.ecr.ap-northeast-2.amazonaws.com
docker pull 612545637826.dkr.ecr.ap-northeast-2.amazonaws.com/test:1
```

#### Container 실행
![image](https://user-images.githubusercontent.com/75412225/146697862-a7447bb6-4436-480f-b70e-d4ebddf3c8dd.png)
![image](https://user-images.githubusercontent.com/75412225/146697865-7e1898c1-5517-4020-88ca-ffd8e024fe90.png)




