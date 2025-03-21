# a428-cicd-labs
# Continuous Integration Pipeline dengan Jenkins

Latihan ini akan membantu Anda dalam membangun Continuous Integration (CI) Pipeline menggunakan Jenkins untuk aplikasi React.

## Prasyarat
Sebelum memulai, pastikan Anda telah menginstal:
- Docker
- Git
- Visual Studio Code (atau editor teks lainnya)

## Langkah-langkah

### 1. Menjalankan Jenkins di Docker
Jalankan perintah berikut untuk membuat jaringan Docker:
```sh
docker network create jenkins
```
Kemudian, jalankan Docker container untuk Jenkins dengan perintah berikut:
```sh
docker run \
  --name jenkins-docker \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  --publish 3000:3000 \
  --restart always \
  docker:dind \
  --storage-driver overlay2
```

### 2. Menjalankan Blue Ocean
Buat Dockerfile dengan isi berikut:
```Dockerfile
FROM jenkins/jenkins:2.346.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \  
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \  
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \  
  https://download.docker.com/linux/debian \  
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"
```
Build image tersebut:
```sh
docker build -t myjenkins-blueocean:2.346.1-1 .
```
Jalankan container Blue Ocean:
```sh
docker run \
  --name jenkins-blueocean \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --restart=on-failure \
  myjenkins-blueocean:2.346.1-1
```

### 3. Menyiapkan Jenkins
1. Akses Jenkins di `http://localhost:8080`
2. Ambil password awal dengan perintah:
   ```sh
   docker logs jenkins-blueocean
   ```
3. Masukkan password pada halaman Jenkins
4. Pilih **Install suggested plugins**
5. Buat akun administrator dan selesaikan setup

### 4. Fork dan Clone Repository React App
1. Fork repository dari **Dicoding Academy**
2. Clone repository ke local dengan perintah:
   ```sh
   git clone -b react-app https://github.com/USERNAME-AKUN-GITHUB-ANDA/a428-cicd-labs.git
   ```
3. Buka project di Visual Studio Code

### 5. Membuat Pipeline Project di Jenkins
1. Buka **Jenkins > New Item**
2. Masukkan nama pipeline (misal `react-app`)
3. Pilih **Pipeline** dan klik **OK**
4. Di bagian **Pipeline script from SCM**, pilih **Git**
5. Masukkan URL repository lokal: `/home/Documents/Belajar_Implementasi_CICD/Jenkins/a428-cicd-labs`
6. Pada bagian **Branch Specifier**, isi dengan `*/react-app`
7. Klik **Save**

### 6. Membuat Jenkinsfile
1. Buat file `Jenkinsfile` di root project
2. Salin isi berikut:
   ```groovy
   pipeline {
       agent {
           docker {
               image 'node:16-buster-slim'
               args '-p 3000:3000'
           }
       }
       stages {
           stage('Build') {
               steps {
                   sh 'npm install'
               }
           }
           stage('Test') {
               steps {
                   sh './jenkins/scripts/test.sh'
               }
           }
       }
   }
   ```
3. Simpan file dan commit ke repository:
   ```sh
   git add .
   git commit -m "Add Jenkinsfile"
   ```

### 7. Menjalankan Pipeline di Jenkins
1. Buka **Jenkins > Open Blue Ocean**
2. Pilih `react-app` dan klik **Run**
3. Klik **OPEN** untuk melihat proses eksekusi pipeline
4. Pastikan proses berjalan sukses dengan tanda hijau

### 8. Menghentikan Jenkins dan Docker
Untuk menghentikan Jenkins dan Docker, jalankan:
```sh
docker stop jenkins-blueocean jenkins-docker
```
Untuk menjalankan ulang Jenkins:
```sh
docker start jenkins-blueocean jenkins-docker
```
