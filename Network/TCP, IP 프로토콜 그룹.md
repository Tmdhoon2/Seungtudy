## TCP/IP 프로토콜 그룹

<img width = "70%" src = "https://blog.kakaocdn.net/dn/lvby2/btryYonh5jg/ZnSRgg7kczIZm6QXtP8m3K/img.png"/>

- 각 장치는 5개의 계층을 가지고 있음
  - **응용 - Application**
  - **전송 - Transport**
  - **네트워크 - Network**
  - **데이터 링크 - Data link**
  - **물리 - Physical**

- 데이터는 **응용 계층**에서 생성되어 여러 계층을 따라 **물리 계층**에서 다른 호스트로 전달됨
- 데이터는 **물리 계층**에서 받아져 계층들을 타고 **응용 계층**으로 전달됨

### 물리층

- 데이터 단위로 **비트 - bit** 사용
- 링크들을 따라 데이터를 전송하는 책임이 있음
- TCP / IP 계층에서 가장 **최하위**

### 데이터 링크층

- 라우터에서 데이터를 전송할 링크를 결정하면 **데이터 그램**을 받아 해당 링크로 전송하는 책임이 있음

### 네트워크층

- 발신지 호스트와 목적지 호스트 사이의 연결을 생성
- 발신지 -> 목적지 사이의 라우터들은 패킷을 위해 최선의 경로를 선택

### 전송층

- 응용층으로부터 데이터를 받아 **전송층 패킷**으로 캡슐화 함
- 목적지 호스트의 전송층에 **논리적 연결**을 통해 전송함
- 응용층에 서비스를 제공하기 위한 책임을 가짐

### 응용층

- 응용층 사이에서 메시지들을 교환함
- 프로세스간 통신은 응용층에서 이루어짐