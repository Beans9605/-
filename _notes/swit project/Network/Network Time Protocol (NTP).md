[[Kubernetes Cluster]]
NTP는 인터넷에서 라우터 및 기타 하드웨어 디바이스의 클럭을 동기화하는 데 널리 사용되는 프로토콜. 기본 NTP 서버는 UTC(Coordinated Universal Time)로 직접 추적할 수 있는 레퍼런스 클럭에 동기화됨. 레퍼런스 클럭에는 GPS 수신기 및 전화 모뎀 서비스가 포함되며, NTP 정확성 기대치는 환경 애플리케이션 요구사항에 따라 달라짐.

NTP server에 맞춰서 기준시간을 정하고 하위 Client들이 동기화하는 방식