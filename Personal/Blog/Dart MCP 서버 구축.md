#mcp

## 개요

요즘 MCP(Model Context Protocol)이 핫하네요. 저도 그렇게까지는 관심이 없었는데 유튜브 알고리즘이 계속 나오다보니 도대체 얼마나 좋은지 확인하기 위해 구축을 시도하게 됐는데, 결과가 매우 놀라웠습니다.

이제는 LLM을 통해서 내 로컬 파일에 접근할 수 있고, API를 연동해놓고 LLM에게 시키기만 하면 작동된다니!

MCP에 대한 정확한 정보는 [modelcontextprotocol.io](https://modelcontextprotocol.io/introduction)에서 찾아 볼 수 있습니다.

### MCP 공식 소개 글

MCP는 애플리케이션이 LLM에 컨텍스트를 제공하는 방식을 표준화하는 개방형 프로토콜입니다. MCP는 AI 애플리케이션을 위한 USB-C 포트와 같다고 생각하면 됩니다. 

USB-C가 장치를 다양한 주변기기 및 액세서리에 연결하는 표준화된 방법을 제공하는 것처럼, MCP는 AI 모델을 다양한 데이터 소스 및 도구에 연결하는 표준화된 방법을 제공합니다.

## 3-Tier Architecture

![[Pasted image 20250405172414.png]]


![[Pasted image 20250405172704.png]]


MCP는 호스트, 클라이언트, 서버로 구성된 3티어 아키텍쳐

### 호스트
- MCP 시스템의 최상위 계층, 사용자와 직접 상호작용하는 어플리케이션
	- Claude Desktop
### 클라이언트
- 클라이언트는 호스트와 서버 사이의 중개자 역할을 수행
	- Claude Desktop을 사용하면 클라이언트 역할도 수행

### 서버
- 실제 기능과 리소스를 제공
	- 다음에 설명할 mcp_dart를 사용하면 간단히 구현 가능

## mcp_dart 패키지

감사하게도 [leehack](https://linktr.ee/leehack)이라는 한국인 개발자께서 Dart로 MCP를 구현해놓은 패키지가 있어서 Dart로 간단히 구축이 가능했습니다.

패키지 사용법은 [mcp_dart](https://pub.dev/packages/mcp_dart)에서 확인하실 수 있습니다.

## 소스 코드
```dart
void main(List<String> arguments) async {
  setUp();

  await server.connect(StdioServerTransport());
}

final McpServer server = McpServer(
  Implementation(name: "jay-server", version: "1.0.0"),
  options: ServerOptions(
    capabilities: ServerCapabilities(tools: ServerCapabilitiesTools()),
  ),
);

void setUp() {
  server.tool(
    "readFile",
    description: '파일 읽기',
    inputSchemaProperties: {
      'path': {'type': 'string'},
    },
    callback: ({args, extra}) async {
      final path = args!['path'];

      File file = File(path);

      if (await file.exists()) {
        String content = await file.readAsString();
        return CallToolResult(content: [TextContent(text: content)]);
      } else {
        return CallToolResult(content: [TextContent(text: 'File not found')]);
      }
    },
  );

  server.tool(
    "todo",
    description: '투두 정보 조회',
    inputSchemaProperties: {
      'id': {'type': 'number'},
      'completed': {'type': 'boolean'},
    },
    callback: ({args, extra}) async {
      Uri uri = Uri.parse('https://jsonplaceholder.typicode.com/todos');
      http.Response data = await http.get(uri);
      List<dynamic> todos = jsonDecode(data.body);

      List<Map<String, dynamic>> notCompleted =
          todos
              .cast<Map<String, dynamic>>()
              .where((e) => e['completed'] == false)
              .toList();

      List<Map<String, dynamic>> completed =
          todos
              .cast<Map<String, dynamic>>()
              .where((e) => e['completed'] == true)
              .toList();

      return CallToolResult(
        content: [TextContent(text: todos.toString())],
        meta: {
          'completed': completed.length,
          'not_completed': notCompleted.length,
          'total': todos.length,
          'completedPercentage': (todos.length / todos.length) * 100,
        },
      );
    },
  );
}
```

저도 간단히 찍먹 용도로 작성해보았는데, 클로드 데스크탑(Claude Desktop)을 통해 실행해볼 수 있었습니다. 기능은 아주 잘 작동하구요.. 단 두 가지 문제점이 있었습니다.

1. 도구 이름은 한글을 지원하지 않는다.
	- "todo"와 같은 영어는 잘 지원되는데 "투두 조회"와 같은 한국어는 지원되지 않습니다. 아마 Claude Desktop에서 에러가 나는 것으로 보아 Claude Desktop의 문제 같긴합니다.
2. 최악인건 클로드를 사용하면 토큰 부족 문제로 원하는만큼 사용할 수 없습니다..


## 결론

MCP로 인해서 LLM의 능력이 대폭 확장된 것 같습니다. 지금 사용중인 IDE인 Zed의 `Agentic Editing` 기능 베타테스터 신청을 해놓았는데 얼른 MCP를 연결해 사용해보고 싶네요.
