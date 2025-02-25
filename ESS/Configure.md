# 구성
###### 클러스터를 먼저 짜야 로직이 들어감
<br></br><br></br>

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

  <br>
  
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

  `mmlsconfig` : 위에 것까지 실행하고 mmlsconfig로 확인 
  
  <br>

  #### 1.3.3 RDMA에 대한 config 선언
  - 딱 4줄 들어감!
  ```
  mmchconfig verbsRdma=enable              //RDMA 기능 활성화
  mmchconfig verbsRdmaCm=disable           //RDMA Connection manager 비활성화
  mmchconfig verbsRdmaSend=yes             // RDMA를 통한 데이터 전송 기능 활성화
  mmchconfig verbsPorts='mlx5_0,mlx5_1 등  // 사용할 RDMA 포트 지정
  ```
  port가 변경되면 무조건 재기동 한 번 해줘야 함!
  
  <br>

  #### 1.3.4 포트 확인
  `ibstatus` <br>
  - 위에서 지정해준 `'mlx5_0' port 1` 이런 게 출력됨
  - 실제 device name을 줘야 하는데 위에서 지정해 준 `'mlx5_0'` 이런 게 dev name임
  - `ip a` 했을 때 뜨는 랜카드 이름은 디바이스 네임이 아님
  - 선 꽂을 때는 균등하게 3:3 이렇게 꽂아줘야 함 (2:4 이런식으로 꽂으면 성능 저하)
  - verbsPorts는 common으로 설정하지 말고 노드별로 range 사용하기!!

  <br>
  
  #### 1.3.5 노드 확인
  - `ssh n1`<br>
  - `ibstatus` : 노드에 range를 통해 Port를 지정해줘야 함 <br>
  
  <br>
  
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
  <br>
  
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

  <br>
  
  #### 1.3.8 vs 생성 (volume)
  - `mmvdisk vs` <br>
      - define을 통해 정의, Raid Code를 통해 레이드 방식 정의
      - 4way copy는 똑같은 걸 4번 복제 (보통 8+2p 방식 사용)
        
  <br>
  
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
  <br>
  
  ```
  mmvdisk vs define --vs VS002 --rg RG01 --code 8+2p --bs 16M --ss 20%
  ```
  - 두 번째 볼륨 생성 완료
  - 볼륨 구성할 때 사이즈는 똑같이 구성해야 함 (성능 이슈)

  <br>
  
  ```
  mmvdisk vs list --vs all
  ```
  - 전체 vdisk 옵션이 다 보임
  - 메타데이터는 무조건 system pool로 들어감

  <br>
  
  ```
  mmlsnsd
  ```
  - 위에서 볼륨 선언은 했지만 아직 만들지 않았기 때문에 아무것도 뜨지 않음

  <br>
  
  ```
  mmvsidk vs create --vs VS001,VS002   // --vs 옵션 통해 만들어줌 + 하나의 볼륨당 4개의 nsd가 생김
  ```

  ```
  mmlsnsd
  ```
  - VS001에 대한 볼륨 4개, VS002에 대한 볼륨 4개 => 총 8개의 볼륨 생성

  <br>
  
  #### 1.3.9 filesystem 생성 (마지막 단계)
  - `mmvdisk fs`
      - --vdisk-set 뒤에는 volume group name

  ```
  mmvdisk fs create --fs gpfs --vs VS001,VS002 --trim auto --mmcrfs -B 16M -m 2 -n 256 -R /gpfs
  ```
  - `--trim auto` : NVMe, SATA 섞여 있을 때 trim을 선언해줌으로써 디스크를 혼용해서 쓴다는 사실 알려줌.
  - `--mmcrfs` : gpfs의 파일 시스템 생성 유틸리티인 mmcrfs 명령을 내부적으로 사용하여 파일시스템 생성
  - `-m 2` : 메타데이터의 복제본 수 설정 (메타데이터의 이중화를 통해 장애 발생 시 안정성을 높임)
  - `-B 16M` : block size
  - `-n 256` : 해당 파일 시스템에 동시에 접근할 수 있는 최대 노드의 수
  - `-T /gpfs` : 생성된 파일 시스템이 마운트될 경로 지정 (마운트 포인트)
  <br>
  ➡️ /gpfs 경로에, 지정된 볼륨셋 사용 및 자동 trim 기능과 mmcrfs를 통하여 gpfs 타입의 파일 시스템을 생성 </br>
  ➡️ 블록 크기는 16MB, 메타데이터 복제본은 2개, 최대 256 노드 접근 가능
  
  <br></br>

  ```
  # mmvdisk recoverygroup change --rg RG01 --declustered-array DA1 --time-da yes
  # df -h
  ```
  - `--declustered-array DA1` : 해당 rg에 연결할 declustered array의 이름 설정.
      - declustered array는 데이터를 여러 디스크에 분산시켜 저장함으로써 성능 및 장애 대응성을 향상시킴
  - `--time-da yes` : declustered array 사용 시 시간 기반(time-based) 기능 활성화
  - gpfs 파일시스템이 보이지 않는 상황
  - `mmount` : 필수옵션, 파일시스템이 이름과 마운트되어 있는 위치 선언

  <br>
  
  ```
  mmount gpfs -a (all)
  ```
  - mmshutdown -a 사용하면 모든 서버가 다운되기 때문에 조심해야 함

  </br>
  
  ```
  df -h
  ```
  - gpfs 파일시스템 생긴 것 확인 가능


