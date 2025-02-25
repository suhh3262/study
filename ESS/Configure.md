## 구성
###### 클러스터를 먼저 짜야 로직이 들어감
### 1. 클러스터 생성<br/>
  #### 1.1 PATH 지정 // 환경변수 지정 <br>
   ```
     vi ~/.bashrc
     export PATH=/usr/lpp/mmfs/bin:$PATH
   ```
  
  ##### 1.1.1 참고
      `mmcrcluster` : {}는 필수옵션, 보통 NodeFile로 생성 <br>
      `man mmcrcluster` : /example 검색해서 n (next)로 예시들 확인</br>


  #### 1.2 생성
  ```
  mmcrcluster -N /root/ecore/node
     
  # ESS는 필요 패키지가 다 깔려서 옴
  # 정해진 tools로 다 하기 때문에 rpm은 건들지 않아도 됨
  ```
  </br>
  
  ```
  vi /etc/hosts
     
    # hostname 등록
    # bmc 망을 통해 health check
    # ib network 망은 따로 짜줌
     e.g. 172.20.0.21 sss6k01a-hs.gpfs sss6k01a-hs h1 //호스트네임 정리를 잘해줘야 함
  ```

  </br>
       ▶ sss6k01a-hs이 아니라 sss6k01a.hs인 경우, 다른 곳으로 통신될 수 있기 때문에 주의해야 함! </br>
       ▶ 대시는 인식 가능하지만 .은 인식하지 못하기 때문 <br>
  </br>

  `mmgetstate -a` : 노드가 전부 다 나옴
  <br>
  `mmgetstate -help` + `h` : 숨은 옵션들 다 나옴 <br>
  `-a`: all <br>
  `-N`: Node 지정 </br>
