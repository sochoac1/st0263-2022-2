# st0263-2022-2

## instalación de docker en AMI Ubuntu:22.04

### Instalar docker en ubuntu:22.04

    sudo apt update
    sudo apt install docker.io -y
    sudo apt install docker-compose -y
    sudo apt install git -y

    sudo systemctl enable docker
    sudo systemctl start docker
    sudo usermod -a -G docker ubuntu