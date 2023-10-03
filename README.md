문서정보 : 2023.08.25.~09.05. 작성, 작성자 [@SAgiKPJH](https://github.com/SAgiKPJH)

<br>

# Monitoring
모니터링에 대한 모든 할 수 있는 내용을 정리합니다.

### 요약
- powershell, docker grafana prometheus를 활용한 모니터링입니다.  

<img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/02c33b6e-8cae-4fce-a903-72ac148fb632" width="30%"/>
<img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/d13fc51d-69bc-4903-add0-c04f326fa64a" width="30%"/>
<img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/2ac8ab43-aa62-4fc4-bdfa-55ef169d8861" width="30%"/>

### 목표
- [x] : [Process Poweshell 모니터링](#process-poweshell-모니터링)
- [x] : [Docker Grafana Prometeus 활용한 모니터링](#docker-grafana-prometeus-활용한-모니터링)
- [x] : [Docker Grafana Prometeus 활용한 다중 PC 모니터링](#docker-grafana-prometeus-활용한-다중-pc-모니터링)
- [x] : [Docker Grafana Prometeus 활용한 nvidia GPU 모니터링](#docker-grafana-prometeus-활용한-nvidia-gpu-모니터링)

### 제작자
[@SAgiKPJH](https://github.com/SAgiKPJH)

<br><br>

---

<br><br>


# Process Poweshell 모니터링

- PowerShell로 특정 Process의 CPU 사용량 및 메모리 사용량을 획득합니다.
- 기본적인 명령어는 다음과 같습니다.  
  ```shell
  Get-Process

  Get-Process -Name notepad
  ```
  ![image](https://github.com/SagiK-Repository/Monitoring/assets/66783849/9fb05ba6-b4af-4cac-865e-d857d9fee276)  
- 다음과 같이 특정 프로세스의 CPU, Memory 사용량을 획득할 수 있습니다.
  ```shell
  (Get-Process -name PBM_Reader).CPU
  (Get-Process -name PBM_Reader).WorkingSet
  ```
  ![image](https://github.com/SagiK-Repository/Monitoring/assets/66783849/57051bbf-25d7-41ca-a707-c3ec17971d1a)
- Power Shell에서 다음과 같이 코드를 구성하여 CPU 및 Memory 사용량을 획득할 수 있습니다.
  ```shell
  while ($true) {
    $process = Get-Process -Name notepad
    Write-Host "CPU Usage: $($process.CPU)"
    Write-Host "Memory Usage: $($process.WorkingSet)"
    Start-Sleep -Seconds 5
  }
  ```
  ![image](https://github.com/SagiK-Repository/Monitoring/assets/66783849/d08b8ffe-63a1-408c-96cf-b76c33ed0c53)
- 이를 파일로 저장하면 다음과 같습니다.  
  ```shell
  $LogFilePath = ".\PBM_Reader_Usage_Log.txt"
  
  while ($true) {
      $process = Get-Process -Name PBM_Reader
      $CurrentTime = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
      $CPUUsage = $process.CPU
      $MemoryUsage = $process.WorkingSet
  
      $Output = "$CurrentTime	CPU Usage:	 $CPUUsage 	Memory Usage:	 $MemoryUsage"

      # 로그 파일에 기록
      Add-Content -Path $LogFilePath -Value $Output

      # 화면에 출력
      Write-Host $Output
      Write-Host ""
  
      Start-Sleep -Seconds 5
  }
  ```
- 이를 (.ps1) 파일로 저장하여 스크립트 파일로 만들어 실행할 수 있도록 합니다.  
  ![image](https://github.com/SagiK-Repository/Monitoring/assets/66783849/02c33b6e-8cae-4fce-a903-72ac148fb632)


<br><br><br>

# Docker Grafana Prometeus 활용한 모니터링

- PC의 CPU, Memory 사용량 등을 모니터링합니다.
- 기본적으로 다음과 같이 모니터링을 진행합니다.
  1. Docker로 띄운 NodeExporter(PC 정보 수집, Prometeus 제공 프로그램 사용)로 정보 수집
  2. Docker로 띄운 Prometeus로 정보 전송
  3. Docker로 띄운 Grafana로 정보 모니터링
 
* 참조 사이트 : https://developer-nyong.tistory.com/49

- 위 세가지 조건을 만족하도록 Docker-compose를 작성합니다. (docker-compose.yml)
  ```yml
  version: '3'
  services:
    node-exporter:
      image: prom/node-exporter
      ports:
        - 9100:9100

    prometheus:
      image: prom/prometheus
      ports:
        - 9090:9090
      volumes:
        - ./prometheus.yml:/etc/prometheus/prometheus.yml

    grafana:
      image: grafana/grafana
      ports:
        - 3000:3000

  networks:
    promnet:
      driver: bridge
  ```
- 그리고 prometeus.yml을 작성하여 환경을 설정합니다.
  ```yml
  global:
    scrape_interval:     15s

  scrape_configs:
    - job_name: 'node-exporter'
      static_configs:
        - targets: ['node-exporter:9100']
  ```
- 다음 명령어를 통해 실행합니다.
  ```shell
  cd "Docker-Compose Directory"
  docker-compose -f docker-compose.yml up -d
  ```  
  - 다음과 같이 설치가 되며 실행 됩니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/b6d61196-cbf1-46cd-aaa3-0f2bea272548"/>  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/b36b72c0-6e44-4335-9252-605d0489f9d9"/>  
- 웹브라우저를 통해 node-expoter에 접속(http://localhost:9100/metrics)하여 정상 수집하고 있는지 확인합니다.
- 웹브라우저를 통해 Prometeus의 Port로 접속(http://127.0.0.1:9090) -> Status > Targets > node-exporter를 확인합니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/8c94ea7c-4d85-4d04-b251-644234828bf7"/>  
- 마찬가지로 Grafana로 접속(http://127.0.0.1:3000), Login admin/admin, Menu > Dashboards > New > Imports > 1860 입력 > Load > Import >
  - 프로메테우스 미설정시 : prometeus - Configure a new data source > Prometeus > URL 입력 (127.0.0.1 또는 LocalHost가 아닌 IP번호를 입력해야 합니다) > Save & Test  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/5b979031-3e98-47fb-8b64-cbee7b0c6cca"/>  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/f5211d3e-deca-4299-a2a0-bd2091c571f0"/>

<br/><br/><br>

# Docker Grafana Prometeus 활용한 다중 PC 모니터링

- 기존 [Docker Grafana Prometeus 활용한 모니터링](#docker-grafana-prometeus-활용한-모니터링)에서 다음 내용을 추가합니다.
  - Port를 변경합니다. (3000 -> 13000, 9090 -> 19090, 9000 -> 19000)
  - Prometeus에서 입력 Port를 추가합니다. (총 2대의 PC 정보를 추가하기 때문에 2개의 Port를 추가합니다)
  - 각 PC에 node-expoter docker를 실행합니다.
- docker-compose.yml
  ```yml
  version: '3'
  services:
    node-exporter:
      image: prom/node-exporter
      ports:
        - 19100:9100

    prometheus:
      image: prom/prometheus
      ports:
        - 19090:9090
      volumes:
        - ./prometheus.yml:/etc/prometheus/prometheus.yml

    grafana:
      image: grafana/grafana
      ports:
        - 13000:3000

  networks:
    promnet:
      driver: bridge
  ```
- prometeus.yml을 작성하여 환경을 설정합니다.
  ```yml
  global:
    scrape_interval:     15s

  scrape_configs:
    - job_name: 'node-exporter-170-3'
      static_configs:
        - targets: ['192.168.170.3:19100']
    - job_name: 'node-exporter-100-185'
      static_configs:
        - targets: ['192.168.100.185:19100']
    - job_name: 'node-exporter-70-27'
      static_configs:
        - targets: ['192.168.70.27:19100']
  ```
- 다음 명령어를 통해 실행합니다.
  ```shell
  docker-compose -f docker-compose.yml up -d
  ```  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/5b74553f-a249-427e-8509-6bb8496b32bf"/>  
- 각 PC에는 다음을 통해 node-expoter를 구성합니다.
  ```shell
  docker run -d -p 19100:9100 --name=node-exporter prom/node-exporter
  ```  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/2c9a2335-2fff-48d5-b3dd-cbabb4a7e25d"/>  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/22b98722-09ab-4f8b-b002-1af1ae9cdd80"/>  
- http://http://localhost:19090에 접근하여 Prometheus가 정상 수집하는지 확인한다.
  - Status > Targets  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/66b75e1a-069a-4f15-95b0-4bce74b7bd41"/>
- 다음과 같이 상태를 확인할 수 있습니다.   
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/d13fc51d-69bc-4903-add0-c04f326fa64a"/>  
- Docker Compose를 Down(삭제)을 할 때는 다음과 같이 합니다.
  ```bash
  docker-compose -f docker-compose.yml down 
  ```
- 이미 실행중인 Docker Compose 내용을 업데이트 할 경우엔, 처음 실행한 내용과 같이 명령어를 입력합니다.


<br/><br/><br>

# Docker Grafana Prometeus 활용한 nvidia GPU 모니터링

- Nvidia smi Expoter를 통해 GPU 상태를 모니터링 합니다.

### NVIDIA DCGM-Exporter

- Nvidia에서 제공하는 Expoter가 존재합니다.
  - [GitHub NVIDIA/dcgm-expoter](https://github.com/NVIDIA/dcgm-exporter)
  - [DockerHub NVIDIA/dcgm-expoter](https://hub.docker.com/r/nvidia/dcgm-exporter)
- 다음과 같이 Docker Run 합니다.
  ```bash
  docker run -d --gpus all --rm -p 19101:9400 --name=nvidia-dcgm-exporter nvcr.io/nvidia/k8s/dcgm-exporter:3.2.5-3.1.7-ubuntu20.04
  ```
  - `unknown flag : --gpus` -> 버전 차이로 나타난 문제, 다음과 같이 입력합니다. (https://kkkkhd.tistory.com/426)
  ```bash
  nvidia-docker run -d --rm -p 19101:9400 --name=nvidia-dcgm-exporter nvcr.io/nvidia/k8s/dcgm-exporter:3.2.5-3.1.7-ubuntu20.04
  ```
- 다음과 같이 Exporter가 정상 수집하는지 확인합니다.  
  - `curl localhost:19101/metrics`
  - 또는 사이트 `http://localhost:19101/metrics` 접근합니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/8b0e4f55-af85-4bdb-85f4-50227155e75e" />
- Grafana는 다음과 같이 진행합니다.  
  - 사이트 [Grafana NVIDIA DCGM Exporter Dashboard](https://grafana.com/grafana/dashboards/12239-nvidia-dcgm-exporter-dashboard/) DashBoard를 띄웁니다.
  - 또는 다음 사이트를 활용할 수 있습니다. (ID : 15117) [Grafana NVIDIA DCGM Exporter](https://grafana.com/grafana/dashboards/15117-nvidia-dcgm-exporter/)
  - Grafana ID를 12239로 입력하여 Load합니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/2ac8ab43-aa62-4fc4-bdfa-55ef169d8861" />

 
<details>  
<summary>docker compose nvidia 추가한 내용 확인</summary>  

- 위 nvidia를 추가한 docker compose를 아래와 같이 구성할 수 있습니다.  
  ```yml
  version: '3'
  services:
    node-exporter:
      image: prom/node-exporter
      ports:
        - 19100:9100

    prometheus:
      image: prom/prometheus
      ports:
        - 19090:9090
      volumes:
        - ./prometheus.yml:/etc/prometheus/prometheus.yml

    grafana:
      image: grafana/grafana
      ports:
        - 13000:3000

    nvidia-dcgm-exporter:
      image: nvcr.io/nvidia/k8s/dcgm-exporter:3.2.5-3.1.7-ubuntu20.04
      ports:
        - 19101:9400
      runtime: nvidia

  networks:
    promnet:
      driver: bridge
  ```
- 단, Docker-Compose를 위한 NVIDIA 관련 파일을 설치해야 합니다.  
</details>  














  
<details>  
<summary>보다 더 자세히 알아보기</summary>  

### utkuozdemir nvidia_gpu_exporter
- 활용할 nvidia-smi-expoter은 다음과 같습니다.
  - [GitHub utkuozdemir/nvidia_gpu_exporter](https://github.com/utkuozdemir/nvidia_gpu_exporter)
  - [Grafana utkuozdemir/nvidia_gpu_exporter](https://grafana.com/grafana/dashboards/14574-nvidia-gpu-metrics/)
  - [Docker Hub utkuozdemir/nvidia_gpu_exporter](https://hub.docker.com/r/utkuozdemir/nvidia_gpu_exporter)
- 다음과 같이 Docker Run 합니다. 
  ```bash
  docker run -d --name nvidia-exporter --privileged -p 19101:9835 utkuozdemir/nvidia_gpu_exporter:1.2.0
  ```  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/8da99dcd-72f9-4123-abcb-d11540742923" />
- [Docker Grafana Prometeus 활용한 다중 PC 모니터링](#docker-grafana-prometeus-활용한-다중-pc-모니터링)와 같이 prometheus.yml파일을 수정합니다.
  ```yml
  global:
    scrape_interval:     15s

  scrape_configs:
    - job_name: 'node-exporter-170-3'
      static_configs:
        - targets: ['192.168.170.3:19100']
    - job_name: 'node-exporter-100-185'
      static_configs:
        - targets: ['192.168.100.185:19100']
    - job_name: 'node-exporter-70-27'
      static_configs:
        - targets: ['192.168.70.27:19100']

    - job_name: 'node-exporter-100-185-GPU'
      static_configs:
        - targets: ['192.168.100.185:19101']
    - job_name: 'node-exporter-70-27-GPU'
      static_configs:
        - targets: ['192.168.70.27:19101']
  ```
- Grafana ID를 14574로 입력하여 Load합니다.


### Grafana 내용 수정
- 현재 GPU의 Grafana는 단일 Job에 대해서 동작합니다.
- 이를 여러 Job을 선택할 수 있도록 개선합니다.
- 생성한 GPU의 DashBoard에서 Settings (톱니바퀴) > Variables > [+ New Variable]
- New Variable Job
  - Name : job
  - Label : Job
  - Query
    - Type : Label values
    - Label : job
    - Metric : node_uname_info
  - Sort : Alphabetical (asc)  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/fcf218e6-9cff-4a79-b811-2d833f5c3cb6" />  
- New Variable node
  - Name : node
  - Label : Host:
  - Query
    - Type : Label values
    - Label : instance
    - Metric : node_uname_info
    - Label filters : job = $job
  - Sort : Alphabetical (asc)  
 <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/38c57da1-189e-403e-83c7-471dfb45ec2e" />  
- Variable 생성시 맨아래 Run Query가 존재합니다. 이를 통해 정확한 값이 나오는지 확인합니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/45fec894-e742-4d8e-a243-81bf0070b010" />  
- 다음과 같이 Variable를 확인합니다.  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/52078f34-0277-4353-b435-08bf4a3b23fe" />  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/2a76b664-3a31-4223-82ac-d7a7caf352af" />
- 이후 각 DashBoard Item의 값을 다음과 같이 수정(Edit)합니다.
  - Edit > Query
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/4de2772d-2203-404c-b4a6-debeb7a06cb3" />  
  ```json
  # As-Is
  nvidia_smi_utilization_gpu_ratio{uuid="$gpu"}

  # To-Be
  nvidia_smi_utilization_gpu_ratio{instance="$node", job="$job", uuid="$gpu"}
  ```
- 작성중...


<br/>

### BugRoger nvidia-expoter

- 활용할 nvidia-smi-expoter은 다음과 같습니다.
  - [Grafana BugRoger Nvidia-smi-exporter](https://grafana.com/grafana/dashboards/6387-gpus/)
  - [GitHub BugRoger/nvidia-smi-exporter](https://github.com/BugRoger/nvidia-exporter)
  - [Docker Hub bugroger/nvidia-exporter](https://hub.docker.com/r/bugroger/nvidia-exporter)
- 다음과 같이 Docker Run 합니다. 
  ```bash
  docker run -d --name nvidia-exporter --privileged -p 19101:9401 bugroger/nvidia-exporter:latest
  ```
- 정상동작 함을 접속하여 확인합니다. (http://localhost:19101/metrics)  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/3fbfc7dc-9fd8-4cbd-baa4-c86dc7993630" />  
- [Docker Grafana Prometeus 활용한 다중 PC 모니터링](#docker-grafana-prometeus-활용한-다중-pc-모니터링)와 같이 prometheus.yml파일을 수정합니다.
  ```yml
  global:
    scrape_interval:     15s

  scrape_configs:
    - job_name: 'node-exporter-170-3'
      static_configs:
        - targets: ['192.168.170.3:19100']
    - job_name: 'node-exporter-100-185'
      static_configs:
        - targets: ['192.168.100.185:19100']
        - targets: ['192.168.100.185:19101']
    - job_name: 'node-exporter-70-27'
      static_configs:
        - targets: ['192.168.70.27:19100']
        - targets: ['192.168.70.27:19101']
  ```
- Grafana ID를 6387로 입력하여 Load합니다.

### 수동으로 Build 할 시
- 위의 BugRoger/nvidia-smi-exporter GitHub를 다운받아 Dockerfile을 빌드합니다.
  ```shell
  docker build -t bugroger/nvidia-smi-exporter .
  ```  
  <img src="https://github.com/SagiK-Repository/Monitoring/assets/66783849/b8ee4fa6-d57d-4a9d-b325-9185e3cc333b"/>  
- 이후 Docker Run을 합니다.
- Docker Image 공유
  > 또는 BugRoger nvidia-smi-exporter를 활용합니다. (https://hub.docker.com/r/bugroger/nvidia-exporter)
  > Docker Image를 쉽게 실행할 수 있도록 다음과 같이 구성하였습니다. (https://hub.docker.com/r/juhyung1021/bugroger-nvidia-smi-exporter)
- 다음 명령어를 통해 실행합니다.
  ```shell
  docker run -d --gpus all -p 19101:9401 --name=nvidia-smi-exporter juhyung1021/bugroger-nvidia-smi-exporter:latest
  ```  


</details>

