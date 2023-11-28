# Python Executable Environment

1. AWS에서 EC2 인스턴스 생성
2. 명령어를 통해 EC2 접속

```
ssh -i /path/to/your-key.pem ubuntu@your-instance-public-dns
```

3. 업데이트와 Dependency 설치

```
sudo apt update && sudo apt upgrade -y
```

4. 도커 설치

```
sudo usermod -aG docker $USER
```

5. 도커 파일 생성

Vim 또는 intellij로 확장자 없는 도커파일(Dockerfile) 생성 후 베이스 이미지 생성

```
FROM python:3.9-slim
WORKDIR /app
CMD ["python3"]

```

6. 도커파일 ec2로 이동

```
scp -i /path/to/your-key.pem /path/to/your/Dockerfile ec2-user@your-instance-public-dns:/path/on/ec2/where/to/put/Dockerfile
```

7. 도커 파일을 통한 파이썬 환경 빌드

```
docker build -t python-execution-env .
```

8. 출력 확인

```
docker run --rm python-execution-env python -c "print('Hello from Python')"
```
