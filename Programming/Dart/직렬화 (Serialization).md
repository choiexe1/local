#dart #serialization

## 직렬화 (Serialization)
- 데이터 구조나 객체 상태를 저장하고 나중에 재구성할 수 있는 포맷으로 변환하는 과정

```mermaid
graph LR

subgraph "External API"
	Data
end

subgraph Server
	Instance
end

JSON

Data --> JSON -->  Instance

Instance --> JSON --> Data 
```

- `jsonDecode()`, `jsonEncode()` 등의 내장 함수 존재

---
[[파일 조작, 다양한 파일 형식]]