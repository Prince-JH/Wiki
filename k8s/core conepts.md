![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3d7382fb-15f9-49a3-9d6f-31b94e37889b/Untitled.png)

## k8s Architecture
- 노드 세트로 구성
    - 마스터 노드
        - 컨트롤 하는 역할
        - docker compose와 유사하는 기능을 하는듯
        - ETCD
            - key-value DB
    - 워커 노드
        - 특정 주어진 기능을 함
