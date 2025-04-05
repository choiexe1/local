#mcp

## 개요

요즘 MCP(Model Context Protocol)이 세계적으로 뜨거운 관심을 받고 있습니다. 저도 그렇게까지는 관심이 없었는데 유튜브 알고리즘으로 하도 나오다보니 구축을 시도하게 됐습니다.

MCP는 아직 초기 단계라 가장 정확한 정보는 [modelcontextprotocol.io](https://modelcontextprotocol.io/introduction)에서 찾아 볼 수 있습니다.

다음은 MCP 공식 소개 글입니다.

MCP는 애플리케이션이 LLM에 컨텍스트를 제공하는 방식을 표준화하는 개방형 프로토콜입니다. MCP는 AI 애플리케이션을 위한 USB-C 포트와 같다고 생각하면 됩니다. 

USB-C가 장치를 다양한 주변기기 및 액세서리에 연결하는 표준화된 방법을 제공하는 것처럼, MCP는 AI 모델을 다양한 데이터 소스 및 도구에 연결하는 표준화된 방법을 제공합니다.

## mcp_dart 패키지

감사하게도 [leehack](https://linktr.ee/leehack)이라는 개발자께서 Dart로 MCP를 구현해놓은 패키지가 있어서 Dart로 간단히 구축이 가능했습니다.

패키지 사용법은 [mcp_dart](https://pub.dev/packages/mcp_dart)에서 확인하실 수 있습니다.

### 구현

먼저 패키지를 설치하고, 