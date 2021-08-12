# Learning Cypher
Neo4j Movies Project Tutorial을 따라가는 Cypher 사용 방법 정리

## 1.What is Cypher?
- Cypher is graph query language for Neo4j Database.

```cypher
# 2000년 이후 개봉한 영화 5개
Match (m:Movie) where m.released > 2000 RETURN m limit 5

# 2005년 이후 개봉한 영화 5개
Match (m:Movie) where m.released > 2005 RETURN m limit 5

# 2005년 이후 개봉한 영화 개수
Match (m:Movie) where m.released > 2005 RETURN count(m)
```

### 2.Nodes and Relationsships
- Nodes(노드)
  - 엔티티
- Relationship(관계)
  - 두 노드는 Relationship으로 연결된다.
  - cypher에서는 대괄호로 묶어서 표기한다.
    - [variable: type of relationship]
  - 두 노드는 하나 이상의 relationship으로 연결될 수 있다.

```cypher
# 2010년 이후 개봉한 영화를 감독한 사람을 함께 조회
MATCH (p:Person)-[d:DIRECTED]-(m:Movie) where m.released > 2010 RETURN p,d,m

# 2010년 이후 개방한 영화를 연기한 사람을 함께 조회
MATCH (p:Person)-[d:ACTED_IN]-(m:Movie) where m.released > 2010 RETURN p,d,m
```

### 3.Labels
- Labels
  - 노드나 관계의 이름/식별자
  - 특정한 노드/관계 타입을 지정할 때 사용
```cypher
# 20개이하의 Person node 조회
MATCH (p:Person) RETURN p limit 20

# 20개 이하의 아무 node 조회
MATCH (n) RETURN n limit 20
```

### 4. Properties
- Properties
  - Node or relationship의 속성값
  - name-value 쌍

```cypher
# movie의 제목과 개봉년도 조회
MATCH (m:Movie) return m.title, m.released

# Person의 이름과 출생년도를 조회
MATCH (p:Person) return p.name, p.born
```

### 5.Create a Node
- Create
  - Clause
  - 노드/관계를 생성

```cypher
# name이 John Doe인 Person 노드를 생성
Create (p:Person {name: 'John Doe'}) RETURN p
```

### 6.Finding Nodes with Match and Where Clause
- Match
  - Clause
  - Neo4j database의 데이터를 얻는 방법
  - [where 절에서 지원하는 조건](https://neo4j.com/docs/cypher-manual/4.1/clauses/where/)

```cypher
# 두 cypher는 같다
Match (p:Person {name: 'Tom Hanks'}) RETURN p
Match (p:Person) where p.name = "Tom Hanks" RETURN p

# 영화 제목이 'Cloud Atlas'인 것 조회
Match (m:Movie) where m.title = "Cloud Atlas" RETURN m
# 또는
MATCH (m:Movie {title: "Cloud Atlas"}) return m

# 영화 개봉년도가 2010에서 2015년 사이인 것 조회
Match (m:Movie) where m.released > 2010 and m.released < 2015 RETURN m
```

### 7.Merge Clause
- Merge
  - clause
  - 존재하는 노드들을 서로 연결하거나 새로운 노드를 만들어서 연결시킨다
  - 즉, create와 match 절을 합친 형태
  - ON MATCH SET : 일치하는 노드가 있는 경우
  - ON CREATE SET : 노드가 없는 경우


```cypher
# 이름이 'John Doe' 사람을 조회하고
# 조회된 사람이 있다면, 마지막 로그인 시점을 
# 조회된 사람이 없다면, 노드를 생성하고 생성시점 저장
MERGE (p:Person {name: 'John Doe'})
ON MATCH SET p.lastLoggedInAt = timestamp()저장
ON CREATE SET p.createdAt = timestamp()
RETURN p

# 제목이 'Greyhound'인 영화를 조회하고
# 조회된 영화가 있다면, 출시일자를 2020으로 설정하고 업데이트 시점을 저장
# 조회된 영화가 없다면, 노드를 생성하고 업데이트 시점을 저장
MERGE (m:Movie {title: 'Greyhound'})
ON CREATE SET m.released = "2020", m.lastUpdatedAt = timestamp()
ON MATCH SET m.lastUpdatedAt = timestamp()
RETURN m
```

### 8.Create a Relationship
- 노드와 노드를 연결하는 명령
 
```cypher
# 이름이 Tom Hanks인 사람과 제목이 Cloud Atlas인 영화를 WATCHED 관계로 연결한다.
MATCH (p:Person), (m:Movie)
WHERE p.name = "Tom Hanks" and m.title = "Cloud Atlas"
CREATE (p)-[w:WATCHED]->(m)
RETURN type(w)

# 이름이 jaesugang인 사람과 제목이 Cloud Atlas인 영화를 WATCHED 관계로 연결한다.
MATCH (p:Person), (m:Movie)
WHERE p.name = "jaesugang" and m.title = "Cloud Atlas"
CREATE (p)-[w:WATCHED]->(m)
RETURN type(w)
```

### 9.Relationship Types
- relationship은 2가지 종류가 있다.
  - incoming
  - outgoing

```cypher
# person 노드가 movie를 ACTED_IN하는 노드 조회
MATCH (p:Person)-[r:ACTED_IN]->(m:Movie)
RETURN p,r,m

# person 노드가 movie를 ACTED_IN하는 노드 조회
MATCH (p:Person)-[r:ACTED_IN]-(m:Movie)
RETURN p,r,m

# person 노드가 movie를 REVIEWED하는 노드 조회
MATCH (p:Person)-[r:REVIEWED]->(m:Movie)
RETURN p,r,m
```

### 10.Advance Cypher queries

1. 누가 영화 'Cloud Atlas'를 감독했는가?
```cypher
MATCH (m:Movie {title: 'Cloud Atlas'}) <- [d:DIRECTED]-(p:Person) return p.name
```

2. Tom Hanks와 같이 연기한 모든 사람
```cypher
MATCH (tom:Person {name: 'Tom Hanks'})-[:ACTED_IN]->(:Movie)<-[:ACTED_IN]-(p:Person) return p
```

3. Cloud Atals와 관계있는 모든 사람
```cypher
# [relatedTo] 는 모든 관계를 의미한다
Match (p:Person)-[relatedTo]-(m:Movie {title: "Cloud Atlas"}) return p.name, type(relatedTo)
```

4. Kevin Bacon으로부터 3 hops 이내에 영화와 배우
```cypher
MATCH (p:Person {name: 'Kevin Bacon'})-[*1..3]-(hollywood) return DISTINCT p, hollywood
```