---
<br></br>

## 2. 계산노드 붙이기 <br/>
#### 참고
- Storage_Scale로 시작하는 것은 gpfs 패키지
- Scale_system_DAE로 시작하는 것은 ESS 패키지
- rpm 설치하기 전에 ssh-keygen을 통해 터널링부터 해야 함
<br>

```
# ssh n1    // 1번 노드 접속
# ls
  Storage_Scale_Data_Access-5.2.1.0-x86_64-Linux-install    // 대충 이런식으로 나옴
```

```
# ./Storage_Scale_Data_Access-5.2.1.0-x86_64-Linux-install --text-only
```
- 실행
- /usr/lpp/mmfs/5.2.1.0 밑으로 파일들이 모두 이동됨

<br>
  
```
# cd /usr/lpp/mmfs/5.2.1.0
# ls
  gpfs_debs (우분투)
  gpfs_rpms (레드햇)    // 여러 결과값이 나옴
```

```
# cd gpfs_rpms/
# ls  
  gpfs.base-4.2.3-4.x86_64.rpm     gpfs.gpl-4.2.3-4.noarch.rpm         gpfs.ext-4.2.3-4.x86_64.rpm
  gpfs.gskit-8.0.50-75.x86_64.rpm  gpfs.msg.en_US-4.2.3-4.noarch.rpm   gpfs.license.std-4.2.3-4.x86_64.rpm
  gpfs.docs-4.2.3-4.noarch.rpm     gpfs.crypto-4.1.1-0.x86_64.update.rpm
```
- `gpfs.base-4.2.3-4.x86_64.rpm`
    -  gpfs의 핵심 구성 요소 포함. (핵심 바이너리와 라이브러리 패키지)
- `gpfs.gpl-4.2.3-4.noarch.rpm`
    - gpfs에서 사용하는 GPL 라이선스가 적용된 오픈소스 구성요소 포함 (오픈소스도구/모듈)
- `gpfs.ext-4.2.3-4.x86_64.rpm`
    - 추가적인 확장 기능 및 드라이버 제공 (특정 H/W 지원이나 추가 기능 모듈 등이 포함되어 GPFS 기능 확장)
- `gpfs.gskit-8.0.50-75.x86_64.rpm`
    - IBM Global Security Kit. (gpfs에서 암호화 및 보안 통신 위해 필요한 보안 라이브러리/기능 제공)
- `gpfs.msg.en_US-4.2.3-4.noarch.rpm`
    - 시스템 로그, 오류 메시지 등에서 사용할 지역화된 텍스트 제공
- `gpfs.license.std-4.2.3-4.x86_64.rpm`
    - gpfs의 표준 라이선스 관련 파일 포함 패키지 (라이선스 인증 정보 제공)
- `gpfs.docs-4.2.3-4.noarch.rpm`
    - gpfs의 문서 자료 포함 (매뉴얼, 가이드, 참조문서 등 사용 및 관리에 필요한 자료)
- `gpfs.crypto-4.1.1-0.x86_64.update.rpm`
    - 암호화기능 업데이트 패키지 (암호화 관련 모듈의 개선사항, 버그 수정 업데이트 포함)

```
yum install gpfs.base ~ ..    // base, docs, gskit, gpl, msg, crypto 총 6개 설치
```

```
gpfs_rpms# cd ..
5.2.1.0# ls
# ls
  genesha  smb 등 여러 rpm 나옴

# cd ..
mmfs# cd bin/
bin# ./mmbuildgpl
```

- ./mmbuildgpl 하는 이유?
    - gpfs의 특정 커널에서 특정 모듈을 사용하겠다고 선언
    - gpfs는 커널 모듈을 타고 올라오기 때문에 선언해주는 과정 필요
    - 버전은 support metrics 참고
    - `mmlsconfig`에서 `minReleaseLevel 5.2.1.0` 되어 있으면 `5.2.0.9`는 사용 불가
    - 최소 릴리즈 버전임

- rpm 설치 후 클러스터 붙여야 함!!

```
mmaddnode -N /
```
- 노드파일 확인

```
# ssh s1
# cat /root/ecore/client
  node01-hs
  node02-hs
```
- hostname 주의! hostfile은 잘못 건드리면 gpfs가 down되기 때문에 통일시켜야 함

<br>

```
mmaddnode -N /root/ecore/client --accept
```

```
mmcluster  // 확인
```
- quorum-manager / quorum / x
- 아무것도 없는 건 node (client라고 보면 됨)

<br>

```
mmchlicense client --accept -N node01-hs.gpfs,node02-hs.gpfs
```
- 계산노드를 붙이고 클라이언트에 라이센스 부여
- 서버가 아닌 노드들은 보통 클라이언트임!

<br>

```
mmlslicense  // 확인?
```

```
mmgetstate -a
mmlsmgr    // filesystem manager, cluster manager 확인 가능
```

```
mmstartup -N node01-hs,node02-hs
mmgetstate -a
```

```
ssh n1
df -h
```






