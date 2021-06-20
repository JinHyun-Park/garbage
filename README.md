# 목차
  - [5 Pillars (설계 원칙)](#5-Pillars-설계-원칙)
  - [진행 순서](#진행-순서)
  - [TaskA](#TaskA)
    - [리전](#리전)
    - [가용 영역](#가용-영역)
    - [VPC](#VPC)
    - [EC2](#EC2)
    - [EBS](#EBS)
    - [ELB](#ELB)
    - [AutoScaling](#AutoScaling)
    - [RDS](#RDS)
    - [ElastiCache (Memcached)](#ElastiCache-Memcached)
    - [Security Group](#Security-Group)
    - [S3, CloudFront, VPC Endpoint](#S3-CloudFront-VPC-Endpoint)
      - [S3 (Simsple Storage Service)](#S3-Simsple-Storage-Service)
      - [CloudFront](#CloudFront)
      - [VPC EndPoint](#VPC-EndPoint)
    - [CloudWatch, KMS, IAM, CloudFormation](#CloudWatch-KMS-IAM-CloudFormation)
      - [CloudWatch](#CloudWatch)
      - [KMS (Key Management Service)](#KMS-Key-Management-Service)
      - [IAM (Identify and Access Management)](#IAM-Identify-and-Access-Management)
      - [CloudFormation](#CloudFormation)
  - [5 Pillars](#5-Pillars)
  - [CICD](#CICD)
    - [Code Pipeline](#Code-Pipeline)
    - [Code Commit](#Code-Commit)
    - [Code Build](#Code-Build)
    - [Code Deploy](#Code-Deploy)
  - [MSA](#MSA)
    - [ECR](#ECR)
    - [ECS](#ECS)
    - [Fargate](#Fargate)
    - [fargate 배포](#fargate-배포)
    - [SQS](#SQS)
    - [SNS](#SNS)
  - [Blah Blah](#Blah-Blah)
  - [RFC 1918](#RFC-1918)
  - [VPC 서브넷 권고사항](#VPC-서브넷-권고사항)
  - [AWS Wavelength](#AWS-Wavelength)

# 5 Pillars (설계 원칙)
- __운영 우수성, 보안, 안정성, 성능 효율성, 비용 최적화__
- 운영 우수성
  - 코드를 통한 운영 : __CloudFormation__
  - 되돌릴 수 있는 변경 내용 적용 : __EBS Snapshot__
  - 수시로 운영 절차 수정
  - 실패 예측 : __Auto Scale__
  - 모든 운영상 실패로부터 학습 : __Cloud Watch, CloudTrail__
- 보안
  - 자격증명 : __IAM__
  - 추적 기능 활성화 : __Cloudwatch, CloudTrail__
  - 모든 계층 보안 적용 : __VPC, 엣지, ELB, EC2, Security Group, NACL__
  - 전송 및 보관 중인 데이터 보호 : __암호화, 액세스 제어, SSL__
  - 데이터에 쉽게 액세스할 수 없도록 유지 : __S3 KMS를 통한 암호화 저장, 99.999999% 내구성__
  - 보안 이벤트 대비 : __CloudFront, WAF__
- 안정성
  - 장애 자동 복구 : __EBS Snapshot을 통한 백업 및 복구__
    - RTO 및 RPO에 대한 요구 사항을 충족하도록 데이터, 애플리케이션 및 구성을 백업
  - 복구 절차 테스트 : __AMI__를 통해 장애 발생 시 바로 복구 가능
  - 수평적 확장으로 워크로트 전체 가용성 증대 : __Auto Scaling__
  - 용량 추적
- 성능 효율성
  - 고급 기술의 대중화 : __ElatiCache, CloudFront, CloudWatch, CloudTrail을 통한 지표 수집 및 평가를 통해 성능 개선__
  - 몇 분만에 전 세계에 배포 : __Region을 변경하여 IaC (CloudFormation)__ 를 통해 전세계에 배포
  - 서버리스 아키텍처
  - 실험 횟수 증가
  - 기계적 동조 고려
  - 스토리지 : __S3__
- 비용 최적화
  - 클라우드 재무 관리 시행 : __AWS Budget - 비용과 사용량에 대한 사전 알림__
  - 지출 및 사용량 인식 : __Cloud Watch - right sizing__
  - 비용 효율적인 리소스 : __Auto Scaling, Cost Explore - 비용과 사용량 파악__
    - CloudFront를 통해 데이터 전송 최소화
    - On-demand, Savings Plan, Reserved Instances, Spot Instances
    - serverless
  - 수요 관리 및 리소스 공급 : 
  - 시간 경과에 따른 최적화 : 

# 진행 순서
- AWS Infra
  - AWS 글로벌 클라우드 인프라는 업계에서 가장 안전하고 광범위하고 안정적인 클라우드 플랫폼으로, 전 세계 데에터 센터를 통해 완전한 기능을 갖춘 200가지가 넘는 서비스 제공
  - 25개 리전에서 출시 - 각 리전에서 다중 Availability Zone 사용
  - 80개의 가용 영역
  - 245개 국가와 지역에서 서비스 제공
- Region -> Availability Zone -> VPC(Subnet, IGW, NACL, NAT) -> EC2(ELB), EBS -> RDS -> Security Group -> S3, CloudFront -> 5 plillars -> CI/CD

# TaskA
## 리전
- AWS가 전 세계에서 데이터 센터를 클러스터링하는 물리적 위치를 리전이라고 한다
- 논리적 데이터 센터의 각 그룹을 가용역역이라고 한다
- 각 AWS 리전은 지리적 영역 내에서 격리되고 물리적으로 분리된 여러 개의 AZ로 구성된다
- AWS는 다른 어떤 클라우드 공급자보다 더 광범위한 국제적 입지를 제공하며, 고객이 전 세계에서 서비스를 이용할 수 있도록 빠르게 새로운 리전을 여는 중. 여러 지리적 리전 유지 관리중
- 서울 리전을 선택한 

## 가용 영역
- AZ는 독립된 전원, 냉각 및 물리적 보안을 갖추고 있으며 지연 시간이 매우 짧은 중복 네트워크를 통해 연결
-  ~~AWS 리전의 중복 전력, 네트워킹 및 연결이 제공되는 하나 이상의 개별 데이터 센터로 구성~~
- AZ를 사용하면 단일 데이터 센터를 사용하는 것보다 더 높은 가용성, 내결함성 및 확장성을 갖춘 프로덕션 애플리케이션과 데이터베이스를 운영 가능
- AZ 리전의 모든 AZ는 높은 대역폭, 짧은 지연 시간 네트워킹, 완전한 중복성을 갖춘 전용 메트로 광 네트워크와 연결되어 있어 AZ 간에 높은 처리량과 지연 시간이 짧은 네트워킹 제공
- AZ 간의 모든 트래픽은 암호화

## VPC 
- __(서브넷, 인터넷 게이트웨이, VPC 엔드포인트, NACL, Security Group, Edge)__
- AWS 클라우드 내 논리적으로 격리된 가상 네트워크
- VPC를 이용하면 사용자가 정의한 가상 네트워크로 AWS 리소스를 시작 가능.
- AWS의 확장 가능한 인프라를 사용한다는 이점과 함께 고객의 자체 데이터 센터에서 운영하는 기존 네트워크와 매우 유사
- 리전당 VPC 5, VPC당 서브넷 200 (증가 요청 가능)
- VPC
  - 사용자의 AWS 계정 전용 가상 네트워크
- 서브넷
  - VPC의 IP 주소 범위
- IGW (인터넷 게이트웨이)
  - VPC의 리소스와 인터넷 간의 통신을 활성화하기 위해 VPC에 연결하는 게이트웨이, 수평 확장되고 가용성이 높은 중복 VPC 구성 요소
  - 인터넷 라우팅 가능 트래픽에 대한 VPC 라우팅테이블에 대상을 제공하고, 퍼블릭 IPv4 주소가 할당된 인스턴스에 대해 NAT를 수행하는 두 가지 목적
- VPC 엔드포인트
  - IGW, NAT, Private Link로 구동하는 지원되는 AWS 서비스 및 VPC 엔드포인트 서비스에 비공개로 연결할 수 있게 함
  - VPC의 인스턴스는 서비스의 리소스와 통신하는 데 퍼블릭 IP 주소를 필요로 하지 않는다 
  - S3에 연결할 경우 고정된 IP가 없기 때문에 조직에서 아웃바운드 all 설정을 허용하지 않는 정책이 있을 경우 이때 적용하기에도 좋음
  - IAM을 통해 특정 S3 Bucket 액세스만 허용 가능 / S3에서도 endpoint에서만 오는 거만 허용하도록 적용 가능
  - AWS 서비스는 기본적으로 인터페이스 호출 방식
- NACL과 Security Group은 기본적으로 deny이므로 허용 정책을 별도 등록해줘야 함
- NACL
  - stateless, 기본적으로 인바운드/아웃바운드 모두 허용, 서브넷 단위 적용
  - 들어오는 모든걸 허용
  - 하나의 NACL로 여러 서브넷 적용 가능. 서브넷은 하나의 NACL만 가능
  - 허용 정책 / 
- Security Group
  - stateful - 인바운드 허용되면 아웃바운드 자동, 인스턴스 단위
  - 모든 걸 block
  - 최소 권한 원칙 준수
  - 허용 규칙 지정 가능하나 거부 규칙 지정 불가
- Route 53 (DNS)
  - 최종 사용자를 인터넷 애플리케이션으로 라우팅하는 안정적이고 비용 효율적인 방법
  - 높은 가용성과 확장성이 뛰어난 클라우드 DNS 웹 서비스
  - DNS + 모니터링 + L4 + GSLB (Global Server Load Balancing) 기능 제공
- CloudFront (CDN)
  - 빠르고 고도로 안전하며 프로그래밍 가능한 콘텐츠 전송 네트워크
  - 짧은 지연 시간과 빠른 전송 속도로 데이터, 동영상, 애플리케이션 및 API를 전 세계 고객에게 안전하게 전송하는 고속 콘텐츠 전송 네트워크 서비스
  - CloudFront는 네트워크 및 애플리케이션 계층 DDoS 공격을 비롯해 여러 유형의 공격으로부터 보호하기 위해 Shield, WAF 및 Route53과 완벽하게 통합되어 필드 수준 암호화 및 HTTPS 지원을 포함한 대부분의 고급 보안 기능을 제공
  - 엣지 네트워킹 위치에 함께 상주하며, AWS 네트워크 백본을 통해 전 세계적으로 확장 및 연결 - 이를 통해 사용자에게 보다 안전하고 뛰어난 성능과 가용성을 보장하는 경험을 지원 가능
  - 고도로 확장 가능하며 전 세계에 분산되어 있음. CloudFront 네트워크에는 225개 이상의 PoP(Point of Presence)가 있음. 이는 AWS 백본으로 연결되어 최종 사용자에게 매우 짧은 지연 시간 성능과 높은 가용성을 제공
  - 엣지보안
    - CloudFront 배포는 Shiled standard에서 웹 사이트 또는 애플리케이션을 대상으로 자주 발생하는 네트워크 및 전송 계층 DDoS 공격에 대해 기본적으로 보호
    - 보다 복잡한 공격으로부터 보호하기 위해 Shiled, WAF와 통합하여 유연한 계층별 보안 경계 추가
  - 비용 효율성
    - ACM(AWS Certificate Manager)에서 사용자 지정 TLS 인증서 무료 제공
    - 선불 요금이 없는 간편한 종량 과금제와 최대 30%까지 비용을 추가로 절감할 수 있는 Saving Bundle을 비롯한 맞춤형 요금 옵션 제공
    - 최소 트래픽 약정(월 10TB 이상)으로 맞춤형 요금을 이용하면 더 큰폭 할인
  - 엣지컴퓨팅
    - Lambda@Edge : SPA 호스팅 등

## EC2
- 안전하고 크기 조정이 가능한 컴퓨팅 용량을 클라우드에서 제공하는 웹 서비스. 확장 가능한 컴퓨팅 용량을 제공하고 EC2를 사용하면 하드웨어에 선투자할 필요가 없어 더 빠르게 애플리케이션을 개발하고 배포할 수 있음
- 전 세계 24개 리전 및 77개 가용 영역
- Savings Plan, Reserved Instance 등으로 최대 72%까지 요금 할인 제공
- EC2 Spot : 뺏길 수 있으나, 배치 등과 같은 비 상시 서비스 용도로 좋음
- 크기 조정이 가능한 컴퓨팅 파워 (AutoScaling)
- AMI를 통해 시작 구성 이미지로 동일 구성 인스턴스를 시작 가능

## EBS
- 대규모 처리량과 트랜잭션 집약적인 워크로드 모두를 지원하기 위해 EC2에서 사용하도록 설계된 사용하기 쉬운 고성능 블록 스토리지 서비스
- 필요할 때 비용 효과적인 스토리지를 사용할 수 있도록 중요한 애플리케이션을 중단하지 않고도 볼륨 유형을 변경하거나, 성능을 조정하거나 볼륨 크기를 늘릴 수 있음
- AZ 내에서 복제되며, 페타바이트의 데이터로 쉽게 확장 가능. 데이터의 지리적 보호와 비즈니스 연속성을 보장하면서, 자동화된 수명 주기 정책을 기반으로 EBS 스냅샷을 사용하여 S3에서 볼륨 백업 가능
- EBS Snapshot은 간단하고 암호화 데이터 보호 솔루션으로 EBS 볼륨, Boot 볼륨을 보호한다. 스냅샷은 증분식 백업이어서 마지막 스냅샷 이후 병경된 디바이스의 블록만 저장된다. 그러면 스냅샷을 만드는 데 필요한 시간이 최소화되며 데이터를 복제하지 않으므로 스토리지 비용 절약 -> S3에 저장
- 백업한 스냅샷으로 다른 리전이나 계정으로 복구가 가능하다

## ELB
- 네트워크 트래픽 분산을 통한 애플리케이션 확장성 개선
- 들어오는 애플리케이션 트래픽을 EC2, 컨테이너, IP주소, Lambda 함수 등 여러 대상에 자동으로 분산
- 단일 가용 영역 또는 여러 가용 영역에서 다양한 애플리케이션 부하를 처리 가능하고, 애플리케이션의 내결함성에 필요한 고가용성, 자동 조정, 강력한 보안 기능을 갖추고 있음
- health check 가능
- Security Group을 통한 보안, SSL 복호화 기능 제공
- CloudWatch를 통한 지표, 로깅, 요청 추적을 통해 애플리케이션 상태와 성능을 실시간으로 모니터링 가능
- ALB, NLB, GLB는 대상을 대상 그룹에 등록하고 트래픽을 대상 그룹에 라우팅, CLB는 로드 밸런서에 인스턴스를 등록
- NLB, GLB는 기본적으로 교차 영역 로드 밸런싱이 비활성화 된다.
- 외부용 NLB, 내부용 ALB - https://aws.amazon.com/ko/blogs/korea/using-static-ip-addresses-for-application-load-balancers/
- ALB (Application)
  - HTTP 및 HTTPS 트래픽의 로드밸런싱, L7에 대한 부하분산을 지원
  - URL이나 hostname을 통해서 분배 가능. MSA, Container based에 적합
  - stickines : 쿠키 등을 통해서 같은 유저가 같은 instance로 붙는 것을 지원
  - path-based 라우팅 지원
  - 라운드로빈 알고리즘
- NLB (Network)
  - L4 부하 분산
  - 고정IP
  - pre-warm 불필요 -> Reverse Proxy 방식의 ALB/CLB와 달리 리턴 트래픽은 ELB가 관여하지 않는 형태
  - Response 시에 인스턴스는 직접 Client에게 패킷을 전달
  - 흐름해시 알고리즘
- GLB (Gateway)
- CLB (Classic)
  - 기존 애플리케이션이 EC2-Classic 네트워크 내에 구축되어 있는 경우
  - L4, L7 모두에서 작동하는 범용성
  - health 체크할 때 /index.html로 체크

## AutoScaling
- 수요에 맞춰 여러 리소스의 규모를 조정해주는 서비스
- Auto Scaling을 통해 애플리케이션의 로드를 처리할 수 있는 정확한 수의 인스턴스를 보유할 수 있도록 보장 가능.
- 내결함성 향상, 가용성 향상, 비용 관리 개선
- 한 AZ가 비정상적일 때 영향을 받지 않은 AZ에서 새 인스턴스 시작. 비정상이 복구되었을 때 모든 AZ에 걸쳐 인스턴스를 자동으로 고르게 배포
- AutoScaling Group을 설정하여 여러 AZ에 걸쳐 조정 가능
- ELB 그룹에 Auto Scaling 그룹을 연결하여 단일접점 역할 로드밸런싱 가능
- 정상 호스트 수가 허용된 것보다 낮을 경우 CloudWatch 경보 생성 가능
- AMI/템플릿을 가지고 AutoScaling 정책 시작
- HTTPS 또는 UDP 리스너와 같은 보안 프로토콜을 사용하여 리스너를 생성해야 하는 경우 ELB 콘솔을 사용하여 추가 리스너 생성해야 함
- 수동으로도 Auto Scaling 그룹 조정 가능 
- 동적 조정 (대상 추적 조정 / 단계 조정 / 단순 조정 / 예약 조정) - 증가는 반올림, 감소는 반내림
  - 여러 조정 정책을 적용할 수 있는데 동시에 적용되는 경우 확장 및 축소를 위한 최대 용량을 제공하는 정책이 선택. 하지만 주의해야함
  - 대상 추적 조정
    - 대상 추적 조정 정책을 auto scaling 그룹에 구성 -> 가용성 우선 시 -> 대상 추적 정책 중 하나라도 확장 허용할 경우 확장, 모둔 대상 추적 정책이 축소를 허용하는 경우에만 대상을 축소
    - 가능한 한 빨리 측정치에 비례하여 확장되고 서서히 축소
    - 지표 선택 가능. CPU, Netwrok In/Out, CountPerTarget(완료된 요청수)
    - ALBRequestCountPerTarget 지표가 없으면 Auto Scaling 그룹을 축소X
  - 단계 조정
    - 2단계 조절 10% 증/감
    - 3단계 조절 30% 증/감
- 조정 옵션
  - 일정 기반
  - 온디맨드 기반
  - 예측 조정
- 인스턴스 웜업 시간 조절 가능 -> scale out/in 을 필요 이상으로 하지 않을 수 있음, 웜업 기간 동안은 현재 용량의 부분으로 간주X
- 스케일링 쿨다운
  - 인스턴스 늘리기 전에 충분한 시간을 갖는 것
  - 다이나믹 스케일링엔 자동, 메뉴얼 스케일링에는 옵션
  - 스케쥴 스케일링에는 X
- 웜풀
  - 인스턴스가 대량의 데이터를 디스크에 써야 하기 때문에 부팅 시간이 예외적으로 긴 애플리케이션의 지연 시간을 줄이는 기능 제공 - 필요하지 않은 경우 불필요한 비용 발생
- 인스턴스 종료
  - 기본 정책 / 사용자 지정 종료 정책 - 축소 이벤트가 발생할 때 먼저 종료할 인스턴스를 제어 가능
  - 기본 정책
    - 온디맨드 인스턴스를 우선순위가 낮은 인스턴스 유형에서부터 점진적으로 이동
    - 가장 오래된 시작 템플릿 또는 구성을 사용하는 인스턴스 확인
    - 종료할 비보호 인스턴스가 여러 개 있는 경우, 다음 결제 시간에 가장 근접한 인스턴스가 무엇인지 확인
  - 사용자 지정
    - Default : 기본 종료 정책
    - AllocationStrategy : 선호하는 인스턴스 유형이 변경된 경우 유용
    - OldestLaunchTemplate : 가장 오래된 템플릿
    - OldestLaunchConfiguration : 가장 오래된 시작 구성
    - ClosestToNextInstanceHour : 다음 번 결제 시간에 가장 근접한 인스턴스
    - NewInstance : 그룹에서 가장 새로운 인스턴스
    - OldInstance : 가장 오래된 인스턴스
- 가영 영역 간 재분배 : 가용성을 높이기 위해 가용 영역 간에 인스턴스를 선제적으로 고르게 분배하기 위해 발생
- 보안
  - 각 계정마다 MFA 인증
  - SSL 사용하여 AWS 통신
  - CloudTrail로 API 및 사용자 활동 로깅 설정
  - 고객 마슽터 키(CMK)를 사용하여 클라우드에 저장된 Amazon EBS 볼륨데이터를 암호화하도록 KMS 그룹 구성 가능


## RDS
- 클릭 몇 번으로 클라우드에서 관계형 데이터베이스를 간편하게 설정, 운영 및 중단없이 확장할 수 있다. 완전 관리형.
- 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같은 시간 소모적인 관리 작업을 자동화하면서 비용 효율적이고 크기 조정 가능한 용량을 제공
- 필요할 때 자동화된 백업을 수행하거나 고유한 백업 스냅샷을 수동으로 만들 수 있음. 이러한 백업을 사용하여 데이터베이스 복원
- 기본 인스턴스 및 문제 발생 시 장애 조치를 수행할 수 있는 동기식 보조 인스턴스에서 가용성을 높일 수 있음. 읽기 전용 복제본을 사용하여 읽기 조정을 높일 수 있음
- IAM을 통한 접근 제어
- 보안 그룹을 통한 액세스 제어
- Shell 액세스 제공 X
- 스토리지 유형(EBS - 필요한 스토리지 용량에 따라 자동으로 데이터를 여러 EBS 볼륨에 나누어 저장하여 성능 강화)
  - 범용SSD, 프로비저닝된 IOPS, 마그네틱
- CPU, 용량, 메모리 등을 독립적으로 확장 가능
- 다중 AZ 배포에서 자동으로 서로 다른 가용 영역에 동기식 예비 복제본을 프로비저닝하고 유지
  - DB 인스턴스 스냅샷 캡쳐 -> 다른 가용 영역으로 복원 -> 새 인스턴스와 동기 복제 설정
  - 다중 AZ를 활성화한 경우 DB 인스턴스에 중단이 발생하면 RDS는 자동으로 다른 가용 영역에 있는 예비 복제본으로 전환 -> 일반적으로 1~2분 -> 장애 조치에 대한 내용을 Noti. 가능
- 온디맨드 DB 인스턴스 / 예약 DB 인스턴스로 비용 효율 가능
- 권장 옵션 - 다중 AZ 장애 조치 / 프로비저닝된IOPS 스토리지 / 삭제 방지 활성화
- RDS Proxy 연결 설정 가능 - 연결을 풀링하고 공유하여 확장성 향상.
- 복제된 read replica는 읽기 전용 복제본이고 업데이트는 비동기식으로 복사됨
  - 복제본을 독립 실행 DB 인스턴스로 승격해야 하는 몇 가지 이유
  - DDL 작업 실행 : 인덱스를 생성하거나 리빌드 하는 등의 DDL 작업은 시간이 걸리 뿐 아니라 DB 인스턴스에 상당한 성능 저하 초래
  - 샤딩 : 무공유 아키텍처를 공유함으로써 대규모 DB를 다수의 소규모 DB로 분할하는 기술
  - 장애 복구 실행
  - autoScaling으로 용량 자동 관리
- 백업 데이터를 장기간 보존하려면 S3로 내보내는 것 좋다
- DB 인스턴스 첫번째 스냅샷은 전체 DB 인스턴스 데이터 포함. 동일한 DB 인스턴스의 후속 스냅샨은 증분식
- 공유 스냅샷을 다른 AWS 계정에 복사 가능 -> KMS에 대한 액세스 권한 / KMS 고객 마스터 키를 사용하여 암호화된 스냅샷 복사 가능
- SSL을 사용한 인스턴스 연결 암호화 / RDS 암호화를 사용한 스냅샷 보호 및 데이터 암호화 - 인스턴스 생선전 가능

## ElastiCache (Memcached)
- Redis 또는 Memcached와 호환되는 완전관리형 인메모리 데이터 스토어. 1밀리초 미만의 지연 시간으로 실시간 애플리케이션을 지원
- 클라우드에서 분산된 인 메모리 데이터 스토어 또는 캐시 환경을 손쉽게 설정, 관리 및 확장할 수 있는 웹 서비스. 확장 가능하고 비용 효율적인 고성능 캐싱 솔루션을 제공
- 확장성
  - 변동이 심한 애플리케이션 수요에 맞춰 스케일아웃, 스케일 인/업 됨.
  - 쓰기 및 메모리 크기 조정은 샤딩을 통해 지원. 읽기 크기 조정은 복제본에서 제공
-  완전 관리형
  -  하드웨어 프로비저닝, 소프트웨어 패치, 설정, 구성, 모니터링, 장애 복구 및 백업과 같은 관리 작업 수행 불필요
-  인메모리 키-밸류 스토어의 주된 목적은 지연 시간이 1밀리초 미만인 매우 빠른 속도와 저렴한 비용으로 데이터 복사본에 액세스할 수 있게 하는 것
-  유지관리 기간 설정 가능 - 기본 설정 없음 / 유지 관리 기간 지정
-  캐싱 전략
  -  지연 로딩 : 필요할 때에만 데이터를 캐시에 로드
  -  라이트-스루 : 데이터베이스에 데이터를 작성할 때마다 데이터를 추가하거나 캐시의 데이터를 업데이트
  -  TTL 추가 : 키가 만료될 때까지 시간을 지정
- Auto Discovery 설정 : 캐시 클러스터에서 노드 개수를 늘리면 새로운 노드가 자신을 등록.
- ElastiCache 노드 / 클러스터
- 분산형 메모리 캐싱 시스템
- 캐시의 성능과 수량을 유연하게 변경할 수 있다.


## Security Group
- 인스턴스에 대한 수신 및 발신 트래픽을 제어하는 가상 방화벽 역할
- VPC 시작할 때에도 보안그룹을 지정해야하는데, 인스턴스를 시작한 이후에는 해당 보안 그룹 변경 불가
- 보안 그룹 규칙은 항상 허용적 -> 액세스를 거부한느 규칙 생성 불가
- 프로토콜과 포트 번호를 기준으로 트래픽 필터링 가능
- 상태 저장 - 한쪽이 허용되면 다른 쪽은 규칙에 관계없이 허용됨
- 연결 추적 - 인스턴스가 송수신하는 트래픽에 대한 정보를 추적 (모든 트래픽에 대해 추적하지는 않음)
- 최대 허용 규칙 적용

## S3, CloudFront, VPC Endpoint
### S3 (Simsple Storage Service)
- 어디서나 원하는 양의 데이터를 저장하고 검색할 수 있도록 구축된 객체 스토리지
- 업계 최고의 확장성, 데이터 가용성 및 보안과 성능, 내구성을 제공하는 객체 스토리지 서비스
  - 99.999999999% 안정성
- 비용 효율적
  - glacier deep archive 압축 보관 - 비용 저렴하여 이익 -> 자동 백업 설정 가능
  - 액세스 패턴에 따라 더 저렴한 요금의 스토리지 클래스로 이동할 데이터 검색 가능. 이러한 이동을 실행하는 S3 라이프사이클 정책 구성 가능
  - 액세스 패턴이 변하거나 액세스 패턴을 알 수 없는 데이터는 액세스 패턴 변화에 따라 객체를 계층화하는 __intelligence tiering__ 에 저장하여 자동으로 비용 절약
- 퍼블릭 액세스 차단을 통해 보안 강화
- 다른 리전으로 복제 가능
- S3 Storage Lens는 객체 스토리지 사용 및 활동 추세에 대한 조직 전체의 가시성을 제공
- IAM, ACL, 쿼리 문자열 인증 등으로 보안 액세스 관리
- VPC 엔드포인트를 사용하여 S3 리소스에 연결
- 정적 웹 사이트 호스팅

### CloudFront
- 빠르고 고도로 안전하며 프로그래밍 가능한 콘텐츠 전송 네트워크 (CDN)
  - 사용자의 요청이 통과해야하는 네트워크의 수가 줄어들어 성능 향상
  - 파일의 사본이 전 세계 여러 엣지 로케이션에 유지(캐시)되므로 안정성과 가용성 향상
- 엣지 보안
  - 네트워크 및 애플리케이션 계층 DDoS 공격을 비롯해 여러 유형의 공격으로부터 보호하기 위해 Shield, WAF 및 Route 53과 완벽하게 통합되어 필드 수준 암호화 및 HTTPS 지원을 포함한 대부분의 고급 보안 기능 제공
  - https를 사용하여 전송 데이터 암호화
  - 저장 데이터 암호화 - 암호화된 EBS 볼륨 사용
  - S3에 대해 WAF 및 ACL 사용
- 엣지 네트워킹 위치에 함께 상주하며, AWS 네트워크 백본을 통해 전 세계적으로 확장 및 연결
  - 고도로 확장 가능하며 전 세계에 분산. 225개 이상의 PoP(Point of Presence)
- 비용 효율성
  - 선불 요금 없는 간편한 종량 과금제와 최대 30%까지 비용을 추가로 절감할 수 있는 CloudFront Security Savings Bundle을 비롯한 맞춤형 요금 옵션 제공
- S3를 오리진으로 사용 가능
- 캐시 적중률 개선
  - 객체를 캐싱하는 시간 지정
  - 쿼리 문자열 파라미터 기반 캐싱
  - 쿠키값 기반 캐싱
  - 요청 헤더 기반 캐싱
  - 압축 불필요 시 Accept-Encoding 헤더 제거
- 온디맨드 비디오 및 라이브 스트리밍 비디오 가능

### VPC EndPoint
- 인터넷 게이트웨이, NAT 디바이스 등이 없이도 AWS 서비스 및 VPC 엔드포인트 서비스에 비공개로 연결 가능
- VPC의 인스턴스는 서비스의 리소스와 통신하는 데 퍼블릭 IP주소 필요 X

## CloudWatch, KMS, IAM, CloudFormation
### CloudWatch
- AWS 및 온프레미스에서 AWS 리소스와 애플리케이션의 관찰 기능
- 수집
- 모니터링
  - 대시보드를 통한 통합된 운영 뷰
  - 이상 탐지
- 조치
  - AutoScaling
  - EKS, ECS 등에서 경보 제공 및 작업 자동화
- 분석
  - 지표에 대한 사용자 지정 작업
  - 로그 분석
- 규정 준수 및 보안
  - IAM과 통합되므로 데이터에 대한 액세스 권한이 있는 사용자 및 리소스 그리고 액세스 방법을 제어 가능

### KMS (Key Management Service)
- 데이터를 암호화 하거나 디지털 서명할 때 사용하는 키를 손쉽게 생성 및 제어
- 완전 관리형
- 애플리케이션의 데이터 암호화
- 저렴한 비용

### IAM (Identify and Access Management)
- AWS 서비스 및 리소스에 대한 액세스를 안전하게 관리

### CloudFormation
- Infrastructure as Code로 클라우드 프로비저닝 가속화
- IaC를 통해 손쉬운 방법으로 관련된 AWS 및 서드 파티 리소스 모음을 모델링하고, 일관된 방식으로 간단히 프로비저닝하고, 수명 주기 전반에 걸쳐 관리 가능

# 5 Pillars
- 운영 우수성
  - CloudFormation, AutoScale, CloudWatch (실패학습), EBS Snapshot (되돌릴 수 있는 백업)
- 보안성
  - IAM, VPC, NACL, SG, KMS를 통한 S3 안전 데이터, SSL, CloudFront (DDoS 대비 등)
- 안정성
  - AutoScale, EBS Snapshot
- 성능 효율성
  - ElastiCache, CloudFront, IaC를 위한 CloudFormation
- 비용 최적화
  - AWS Budget, CloudWatch 비용 알림, CloudFront, AutoScale, billing 페이지 등을 통해 상세 청구 항목 확인
  - 사용한 만큼만 지불

# CICD
## Code Pipeline
- 빠르고 안정적인 애플리케이션 및 인프라 업데이트를 위해 릴리스 파이프라인을 자동화 하는 데 도움이 되는 완전관리형 CICD 서비스

## Code Commit
- 안전한 Git 기반 리포지토리를 호스팅하는 완전관리형 소스제어 서비스

## Code Build
- 소스 코드를 컴파일하는 단계부터 테스트 실행 후 소프트웨어 패키지를 개발하여 배포하는 단계까지 마칠 수 있는 완전 관리형의 지속적 통합 서비스
- 작업을 완료하는 데 필요한 운영 체제, 프로그래밍 언어 런타임 및 빌드 도구가 포함된 미리 구성된 빌드 환경에서 빌드를 실행 및 테스트
- 코드를 실행하고 아티팩드를 S3에 저장. 또는 빌드 명령을 사용해 아티팩트 레포지토리에 업로드 가능

## Code Deploy
- 다양한 컴퓨팅 서비스에 대한 소프트웨어 배포를 자동화하는 완전관리형 배포 서비스
- 가동 중지 시간 최소화
- 하나의 애플리케이션을 여러 배포 그룹에 배포 가능
- ELB와 통합하여 배포 전략 선택 가능
  - 블루/그린 : 배포된 배포 그룹(대체 환경)에서 새 인스턴스로 트래픽이 라우팅되도록 한 다음, 이전 인스턴스에서의 트래픽 차단
  - In-place : 배포 대상 인스턴스로 라우팅되지 않도록 하고 해당 인스턴스 배포 완료되면 다시 트래픽을 처리하게 함


# MSA
## ECR
- 안전하고 확장 가능하며 신뢰할 수 있는 AWS 관리형 컨테이너 이미지 레지스트리 서비스
- 도커 이미지와 같은 이미지들을 저장하는 공간

## ECS
- 클러스터에서 컨테이너를 쉽게 실행, 중지 및 관리할 수 있게 해주는 확장성과 속도가 뛰어난 컨테이너 관리 서비스
- 한 리전 내의 여러 AZ에 걸쳐 고가용성 방식으로 컨테이너 실행을 간소화하는 리전 서비스
- VPC에서 ECS 클러스터를 생성할 수 있고 클러스터가 실행되면 실행할 컨테이너 이미지를 정의하는 작업 정의 생성 가능. 작업 정의는 작업을 실행하거나 서비스를 생성하는 데 사용
- 컨테이너 이미지는 ECR에서 저장 및 가져온다
- Task definition : 애플리케이션이 ECS에서 실행되도록 준비. 컨테이너 몇 개 뜰 건지
- task and schedueling
  - task는 클러스터 내 작업 정의를 인스턴스화 하는 것. task definition 만들고 클러스터에서 실행할 작업 수 지정 가능
  - 스케줄러는 클러스터 내에 작업을 배치하는 일
- cluster :  task 또는 서비스의 논리적 그룹. task가 fargate에서 실행되면 클러스터 리소스도 fargate에서 관리
- 컨테이너 에이전트
  - 클러스터의 각 컨테이너 인스턴스에서 실행. 리소스의 현재 실행 중인 작업 및 리소스 사용률에 대한 정보를 ECS로 전송. 요청을 수신할 때마다 작업을 시작하고 중지
- https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/Welcome.html

## Fargate
- 컨테이너에 적합한 서버리스 컴퓨팅 엔진으로, ECS 및 EKS에서 모두 작동
- 애플리케이션을 빌드하는 데 보다 쉽게 초점을 맞출 수 있도록 해줌
- 인프라가 아닌 애플리케이션을 배포/관리
  - 서버를 프로비저닝하고 관리할 필요가 없어 애플리케이션 별로 리소스를 지정하고 관련 비용을 지불할 수 있으며, 계획적으로 애플리케이션을 격리함으로써 보안 성능을 향상시킬 수 있음
- 요금 옵션이 유연한 적정 규모의 리소스
  - 컨테이너용으로 지정하는 리소스 요구 사항과 밀접하게 일치하도록 컴퓨팅을 시작하고 규모를 조정
  - 추가 서버를 사용하기 위해 과도하게 프로비저닝하거나 관련 비용을 지불하지 않아도 됨.
  - EC2 인스턴스와 마찬가지로 요금 옵션 이용 가능
- 계획적으로 안전하게 격리
  - 개별 ECS task 또는 EKS 팟은 각각 자체 전용 커널 런타임 환경에서 실행되고, 다른 태스크 및 팟과 cpu, 메모리, 스토리지, 네트워크 리소스를 공유 X. 이를 통해 워크로드가 격리되고 각 태스크 또는 팟의 보안 성능 향상
- 높은 수준의 애플리케이션 가시성
  - CloudWatch를 비롯한 AWS 서비스와의 내장된 통합 기능을 통해 가시성을 즉시 확보 가능

## fargate 배포
- ECS에서 fargate 태스크 만들고 (ECR 필요)
- ALB 필요 (타겟 그룹)
- ECS 클러스터로 가서 task definition -> Cluster (배포 방법 선택) -> 컨테이너 생성됨
- code build에서 ECR에 이미지 업로드
- code deploy가 fargate의 task definition을 업데이트 (블루/그린 등 선택 가능)

## SQS
- 마이크로 서비스, 분산 시스테 및 서버리스 애플리케이션을 쉽게 분리하고 확장할 수 있도록 지원하는 완전 관리형 메시지 대기열 서비스
- 관리 오버헤드 제거
- 메시지를 안정적으로 전달
  - 애플리케이션 구성 요소를 분리할 수 있으므로 이러한 구성 요소가 독립적으로 실행되고 장애가 발생하여 시스템의 전체 내결함성 향상
- 민감한 데이터를 안전하게 유지
  - 서버 측 암호화를 통해 각 메시지 본문을 암호화하여 애플리케이션 간에 민감한 데이터 교환 가능 (KMS 통해)
- 탄력적이고 비용 효율적으로 확장
  - 필요에 따라 동적으로 확장. 애플리케이션에 따라 탄력적으로 확장. 프로비저닝 걱정X
  - 대기열당 메시지 수에 제한X, 표준 대기열은 거의 무제한의 처리량 제공

## SNS
- 애플리케이션 간 및 애플리케이션과 사용자 간 통신 모두를 위한 완전 관리형 메시징 서비스
- 완전 관리형 pub/sub 메시징, SMS, 이메일 및 모바일 푸시 알림
- 메시지 순서 지정 및 중복 제거로 정확성 보장
- 메시지 필터링을 통한 아키텍처 간소화

## Aurora
- 클라우드를 위해 구축된 MySQL 및 PostgreSQL 호환 관계형 데이터 베이스. 1/10 비용으로 상용 데이터베이스 수준의 성능 및 가용성 제공
- MySQL DB보다 최대 5배 빠르고, PostgreSQL DB보다 3배 빠름
- 내결함성을 갖춘 자가 복구 분산 스토리지 시스템으로, 데이터베이스 인스턴스당 최대 128TB까지 자동으로 확장
- 지연 시간이 짧은 읽기 전용 복제본 최대 15개, 특정 시점으로 복구, S3로 지속적 백업, 3개 AZ에 걸친 복제를 통해 뛰어난 성능과 가용성 제공
- AutoScaling : 스토리지 용량을 늘려야 하는 경우 DB 볼륨 크기를 자동으로 늘린다 128TB 까지 10GB 단위 증가
- 지연 시간이 짧은 읽기 전용 복제본
  - AZ간 공유되는 shared storage, Auto Scaling을 지원하므로 read replica를 자동으로 추가하고 제거


# Blah Blah
- IAM 최소 권한
- NLB, ALB, Static 차이
- AutoScale, CloudWatch, ELB 연계한 그림
- CloudTrail : API에 대해 이루어진 호출 추적 -> S3 버킷의 로그 파일에 정보 저장

# RFC 1918
- 10.0.0.0 - 10.255.255.255 (10/8 prefix)
- 172.16.0.0 - 172.31.255.255 (172.16/12 prefix)
- 192.168.0.0 - 192.168.255.255 (192.168/16 prefix)

# VPC 서브넷 권고사항
- /16 VPC (64k address)
- /24 subnets (251 address, 5개 예약 - 관리 목적)

# AWS Wavelength
- 개발자가 모바일 디바이스 및 최종 사용자 측에서 발생하는 지연 시간이 10밀리초 미만인 애플리케이션을 제작 가능
- AWS 개발자는 이동 통신 사업자 데이터 센터 내의 5G 네트워크 엣지에 AWS 컴퓨팅 및 스토리지 서비스를 포함하는 AWS 인프라 배포 환경인 Wavelength Zone에 애플리케이션을 배포하고 해당 리전의 다양한 AWS 서비스에 원할하게 액세스 가능
- 게임 및 라이브 동영상 스트리밍, 엣지의 기계 학습 추론, 증강 및 가상 현실 등 10밀리초 미만의 지연 시간이 요구되는 애플리케이션 제공 가능
