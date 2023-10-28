# 2강 ElasticSearch의 개념

### ElasticSearch 란?

- 루씬(Lucene : 자바 라이브러리) 기반의 오픈소스 검색 엔진
- JSON 기반의 문서를 저장하고 검색할 수 있으며 분석 작업도 가능

### ElasticSearch의 특징

1. 준실시간 검색 시스템 : 실시간이라고 생각될 만큼 색인된 데이터가 빠르게 검색
2. 고가용성을 위한 클러스터 구성 : 두 대 이상의 노드로 클러스터를 설정하여 높은 수준의 안정성을 달성하고 부하 분산이 가능합니다.
3. 동적 스키마 생성 : 입력될 데이터들에 대해 미리 스키마를 정의하지 않아도 동적으로 스키마 생성이 가능
4. Rest API 기반의 인터페이스 : 비교적 사용을 위한 진입 장벽이 낮습니다.

---

# 3강 클러스터와 노드

### 클러스터란?

- 컴퓨터 클러스터는 **여러 대의 컴퓨터들이 연결되어 하나의 시스템처럼** 동작하는 컴퓨터들의 집합
- ElasticSearch도 **여러대의 노드들이 각자의 역할을 바탕으로 하나의 시스템처**럼 동작
    
    → **그래서 어떤 노드에 어떤 요청을 해도 동일한 응답을 줍니다.**
    
    → 각각의 노드가 본연의 역할에 충실 할 수 있도록 구성하는 것이 중요
    
- 클러스터의 성능이 부족하다면 노드를 늘려서 대응할 수 있습니다.
    
     → but 노드를 늘린다고 모두 성능 개선X
    

### 노드의 종류

1. 마스터 노드 : 클러스터 상태 관리 및 메타데이터 관리
2. 데이터 노드 : 문서 색인 및 검색 요청 처리
3. 코디네이팅 노드 : 검색 요청 처리
4. 인제스트 노드 : 색인되는 문서의 데이터 전처리

### 마스터 노드

1. 마스터 노드 : 지금 클러스터에서 마스터 노드의 역할을 하고 있는 노드
2. 마스터 후보 노드 : 마스터 노드에 문제가 생겼을 때 마스터 노드가 될 수 있는 노드

# 4강 인덱스와 샤드

### ElasticSearch와 RDBMS 비교

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/2ad52309-6182-4268-86f6-1cc4430c8753)

### 인덱스란?

- 문서가 저장되는 논리적인 공간 (문서가 저장하기 위해서는 반드시 존재해야함)

**인덱스를 설계**하는 것이 엘라스틱서치를 사용하기 위해서 고려해야하는 첫 단계입니다.

### 예시)도서관 자료 검색 시스템

1. library → library 라는 이름의 인덱스를 만들어서 모든 자료를 여기에 저장
2. book, magazine, multimedia,etc → 각 자료별로 인덱스를 따로 만들어서 저장

### 인덱스 설계에 따라 달라지는 문서의 구조 및 검색 쿼리가 달라짐!

```jsx
{
"type" : "book",
"title" : "elasticSearch essental",
"author" : "alden"
}
```

```jsx
GET /libary/_search

{
	"query" : {
		"bool" : {
			"must" : {
				{"match" : {"type": "book"}},
				{"match" : {"author": "alden"}}	
			}
		}
	}
}
```

**사용 패턴과 문서의 특성에 따라 설계해야 한다!!**

### 인덱스 설계

1. 하나의 인덱스를 사용할때
    1. 장점 : 관리해야 할 인덱스의 수가 적어 관리 리소스가 적게 발생
    2. 단점 : 쿼리와 문서의 구조가 복잡해 질 수 있다.
2. 여러개의 인덱스로 나눌 때
    1. 장점 : 각각의 경우에 최적화된 쿼리와 문서구조를 사용할 수 있다.
    2. 단점 : 관리해야 할 인덱스의 수가 많아 관리 리소스가 발생할 수 있다.

### 샤드란 ?

- 인덱스에 색인되는 문서가 저장되는 공간
- 하나의 인덱스는 반드시 하나 이상의 샤드를 가진다
![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/5733f6f4-612a-4fbc-96ca-6e69cbb51aad)


### 샤드의 종류

1. 프라이머리 샤드 : 문서가 저장되는 원본샤드, **색인과 검색 성능**에 모두 영향을 줌
2. 레플리카 샤드 : 프라이머리 샤드의 복제 샤드, **검색 성능**에 영향을 줌

**프라이머리 샤드에 문제가 생기면 레플리카 샤드가 프라이머리 샤드로 승격**

