# MCP LLMS-TXT 문서 서버

*본 저장소는 편의를 위해 LangChain의 [mcpdoc](https://github.com/langchain-ai/mcpdoc)을 한국어로 단순 번역한 저장소입니다.*

## 개요

[llms.txt](https://llmstxt.org/)는 LLM을 위한 웹사이트 인덱스로, 배경 정보, 가이드, 그리고 상세한 마크다운 파일에 대한 링크를 제공합니다. Cursor, Windsurf와 같은 IDE나 Claude Code/Desktop과 같은 애플리케이션은 `llms.txt`를 사용하여 작업에 필요한 컨텍스트를 검색할 수 있습니다. 하지만 이러한 앱들은 `llms.txt`와 같은 파일을 읽고 처리하기 위해 서로 다른 내장 도구를 사용합니다. 검색 프로세스는 불투명할 수 있으며, 도구 호출이나 반환된 컨텍스트를 감사할 방법이 항상 있는 것은 아닙니다.

[MCP](https://github.com/modelcontextprotocol)는 개발자들이 이러한 애플리케이션에서 사용하는 도구를 *완전히 제어*할 수 있는 방법을 제공합니다. 여기서는 MCP 호스트 애플리케이션(예: Cursor, Windsurf, Claude Code/Desktop)에 (1) 사용자 정의 `llms.txt` 파일 목록과 (2) 제공된 `llms.txt` 파일 내의 URL을 읽을 수 있는 간단한 fetch_docs 도구를 제공하는 [오픈소스 MCP 서버](https://github.com/modelcontextprotocol)를 만듭니다. 이를 통해 사용자는 각 도구 호출과 반환된 컨텍스트를 감사할 수 있습니다.


<img src="https://github.com/user-attachments/assets/736f8f55-833d-4200-b833-5fca01a09e1b" width="60%">

## llms-txt

langgraph와 langchain의 llms.txt 파일은 다음에서 찾을 수 있습니다:

| 라이브러리          | llms.txt                                                                                                   |
|------------------|------------------------------------------------------------------------------------------------------------|
| LangGraph Python | [https://langchain-ai.github.io/langgraph/llms.txt](https://langchain-ai.github.io/langgraph/llms.txt)     |
| LangGraph JS     | [https://langchain-ai.github.io/langgraphjs/llms.txt](https://langchain-ai.github.io/langgraphjs/llms.txt) |
| LangChain Python | [https://python.langchain.com/llms.txt](https://python.langchain.com/llms.txt)                             |
| LangChain JS     | [https://js.langchain.com/llms.txt](https://js.langchain.com/llms.txt)                                     |

## 시작하기

#### uv 설치
* `uv`를 설치하는 다른 방법은 [공식 uv 문서](https://docs.astral.sh/uv/getting-started/installation/#installation-methods)를 참조하세요.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### 사용할 `llms.txt` 파일 선택
* 예를 들어, [여기](https://langchain-ai.github.io/langgraph/llms.txt)에 LangGraph `llms.txt` 파일이 있습니다.

> **참고: 보안 및 도메인 접근 제어**
> 
> 보안상의 이유로 mcpdoc는 엄격한 도메인 접근 제어를 구현합니다:
> 
> 1. **원격 llms.txt 파일**: 원격 llms.txt URL(예: `https://langchain-ai.github.io/langgraph/llms.txt`)을 지정하면, mcpdoc는 자동으로 해당 특정 도메인(`langchain-ai.github.io`)만 허용된 도메인 목록에 추가합니다. 이는 도구가 해당 도메인의 URL에서만 문서를 가져올 수 있다는 의미입니다.
> 
> 2. **로컬 llms.txt 파일**: 로컬 파일을 사용할 때는 어떤 도메인도 자동으로 허용 목록에 추가되지 않습니다. `--allowed-domains` 매개변수를 사용하여 허용할 도메인을 명시적으로 지정해야 합니다.
> 
> 3. **추가 도메인 추가**: 자동으로 포함된 도메인 외에 다른 도메인에서 가져오기를 허용하려면:
>    - `--allowed-domains domain1.com domain2.com`을 사용하여 특정 도메인 추가
>    - `--allowed-domains '*'`를 사용하여 모든 도메인 허용 (주의해서 사용)
> 
> 이 보안 조치는 사용자가 명시적으로 승인하지 않은 도메인에 대한 무단 접근을 방지하여, 신뢰할 수 있는 소스에서만 문서를 검색할 수 있도록 보장합니다.

#### (선택사항) 선택한 `llms.txt` 파일로 MCP 서버를 로컬에서 테스트:
```bash
uvx --from mcpdoc mcpdoc \
    --urls "LangGraph:https://langchain-ai.github.io/langgraph/llms.txt" "LangChain:https://python.langchain.com/llms.txt" \
    --transport sse \
    --port 8082 \
    --host localhost
```

* 다음 주소에서 실행됩니다: http://localhost:8082

![Screenshot 2025-03-18 at 3 29 30 PM](https://github.com/user-attachments/assets/24a3d483-cd7a-4c7e-a4f7-893df70e888f)

* [MCP inspector](https://modelcontextprotocol.io/docs/tools/inspector)를 실행하고 실행 중인 서버에 연결합니다:
```bash
npx @modelcontextprotocol/inspector
```

![Screenshot 2025-03-18 at 3 30 30 PM](https://github.com/user-attachments/assets/14645d57-1b52-4a5e-abfe-8e7756772704)

* 여기서 `tool` 호출을 테스트할 수 있습니다.

#### Cursor에 연결

* `Cursor 설정`과 `MCP` 탭을 엽니다.
* `~/.cursor/mcp.json` 파일이 열립니다.

![Screenshot 2025-03-19 at 11 01 31 AM](https://github.com/user-attachments/assets/3d1c8eb3-4d40-487f-8bad-3f9e660f770a)

* 다음 내용을 파일에 붙여넣습니다 (`langgraph-docs-mcp` 이름을 사용하고 LangGraph `llms.txt`에 링크합니다).

```
{
  "mcpServers": {
    "langgraph-docs-mcp": {
      "command": "uvx",
      "args": [
        "--from",
        "mcpdoc",
        "mcpdoc",
        "--urls",
        "LangGraph:https://langchain-ai.github.io/langgraph/llms.txt LangChain:https://python.langchain.com/llms.txt",
        "--transport",
        "stdio"
      ]
    }
  }
}
```

* `Cursor Settings/MCP` 탭에서 서버가 실행 중인지 확인합니다.
* 모범 사례는 Cursor Global (User) 규칙을 업데이트하는 것입니다.
* Cursor `Settings/Rules`을 열고 다음과 같이 `User Rules`을 업데이트합니다:

```
for ANY question about LangGraph, use the langgraph-docs-mcp server to help answer -- 
+ call list_doc_sources tool to get the available llms.txt file
+ call fetch_docs tool to read it
+ reflect on the urls in llms.txt 
+ reflect on the input question 
+ call fetch_docs on any urls relevant to the question
+ use this to answer the question
```

* `CMD+L` (Mac에서)로 채팅을 엽니다.
* `agent`가 선택되어 있는지 확인합니다.

![Screenshot 2025-03-18 at 1 56 54 PM](https://github.com/user-attachments/assets/0dd747d0-7ec0-43d2-b6ef-cdcf5a2a30bf)

그런 다음 다음과 같은 예시 프롬프트를 시도해보세요:
```
what are types of memory in LangGraph?
```

![Screenshot 2025-03-18 at 1 58 38 PM](https://github.com/user-attachments/assets/180966b5-ab03-4b78-8b5d-bab43f5954ed)

### Windsurf에 연결

* `CMD+L` (Mac에서)로 Cascade를 엽니다.
* `Configure MCP`를 클릭하여 설정 파일 `~/.codeium/windsurf/mcp_config.json`을 엽니다.
* 위에서 언급한 대로 `langgraph-docs-mcp`로 업데이트합니다.

![Screenshot 2025-03-19 at 11 02 52 AM](https://github.com/user-attachments/assets/d45b427c-1c1e-4602-820a-7161a310af24)

* `Windsurf Rules/Global rules`을 다음과 같이 업데이트합니다:

```
for ANY question about LangGraph, use the langgraph-docs-mcp server to help answer -- 
+ call list_doc_sources tool to get the available llms.txt file
+ call fetch_docs tool to read it
+ reflect on the urls in llms.txt 
+ reflect on the input question 
+ call fetch_docs on any urls relevant to the question
```

![Screenshot 2025-03-18 at 2 02 12 PM](https://github.com/user-attachments/assets/5a29bd6a-ad9a-4c4a-a4d5-262c914c5276)

그런 다음 예시 프롬프트를 시도하세요:
* 도구 호출을 수행하는 것을 확인할 수 있습니다.

![Screenshot 2025-03-18 at 2 03 07 PM](https://github.com/user-attachments/assets/0e24e1b2-dc94-4153-b4fa-495fd768125b)

### Claude Desktop에 연결

* `Settings/Developer`를 열어 `~/Library/Application\ Support/Claude/claude_desktop_config.json`을 업데이트합니다.
* 위에서 언급한 대로 `langgraph-docs-mcp`로 업데이트합니다.
* Claude Desktop 앱을 다시 시작합니다.

> [!Note]
> Claude Desktop에 MCPDoc 도구를 추가하려고 할 때 Python 버전 비호환성 문제가 발생하면, `uvx` 명령에서 `python` 실행 파일의 경로를 명시적으로 지정할 수 있습니다.
>
> <details>
> <summary>예시 구성</summary>
>
> ```
> {
>   "mcpServers": {
>     "langgraph-docs-mcp": {
>       "command": "uvx",
>       "args": [
>         "--python",
>         "/path/to/python",
>         "--from",
>         "mcpdoc",
>         "mcpdoc",
>         "--urls",
>         "LangGraph:https://langchain-ai.github.io/langgraph/llms.txt",
>         "--transport",
>         "stdio"
>       ]
>     }
>   }
> }
> ```
> </details>

> [!Note]
> 현재 (2025/3/21) Claude Desktop은 전역 규칙을 위한 `rules`를 지원하지 않는 것으로 보이므로, 프롬프트에 다음을 추가하세요.

```
<rules>
for ANY question about LangGraph, use the langgraph-docs-mcp server to help answer -- 
+ call list_doc_sources tool to get the available llms.txt file
+ call fetch_docs tool to read it
+ reflect on the urls in llms.txt 
+ reflect on the input question 
+ call fetch_docs on any urls relevant to the question
</rules>
```

![Screenshot 2025-03-18 at 2 05 54 PM](https://github.com/user-attachments/assets/228d96b6-8fb3-4385-8399-3e42fa08b128)

* 채팅 입력 하단 오른쪽에 도구가 표시됩니다.

![Screenshot 2025-03-18 at 2 05 39 PM](https://github.com/user-attachments/assets/71f3c507-91b2-4fa7-9bd1-ac9cbed73cfb)

그런 다음 예시 프롬프트를 시도하세요:

* 요청을 처리하는 동안 도구 호출 승인을 요청합니다.

![Screenshot 2025-03-18 at 2 06 54 PM](https://github.com/user-attachments/assets/59b3a010-94fa-4a4d-b650-5cd449afeec0)

### Claude Code에 연결

* [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)를 설치한 후 터미널에서 다음 명령을 실행하여 프로젝트에 MCP 서버를 추가합니다:
```
claude mcp add-json langgraph-docs '{"type":"stdio","command":"uvx" ,"args":["--from", "mcpdoc", "mcpdoc", "--urls", "langgraph:https://langchain-ai.github.io/langgraph/llms.txt", "LangChain:https://python.langchain.com/llms.txt"]}' -s local
```
* `~/.claude.json`이 업데이트된 것을 확인할 수 있습니다.
* Claude Code를 실행하고 다음을 실행하여 도구를 확인하여 테스트합니다:
```
$ Claude
$ /mcp 
```

![Screenshot 2025-03-18 at 2 13 49 PM](https://github.com/user-attachments/assets/eb876a0e-27b4-480e-8c37-0f683f878616)

> [!Note]
> 현재 (2025/3/21) Claude Code는 전역 규칙을 위한 `rules`를 지원하지 않는 것으로 보이므로, 프롬프트에 다음을 추가하세요.

```
<rules>
for ANY question about LangGraph, use the langgraph-docs-mcp server to help answer -- 
+ call list_doc_sources tool to get the available llms.txt file
+ call fetch_docs tool to read it
+ reflect on the urls in llms.txt 
+ reflect on the input question 
+ call fetch_docs on any urls relevant to the question
</rules>
```

그런 다음 예시 프롬프트를 시도하세요:

* 도구 호출 승인을 요청합니다.

![Screenshot 2025-03-18 at 2 14 37 PM](https://github.com/user-attachments/assets/5b9a2938-ea69-4443-8d3b-09061faccad0)

## Command-line Interface (CLI)

`mcpdoc` 명령은 문서 서버를 시작하기 위한 간단한 CLI를 제공합니다.

세 가지 방법으로 문서 소스를 지정할 수 있으며, 이들을 조합할 수 있습니다:

1. YAML 설정 파일 사용:

* 본 저장소의 `sample_config.yaml` 파일에서 LangGraph Python 문서를 로드합니다.

```bash
mcpdoc --yaml sample_config.yaml
```

2. JSON 설정 파일 사용:

* 본 저장소의 `sample_config.json` 파일에서 LangGraph Python 문서를 로드합니다.

```bash
mcpdoc --json sample_config.json
```

3. llms.txt URL을 선택적 이름과 함께 직접 지정:

* URL은 일반 URL 또는 `name:url` 형식으로 선택적 이름과 함께 지정할 수 있습니다.
* `--urls` 매개변수를 여러 번 사용하여 여러 URL을 지정할 수 있습니다.
* 위에서 MCP 서버에 `llms.txt`를 로드한 방법입니다.

```bash
mcpdoc --urls LangGraph:https://langchain-ai.github.io/langgraph/llms.txt --urls LangChain:https://python.langchain.com/llms.txt
```

이러한 방법들을 조합하여 문서 소스를 병합할 수도 있습니다:

```bash
mcpdoc --yaml sample_config.yaml --json sample_config.json --urls LangGraph:https://langchain-ai.github.io/langgraph/llms.txt --urls LangChain:https://python.langchain.com/llms.txt
```

## 추가 옵션

- `--follow-redirects`: HTTP 리디렉션 따르기 (기본값은 False)
- `--timeout SECONDS`: HTTP 요청 타임아웃(초) (기본값은 10.0)

추가 옵션이 포함된 예시:

```bash
mcpdoc --yaml sample_config.yaml --follow-redirects --timeout 15
```

이는 15초 타임아웃으로 LangGraph Python 문서를 로드하고 필요한 경우 HTTP 리디렉션을 따릅니다.

## 구성 형식

YAML과 JSON 구성 파일 모두 문서 소스 목록을 포함해야 합니다.

각 소스는 `llms_txt` URL을 포함해야 하며 선택적으로 `name`을 포함할 수 있습니다:

### YAML 구성 예시 (sample_config.yaml)

```yaml
# mcp-mcpdoc 서버용 샘플 구성
# 각 항목은 llms_txt URL을 가져야 하며 선택적으로 이름을 가질 수 있습니다
- name: LangGraph Python
  llms_txt: https://langchain-ai.github.io/langgraph/llms.txt
```

### JSON 구성 예시 (sample_config.json)

```json
[
  {
    "name": "LangGraph Python",
    "llms_txt": "https://langchain-ai.github.io/langgraph/llms.txt"
  }
]
```

## 프로그래밍 방식 사용

```python
from mcpdoc.main import create_server

# 문서 소스로 서버 생성
server = create_server(
    [
        {
            "name": "LangGraph Python",
            "llms_txt": "https://langchain-ai.github.io/langgraph/llms.txt",
        },
        # 여러 문서 소스를 추가할 수 있습니다
        # {
        #     "name": "Another Documentation",
        #     "llms_txt": "https://example.com/llms.txt",
        # },
    ],
    follow_redirects=True,
    timeout=15.0,
)

# 서버 실행
server.run(transport="stdio")
```
