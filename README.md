문서정보 : 2023.08.25.~ 작성, 작성자 [@SAgiKPJH](https://github.com/SAgiKPJH)

<br>

# Monitoring
모니터링에 대한 모든 할 수 있는 내용을 정리합니다.

### 목표
- [x] : [Process Poweshell 모니터링](#process-poweshell-모니터링)
- [ ] : Docker Grafana Prometeus 활용한 모니터링

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


<br><br>

# Docker Grafana Prometeus 활용한 모니터링

- PC의 CPU, Memory 사용량 등을 모니터링합니다.
- 기본적으로 다음과 같이 모니터링을 진행합니다.
  1. PC 정보 수집 (Prometeus 제공 프로그램 사용)
  2. Docker로 띄운 Prometeus로 정보 전송
  3. Docker로 띄운 Grafana로 정보 모니터링
 
* 참조 사이트 : https://developer-nyong.tistory.com/49

### 1. PC 정보 수집 프로그램 셋팅