### 샤드 설정

```jsx
PUT /library/_settings

{
	"index" : {
		"number_of_shards" : 3,
		"number_of_replicas" : 1	
	}
}
```

프라이머리 샤드: 3개

레플리카 샤드( number_of_shards X number_of_shards) : 3개 

인덱스의 총 샤드 개수: 6개

```jsx
PUT /library/_settings

{
	"index" : {
		"number_of_shards" : 5,
		"number_of_shards" : 2	
	}
}
```

프라이머리 샤드: 5개

레플리카 샤드( number_of_shards X number_of_shards) : 10개 

인덱스의 총 샤드 개수: 15개

### 샤드 라우팅

- 문서들은 0~2번까지의 샤드에 고르게 저장

⇒ 샤드의 개수가 바뀐다면 문서가 저장되는 규칙이 완전히 바뀌게 됩니다.

**Routing Rule = (문서의 ID) % (샤드의 개수)**

⇒ 인덱스를 생성 할 때 프라이머리 샤드의 개수를 설정 하는 건 매우 중요

⇒ 왜냐면 인덱스 생성 이후 프라이머리 샤드 개수 변경은 불가능 하기 때문이다

**number_of_shards 의 기본값은 1** 

⇒ 하지만 기본값을 그대로 사용하면 성능에 큰 영향을 미침

### 인덱스 템플릿이란?

- nginx-logs- 로 시작하는 모든 인덱스는 프라이머리 샤드 3개ㅏ , 레플리카 샤드 6개로 생성
- ex. nginx-logs-2022.11.01, nginx-logs-2022.11.02,….
- 인덱스 템플릿을 통해 인덱스 생성 시 샤드 개수를 미리 설정 가능

---

# 5강 매핑

### 매핑(mapping)이란?

- 문서의 구조를 나타내는 정보

### 매핑의 종류

1. 동적매핑 :  처음 색인되는 문서를 바탕으로 매핑정보를 ElasticSearch가 동적으로 생성 
    1. 장점: 어떤 문서가 색인될지 스키마에 미리 정의하지 않아도 됩니다.
    2. 동적 매핑에 의해 매핑정보가 생성된 후에는 타입이 안맞을 경우 파싱 에러가 발생
    - 참고 사진
        ![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/1cb28551-f536-4137-8c78-f7fd66ab84dd)
        ![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/6f0cca9b-ef18-4f9d-84d7-8b6788fe2191)

        
        갑자기 rating에 “”하면 float 인데 string 넣어서 오류
        
2. 정적매핑 : 문서의 매핑 정보를 미리 정의
    1. 장점 : 중요한것만 미리 정적으로 매핑해놓고, 안중요한건 매핑 안해도됨. 유연하게 사용
    - 참고 사진
        ![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/ce64a2e2-a08d-4ac7-b337-72931d11b838)


정적 매핑은 언제 도움이 될까??

1. 문서들이 필드들이 가지는 값에 따라 타입을 지정해 줄 필요가 있을 때
2. 불필요한 색인이 발생하지 않게 하기 위해 필요

---
# 6강 색인 과정 이해하기

### 색인이란?

- 문서를 분석하고 저장하는 과정

### 색인 과정

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/700fd85a-c577-40a6-8556-613286f41f7d)

inverted index → 검색과 연관되어있음

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4f2aec43-2087-4c65-9297-af9eb9a257ee/cb4a66d6-c8e0-4e24-ada6-52179e0e774f/Untitled.png)

프라이머리 샤드가 1개이기 때문에 색인이 하나의 데이터 노드에서만 일어남

→ 데이터노드가 3대지만 색인에 있어서는 사실상 1개만 이용

### → 클러스터로서의 이점을 전혀 살리지 못하는 상황!

→ 그래서 적절한 수의 샤드 개수를 설정하는 것이 성능에 큰 영향을 미친다!

→ 최대한 많은 노드들이 색인에 참여하고 있는지 그걸 가장 먼저 확인해야 함

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4f2aec43-2087-4c65-9297-af9eb9a257ee/f21bd3a3-5948-4f30-833b-743423c65609/Untitled.png)

```jsx
PUT /library/_settings

{
	"index" : {
		"number_of_shards" : 3,
		"number_of_replicas" : 1	
	}
}
```

프라이머리 샤드가 3개이기 때문에 3대의 데이터 노드 모두 색인에 참여

아주 이상적인 상황 !

→ 데이터 노드가 하나 더 추가된다면?

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/b150fcbe-ba1b-4d36-b3bc-a4331cdd6796)


샤드의 개수가 고르게 분배되지 않아 용량 불균형이 일어날 수 있습니다.

