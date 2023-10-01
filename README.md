# 0. \[WHS\] 20230923 오프라인 -AWS 취약점 분석 -과제
안녕하세요 저는 화이트 햇 스쿨 1기 박여웅 교육생입니다.

지난 AWS 취약점 분석 오프라인 수업에서 내주셨던 과제인, 실제 도커 환경에서 Provisoning 실습하기를

한번 진행해 보겠습니다.

# 1-0. Slave 컨테이너 생성
시작에 앞서 docker에서 slave 라는 ~~불쌍한(?)~~ 이름을 가진 컨테이너를 하나 만들어줍니다.
![](/src/Pasted_image_20231001083804.png)
이 컨테이너가 Provisoning 당할 친구 입니다!

계속해서 진행 해보죠

# 1-1. slave 기본 세팅
계속해서 캡처 뜨기가 불편해서 걍 이 코드로 어떤일이 일어났는지 보여드리겠습니다.

본론으로 앞서, `apt update && apt install -y sudo net-tools vim`을 이용해 필요한 패키지를 받아오겠습니다.

두번째로 `adduser slave`를 사용해 slave 라는 이름의 유저 하나를 생성시켜줍니다.

마지막으로 slave 라는 계정에 sudo라고 불리는 흔히 윈도우에서 '관리자 권한으로 실행' 과 같은 기능을 하는 명령어의 권한을 넣어주기 위해 `visudo -f etc/sudoers` 라는 파일을 수정해 주겠습니다.
![](/src/Pasted_image_20231001085507.png)
위와 같이 `test ALL=(ALL:ALL) NOPASSWD: ALL`라고 적어주어 패스워드 없이 sudo 명령어를 사용할 수 있게 되었습니다. 마지막으로 `:wq!`를 입력하면 저장 후 나가기 처리가 됩니다.

이처럼 Slave 컨테이너에서의 작업은 마무리가 되었습니다 :)

# 2-0. Master 컨테이너 생성
좀전에 Slave 컨테이너를 생성해준것처럼 똑같이 이름만 'Master' 로 설정하고 똑같이 우분투 환경으로 진행하겠습니다.
![](/src/Pasted_image_20231001090126.png)
이와같이 Master 컨테이너를 생성해주었습니다 :)

# 2-1. master 기본 셋팅
좀전에 slave 작업에서 했던 패키지 업뎃과 요구 패키지 설치까지 한번에 처리할 명령어인

`apt update && apt install -y net-tools vim ansible`

를 사용해 진행하였습니다.

다음으로, `ssh-keygen -t rsa`를 통해 slave 컨테이너에 접속시에 사용되는 RSA 인증서를 생성해 slave 컨테이너 측에 이 인증서를 사용해 접속할 수 있도록 밑작업(?)을 진행해 주겠습니다.

`ssh-keygen -t rsa`를 통해 RSA ssh 인증서를 만들어주고, `ssh-copy-id -i ~/.ssh/id_rsa.pub slave@172.17.0.2` 를 사용해 발급한 인증서를 서버에 접속시 사용할 수 있게 만들어 주겠습니다.

# 2-2. 문제 발생
흠... 왜 인지 정상적으로 접근이 되지 않았습니다. 생각을 해보니 slave 측에 ssh 패키지를 설치를 안했더라고요? 

그래서 `apt install ssh -y`로 바로 설치를 진행했습니다.

설치 후엔 `service ssh restart`를 이용해 ssh를 실행시킵니다. 이젠 연결이 가능해집니다.

# 2-3. ansible 기본 셋팅
ansible 패키지를 사용해 Provisioning을 실행하도록 ansible의 기본 셋팅을 진행하겠습니다.

앞서, ansible의 기본 설정을 위해 `mkdir /etc/ansible`을 통해 slave 접속 정보를 넣어놓을 디렉터리를 생성해줍니다.

두번째로 `vim /etc/ansible/hosts` 를 이용해 slave 접속 정보를 아래와 같이 입력해줍니다.
![](/src/Pasted_image_20231001104434.png)
저장 후 종료해주고, `ansible clients -m ping`를 통해서 정상적인 통신이 진행되는지 확인합니다.

# 3-0. Provisoning
방금 보낸 ping 결과가 success가 떴다면 정상적인 통신 진행이 확인 되었습니다.

마지막 단계로 이제 Provisoning을 하기위한 실행 코드를 아래와 같은것으로 사용하겠습니다.
![](/src/Pasted_image_20231001105444.png)
apt update와 루트 디렉터리에 hahaha 라는 디렉터리가 생성이 되는 것을 수행합니다.

그럼 한번 `ansible-playbook security.yml`을 통해 실행해보도록 하겠습니다.
![](/src/Pasted_image_20231001120517.png)
위와 같이 정상적으로 실행 되는 것을 볼 수 있습니다.
![](/src/Pasted_image_20231001120853.png)
실제 slave 콘솔을 확인하면 hahaha 라는 디렉터리가 루트 디렉터리에 생성 된것을 볼 수 있습니다.

# 4. 마치며
위 활동을 진행 하면서 AWS 같은 IaaS 환경에서 시큐리티 작업은 어떤식으로 진행되고 어떤식으로 프로비저닝 하는지 등 에 대해 배웠고, 저는 아직 온 프레미스 환경밖에 사용하지 않아 실제로 수업에서 나온 기술을 사용할 기회는 적지만 나중에 리얼 월드 에서 사용할 IaaS환경을 간접적으로나마 체험할 수 있어서 좋았습니다.

이상으로 WHS 1기 6반 박여웅 교육생의 보고서 였습니다. 감사합니다 :)