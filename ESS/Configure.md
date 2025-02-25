# 구성
###### 클러스터를 먼저 짜야 로직이 들어감
## 1. 클러스터 생성<br/>
  ### 1.1 PATH 지정 // 환경변수 지정 <br>
   ```
     vi ~/.bashrc
     export PATH=/usr/lpp/mmfs/bin:$PATH
   ```
  
  #### 1.1.1 참고
      `mmcrcluster` : {}는 필수옵션, 보통 NodeFile로 생성 <br>
      `man mmcrcluster` : /example 검색해서 n (next)로 예시들 확인</br>


  ### 1.2 생성
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
  `-N`: Node 지정 </br><br>
  `mmchlicense` : 라이센스 등록 <br>
  `mmchlicense server --accept -N sss6k01a-hs, sss6k01b-hs` : 두 개의 노드에 라이센스 선언
  <br></br>
  
  ### 1.3 구성
  `mmvdisk` <br>
  - Usage 부분 위에서부터 5줄 기억 (파일시스템 구성 순서), 그 아래는 조회하는 명령어
  - 위에서부터 5줄 순서대로 설정해줘야 함

  #### 1.3.1 node class
  `mmvdisk nodeclass` : nodeclass 대신 nc 사용가능 <br>
  `mmvdisk nv create --nc BB`: Building Block (ESS가 3대일 경우 BB도 3개) </br>

  ```
  mmvdisk nv create --nc BB01 -N sss6k01a-hs,sss6k01b-hs // 생성
  ```

  #### 1.3.2 server 설정
  `mmvdisk server`<br>
  `mmlsconfig` : 기본 옵션만 출력되는데, 히든 옵션을 통해 생성해줘야 함 </br>

  ```
  mmvdisk server configure // 옵션 출력
  ```

  ```
  mmvdisk server configure --nc BB01 --maxblocksize 16M --pagepool 60% --recycle none
  ```
  - BB01에 노드 두 대가 들어있음
  - 파일이 들어올 때 16M 단위로 쪼갬 (단, 16M 안에서도 서브블록 단위로 계속 쪼개짐)
  - pagepool은 swap 개념인데 cache 역할이라고 생각 (보통 60% 할당)
  - reycle은 재기동 유무 옵션
      - one: 한 번 내려갔다가 올라감
      - recycle은 none으로 해야 함! 기본 옵션은 all인데, gpfs가 모두 꺼지기 떄문에 무조건 none으로 주기

  `mmlsconfig` : 위에 것까지 실행하고 mmlsconfig로 확인 <br>

  #### 1.3.3 RDMA에 대한 config 선언
  - 딱 4줄 들어감!
  ```
  mmchconfig verbsRdma=enable              //RDMA 기능 활성화
  mmchconfig verbsRdmaCm=disable           //RDMA Connection manager 비활성화
  mmchconfig verbsRdmaSend=yes             // RDMA를 통한 데이터 전송 기능 활성화
  mmchconfig verbsPorts='mlx5_0,mlx5_1 등  // 사용할 RDMA 포트 지정
  ```
  port가 변경되면 무조건 재기동 한 번 해줘야 함!<br>

  #### 1.3.4 포트 확인
  `ibstatus` <br>
  - 위에서 지정해준 `'mlx5_0' port 1` 이런 게 출력됨
  - 실제 device name을 줘야 하는데 위에서 지정해 준 `'mlx5_0'` 이런 게 dev name임
  - `ip a` 했을 때 뜨는 랜카드 이름은 디바이스 네임이 아님
  - 선 꽂을 때는 균등하게 3:3 이렇게 꽂아줘야 함 (2:4 이런식으로 꽂으면 성능 저하)
  - verbsPorts는 common으로 설정하지 말고 노드별로 range 사용하기!!
  
  #### 1.3.5 노드 확인
  - `ssh n1`<br>
  - `ibstatus` : 노드에 range를 통해 Port를 지정해줘야 함 <br>
  
  #### 1.3.6 설정 완료 및 start

  ```
  # mmstartup -a    // 모두 켜짐
  # cd /var/mmfs/gen/
  # tail -f mmfslog
  ```
  - 이전 로그까지 확인

  ```
  # ls -la
  # mmfslog > /var/adm/ras/mmfs.log.latest    // 링크를 걸어서 실제 경로는 저렇게 되어 있음
  # cd /var/adm/ras
  /ras# ls
  ```
  - gpfs down 될 때마다 log가 쌓임
  - `mmfs.log.previous` : 과거 로그
  - `mmfs.log.latest` : 최신 로그 <br>

  - `mmgetstate -a` : gpfs 상태 확인
  - `mmfsadm test verbs status` <br>
      - rdma의 상태 확인
      - started로 되어 있으면 시작된 것
      - not started는 ib는 쓰는데, rdma는 사용하지 않는 것
   
  #### 1.3.7 recovery group 생성
  - `mmvdisk rg` : 명령어 출력 <br>

  ```
  mmvdisk rg create --rg RG01    // RG name은 세트당 하나. 이름은 간결하게
  (보통 3세트면 RG도 3개)
  - resize 옵션은 용량 증설 시 사용 (디스크 붙이고 resize)
  ```

  ```
  mmvdisk rg create --rg RG01 --nc BB01
  ```

  - 참고
      - hybrid / capacity(용량) model
        - 용량 베이스
        - nvme가 풀로 꽂힌 게 있고, 4장만 꽂히고 디스크가 붙어있는 게 있음
        - 고용량을 필요로 할 때 사용. 성능은 하이브리드가 더 좋음
        - 하이브리드는 NVMe + SATA 같이 꽂힘

  - `mmvdisk rg list` : rg 출력
  - `mmvdisk nc list` : nc 출력
  - nc랑 rg 모두 출력되면 구성 끝!

  #### 1.3.8 vs 생성 (volume)
  - `mmvdisk vs` <br>
      - define을 통해 정의, Raid Code를 통해 레이드 방식 정의
      - 4way copy는 똑같은 걸 4번 복제 (보통 8+2p 방식 사용)

  ```
  mmvdisk vs define --vs VS001 --rg RG01 --code 8+2p --bs 16M --ss 20%
  ```
  - `--vs` 옵션을 통해 이름 정의
  - `bs` : block size
  - `ss` : set size
  - 보통 2개의 볼륨을 짜줌
      - e.g. 8페타 스토리지, 1개의 볼륨으로 구성 -> 위험. (용량에 맞춰 분할해서 볼륨을 구성해야 함)
  - VS001에 20%를 할당해주었기 때문에 recovery group을 확인하면 100%에서 80%로 변경된 것 확인 가능
  - gpfs는 메타와 데이터를 함께 사용함 (dataAndMetadata)
  - 만약 NVMe와 sata 따로 파일시스템 구성하고 싶으면 볼륨 따로 생성
  - NVMe 특징 : 빠르지만 90% 이상 사용하면 속도가 저하되기 때문에 15~20% 용량은 남겨놔야 함 (FULL 구성 x)

  ```
  mmvdisk vs define --vs VS002 --rg RG01 --code 8+2p --bs 16M --ss 20%
  ```
  - 두 번째 볼륨 생성 완료
  - 볼륨 구성할 때 사이즈는 똑같이 구성해야 함 (성능 이슈)

  ```
  mmvdisk vs list --vs all
  ```
  - 전체 vdisk 옵션이 다 보임
  - 메타데이터는 무조건 system pool로 들어감

  ```
  mmlsnsd
  ```
  - 위에서 볼륨 선언은 했지만 아직 만들지 않았기 때문에 아무것도 뜨지 않음

  

  
  
  
  