### 결론

**적절한 샤드 개수를 배치하는 것이 중요!**

---

# 7강 검색 과정
![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/8f7027c8-7514-4aab-91e9-c38104e75215)

### inverted index 란?

문자열을 분석한 결과를 저장하고 있는 구조체

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/4f2aec43-2087-4c65-9297-af9eb9a257ee/528e6d81-2f22-4092-9019-9e98b881e59b/Untitled.png)

### 애널라이저(Analyzer)

- 문자열을 분석해서 inverted index 구성을 위한 토큰을 만들어 내는 과정
![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/5725b408-55a1-4fca-a87c-ca512d9a8700)

- Character filter(특수문자 제거)  tokenizer (공백 나누기)  token filter (소문자 만들기)→애널라이저
- 한글을 검색하는 경우에서 많이 사용

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/dbc69385-2614-41f1-9563-336208af9e08)

                                                                                                                       결과 

- 검색 : linux kernel ⇒ linux, kernel 두개의 토큰 생성

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/9184b2d3-1440-401e-a6d4-f37051b4f4f6)

⇒ analyzer를 적용해서 생성한 토큰을 바탕으로 inverted index을 구성

⇒ 검색어로부터 생성된 토큰들을 inverted index 에서 찾는 과정이 **검색**

### 색인 과정 → 프라이머리 샤드

### 검색요청 → 프라이머리 샤드와 레플리카 샤드 모두 처리 가능

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/a17d8132-013e-4279-9daf-57902ad1e75f)

- number_of_shards : 2
- number_of_replicas : 1

색인 성능은 충분한데 **검색 성능을 높이고 싶으면 number_of_replicas 를 늘리자** ! 

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/e8835337-f044-4fb2-934c-e20258115a96)

- 검색 성능에 문제가 있다면 클러스터로서의 이점을 살리고 있는지를 먼저 확인해보기
    - number_of_shards : 2
    - number_of_replicas : 1
    - 데이터 노드를 아무리 늘린다해도 검색 성능을 높이는 것이 아니라 레플리카 수를 늘려야함!

### 요약

1. analyzer를 적용해서 생성한 토큰을 바탕으로 inverted index을 구성
2. 검색어로부터 생성된 토큰들을 inverted index 에서 찾는 과정이 **검색**
3. 검색요청 → 프라이머리 샤드와 레플리카 샤드 모두 처리 가능
4. 색인과 검색 모두 적절한 샤드의 수가 성능을 결정하며, 적절한 샤드의 수가 클러스터로서의 이점을 활용하는지 결정
5. 엘라스틱서치는 클러스터로 구성되기 때문에 모든 노드가 색인과 검색을 처리할 수 있도록 구성하는 것이 중요

---

# 8강 text vs keyword

### 공통점

문자열을 나타내기 위한 타입

### 차이점

- text : 전문검색(Full-text-search)을 위해 토큰이 생성
- keyword: Exact Matching을 위해 토큰이 생성

**analyze API 를 사용해서 어떤식으로 토큰이 만들어지는지 확인하는 것이 가장 정확**

1. text 검색


![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/fa984319-ea62-485e-b144-c7eca2354c70)


- analyzer : standard → text 검색
- text : “I am a boy”
- 결과 : I , am , a , boy 라는 4개의 토큰이 검색

1. keyword 검색

![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/630520c7-31b8-4b0f-a0ad-6676f895c387)


- analyzer : standard → text 검색
- text : “I am a boy”
- 결과 : “I am a boy” 라는 1개의 토큰이 검색

   ⇒ **Exact Matching** 을 위한 것
![image](https://github.com/yj0111/ElasticSearch-Study/assets/118320449/4c50c72f-7013-4eaa-a823-67ca116542ca)


⇒ **토큰이 어떻게 생성되는지 용도가 뭔지 에 따라서 다르게 생성 될 수 있음**

- keyword 타입이 색인 속도가 더 빠릅니다 **→ cpu를 더 적게 사용하기 때문**
- 문자열 필드가 동적 매핑이 되면 text 가 keyword 타입 두개가 모두 생성
- 굳이 토크나이징이 필요하지 않는 필드의 경우
    
    → 문자열의 특성에 따라서 text랑 keyword를 정적 매핑 해주면 성능 개선에 도움
    

### text로 정의되면 좋을 만한 필드

- 주소
- 이름
- 물품
- 상세 정보

### keyword로 정의되면 좋을 만한 필드

- 성별
- 물품 카테고리

⇒ 사용자 중심으로 생각을 해보면 좋을 듯
