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



# Getting Started Jenkins CI/CD #2
```
- Continuous Deploy Pipeline 구축
```
### 사전준비
```
➔	Getting Started Jenkins CI/CD #1의 모든 작업
➔	추가 Container 배포를 위한 추가 EC2 Instance(docker & aws cli 설치 필요)
```
#### Jenkins Plugin Download - SSH Agent Plugin
![image](https://user-images.githubusercontent.com/75412225/146697962-c6606b7a-507b-454c-8630-1ebbf94eb670.png)
#### SSH Key 등록 - Jenkins Credentials
![image](https://user-images.githubusercontent.com/75412225/146697989-ff6dd740-3791-4ceb-bb26-cfcf7f32c3d8.png)
#### Jenkinsfile 수정
##### 환경 변수 추가
![image](https://user-images.githubusercontent.com/75412225/146698107-62bbbc64-fa43-4460-ae92-72698cf00753.png)
```
TARGET_SVR_USER : SSH 접속 Command에 사용
TARGET_SVR : SSH 접속 Command에 사용
CONTAINER_NAME : docker run or docker rm -f Command에 사용
```
##### stage 추가 - Prompt for deploy
![image](https://user-images.githubusercontent.com/75412225/146698124-507772ed-47d8-4861-bb27-0d29bc3d3ed5.png)
##### stage 추가 - Deploy workload
![image](https://user-images.githubusercontent.com/75412225/146698131-1876bbbb-2b2e-4c11-9b02-7e80ee4b90ae.png)
#### 실행 결과
##### Prompt for deploy - 담당자 승인 절차 진행
![image](https://user-images.githubusercontent.com/75412225/146698155-f1a08c7d-cd81-420d-bee1-0c087b27ba28.png)
##### Deploy workload - ① ssh 접속 -> ② 이미 존재하는 container 삭제 -> ③ docker run
![image](https://user-images.githubusercontent.com/75412225/146698167-de016c1b-8d34-4670-88f1-f1ad0d1157b5.png)
##### container 상태 확인
![image](https://user-images.githubusercontent.com/75412225/146698249-cf20d7ea-a7e6-43e7-8805-576d74f79760.png)
##### application 호출
![image](https://user-images.githubusercontent.com/75412225/146698261-4646a03b-ec91-4627-b24b-4aa1b58de504.png)


# Getting Started Jenkins CI/CD #3
```
- Jenkins Slave 구축을 통한 Job 분산 처리
```
### 사전준비
```
➔	Getting Started Jenkins CI/CD #1,2의 모든 작업
➔	추가 Slave 구축를 위한 추가 EC2 Instance 2개 (docker,git,java 등 task에 필요한 S/W설치)
sudo apt install openjdk-8-jdk git -y
curl -fsS: https://get.docker.com/ |sudo sh
sudo usermod -aG docker $USER
```
![image](https://user-images.githubusercontent.com/75412225/146698314-94098308-8677-4908-90cb-b90f51260b25.png)

#### Node 등록
![image](https://user-images.githubusercontent.com/75412225/146698371-8cd7f737-7cef-4d78-90d9-de11abae5a40.png)
![image](https://user-images.githubusercontent.com/75412225/146698378-f5e6af7a-6f1a-43a1-ab31-563f8e3df460.png)

#### Deploy & Build 두개 노드 등록 완료
![image](https://user-images.githubusercontent.com/75412225/146698396-1eb1537b-7c70-437c-925e-546b57e91c31.png)

#### Jenkinsfile 수정 - 각각의 Stage에 Agent Label 추가
![image](https://user-images.githubusercontent.com/75412225/146698549-622bdea7-faa0-40a7-88ea-d21a6c805506.png)
![image](https://user-images.githubusercontent.com/75412225/146698556-9f84c23e-9e2b-45d3-9ba4-8080a9de5319.png)
![image](https://user-images.githubusercontent.com/75412225/146698561-58bc264a-08b9-46b2-9bda-b4fdeb8612c4.png)
![image](https://user-images.githubusercontent.com/75412225/146698565-ff24ad62-6a65-41f1-bb2c-7f86be635ea1.png)

#### 검증
##### Dashboard
![image](https://user-images.githubusercontent.com/75412225/146698584-c5fb27ac-88b7-4118-bd61-7ff70a2464e1.png)
##### Build Docker Image Stage
![image](https://user-images.githubusercontent.com/75412225/146698599-27e005ac-7f37-4763-8bd6-3f02948182b9.png)
##### Push Docker Image Stage
![image](https://user-images.githubusercontent.com/75412225/146698623-82be7d4e-4fdf-4a2b-b8d3-7cd3371f9678.png)
##### Deploy workload Stage
![image](https://user-images.githubusercontent.com/75412225/146698633-c7a288fd-7bfc-4223-82c0-ca46d6cd6b21.png)
