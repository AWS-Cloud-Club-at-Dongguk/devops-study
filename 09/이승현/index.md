# 웹 크롤러 설계
> 검색 엔진에서 널리 쓰는 기술로, 웹에 새로 올라오거나 갱신된 콘텐츠를 찾아내는 것이 주된 목적

- 검색 엔진 인덱싱
- 웹 아카이빙
- 웹 마이닝
- 웹 모니터링

## 1단계 문제 이해 및 설계 범위 확정
- 규모 확장성
- 안정성
- 예절
- 확장성

### 개략적 규모 추정

- 매달 10억 개의 웹 페이지 다운로드
- QPS = 10억
- 최대 QPS = 2 X QPS = 800
- 웹 페이지의 크기 평균 = 500k 가정
- 10억 페이지 500k = 500TB/월
- 5년 보관한다고 가정 시 30PB의 저장 용량 필요

## 2단계 개략적 설계안 제시 및 동의 구하기
<img width="632" height="436" alt="image" src="https://github.com/user-attachments/assets/cdfa4cfc-580f-4e63-b13b-af1055ad4bb3" />


- 시작 URL 집합: 크롤링을 시작하는 출발점
- 미방문 URL 저장소(URL Frontier): 다운로드할 URL을 저장하는 큐 역할
- HTML 다운로더: 인터넷에서 웹 페이지를 내려받는 컴포넌트
- 도메인 이름 변환기(DNS Resolver): URL을 IP 주소로 변환
- 콘텐츠 파서: 웹 페이지를 파싱하여 올바른 형식인지 확인
- 중복 콘텐츠 감지: 이미 수집한 콘텐츠인지 확인하여 중복 저장을 방지
- URL 추출기: HTML 내의 링크를 골라내어 URL 저장소에 전달
  <img width="558" height="303" alt="image" src="https://github.com/user-attachments/assets/8b917a60-d51c-4fb1-93b7-659148794d97" />
- URL 필터: 특정 유형이나 확장자를 가진 URL을 제외
- 웹 크롤러 작업 흐름
  <img width="604" height="431" alt="image" src="https://github.com/user-attachments/assets/081af052-59ba-4330-b2bb-42e2700c5463" />



## 3단계 상세 설계
> 중요한 컴포넌트와 그 구현 기술들

- DFS vs BFS: 웹은 깊이가 깊으므로 주로 BFS(너비 우선 탐색)를 사용하되, 특정 서버에 요청이 몰리지 않도록 설계해야 함
  - 요청 링크들을 병렬로 처리하게 된다면 수많은 요청으로 과부하에 걸림 -> 보통 'impolite' 크롤러로 간주
  - 표준적 BFS 알고리즘은 URL 간에 우선순위를 두지 않음
  <img width="528" height="396" alt="image" src="https://github.com/user-attachments/assets/0df6cb63-e2a4-42c7-8666-a5064ea2ef81" />

- URL Frontier의 역할: '예의'를 지키기 위해 동일 웹사이트에는 일정 간격을 두고 방문하며, 우선순위에 따라 크롤링 순서를 결정
- 성능 최적화
  1. 분산 크롤링
     <img width="385" height="318" alt="image" src="https://github.com/user-attachments/assets/22bacf4b-2199-4796-b7c6-d877f2ecad2a" />
  3. DNS 결과 캐싱
  4. HTML 다운로더의 비동기 I/O 사용 등을 통해 속도를 높임
- 안정성: 안정적인 체크포인트를 두어 장애 발생 시 복구할 수 있도록 하며, 예외 처리를 철저히 함
- 확장성: 새로운 형태의 콘텐츠(이미지, 영상 등)를 쉽게 처리할 수 있도록 모듈화
  <img width="728" height="423" alt="image" src="https://github.com/user-attachments/assets/3aa7b4b8-a140-4f56-9802-f448abdbc014" />


## 4단계 마무리
> 추가 고려 사항들

- 서버 측 렌더링(JS): 자바스크립트로 생성되는 페이지를 위한 동적 렌더링 지원 여부.
- 스팸 방지: 질 낮은 페이지나 스팸 사이트 걸러내기.
- 데이터베이스 다중화 및 샤딩: 수집된 방대한 데이터를 효율적
