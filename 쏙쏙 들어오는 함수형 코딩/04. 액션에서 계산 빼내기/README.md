# MegaMart.com에 오신 것을 환영합니다
  ```
  const sh
  ```


# 랭체인
LangChain은 LLM(대규모 언어 모델)을 활용한 애플리케이션 개발을 가장 쉽게 시작할 수 있는 환경을 제공하는 동시에, 실제 서비스 환경에서 요구되는 유연성과 실용성을 갖추도록 설계된 프레임워크이다. 단순히 LLM을 호출하는 수준을 넘어, 프롬프트 구성, 외부 지식 연결(RAG), 도구 연동(Tool), 메모리 관리(Memory), 그리고 에이전트 실행(Agent orchestration)까지 LLM 기반 시스템을 구성하는 핵심 요소들을 조립 가능한 형태로 제공한다.

## 철학
  - 대규모 언어 모델(LLM)은 훌륭하고 강력한 새로운 기술이며 외부 데이터 소스와 결합할 때 훨씬 더 효과적이다.
  - 미래의 애플리케이션은 점점 더 에이전트 중심적인 형태이며, 아직 초기단계이다.
  - 에이전트 기반 애플리케이션의 프로토타입을 만드는 것은 쉽지만, 실제 운영 환경에 배포할 만큼 신뢰할 수 있는 에이전트를 구축하는 것은 여전히 ​​매우 어렵다.

## LLM 기반 애플리케이션 vs LLM 기반 Agent

### 1) LLM 기반 애플리케이션 (LLM App)
LLM 기반 애플리케이션은 특정 목적에 맞게 설계된 고정된 파이프라인을 가진다.

> 입력 -> LLM -> 출력

이 방식은 개발자가 미리 정해둔 구조에 따라 동작하며, 사용자 요청에 대해 예측 가능한 흐름으로 결과를 생성한다. 예를 들어 문서 요약, 분류, 번역, 검색 기반 QA 같은 기능은 대부분 LLM App 형태로 구현된다. 안정성과 재현성이 중요할수록 LLM App 구조가 적합하다.

### 2) LLM 기반 에이전트 (LLM Agent)
LLM 기반 에이전트는 LLM이 스스로 추론하고 행동하는 동적 시스템이다.

> 입력(목표) → 추론 → 도구 선택/사용 → 판단 → 반복 → 최종 출력

즉 “무엇을 할지”를 개발자가 전부 결정하지 않고, LLM이 목표 달성을 위해 필요한 단계를 스스로 선택한다. 이 과정에서 검색, DB 조회, 코드 실행, 외부 API 호출, 파일 생성 등 다양한 도구를 사용하며, 결과를 관찰하고 필요하면 계획을 수정하는 반복 루프를 수행한다.

에이전트는 복잡한 작업 자동화에 강력하지만, 그만큼 불확실성과 운영 난이도가 높기 때문에 신뢰성 확보를 위한 설계와 제어가 핵심이 된다.

## LLM 모델의 5가지 역할 (Capabilities)

LLM은 단순히 “텍스트를 생성하는 모델”을 넘어, 애플리케이션에서 다양한 기능적 역할을 수행할 수 있다. 아래 5가지는 LLM 시스템을 설계할 때 자주 사용하는 대표적인 역할이다.

### 1) 텍스트 생성 (Text Generation)

자연어를 생성하는 가장 기본 역할 (사람 처럼 텍스트를 생성)

### 2) 구조화된 답변 생성 (Structured Output)

사람이 읽는 텍스트가 아니라 JSON/스키마 형태로 결과를 생성

### 3) 멀티모달 (Multimodality)

텍스트뿐 아니라 이미지/음성/문서(PDF) 등 다양한 입력을 이해하고 처리

### 4) 추론 (Reasoning)

단순 생성이 아니라 문제 해결을 위한 판단/계획/논리적 사고를 수행

### 5) 도구 호출 (Tool Calling)

LLM이 스스로 외부 기능(API, DB, 검색, 코드 실행 등)을 호출하여 행동(Action) 을 수행


## Openai 맛보기

### 1) SDK 설치
```
pip install openai
```

### 2) 키 설정
```
import os

os.environ["OPENAI_API_KEY"] ="{오픈 API 키}"
```

### 3) 랭체인 시작전 간단한 예시 코드 호출 해보기
```
from openai import OpenAI
client = OpenAI()

response = client.responses.create(
    model="gpt-5-nano",
    input="Agent가 뭔가요?"
)

print(response.output_text)
```

### 4) 랭체인 시작전 간단한 예시 추론 적용해보기
 - https://platform.openai.com/docs/guides/reasoning#page-top
```
prompt = """
앵무새의 털 색상이 여러개인 이유가 뭐야?
"""

response = client.responses.create(
    model="gpt-5-nano",
    reasoning={"effort": "medium"},
    input=[
        {
            "role": "user", 
            "content": prompt
        }
    ]
)

print(response.output_text)

다양한 색이 보이는 이유는 주로 세 가지가 결합되어 생깁니다: 색소, 구조색, 그리고 유전·환경 요인.

- 색소에 의한 색깔
  - 멜라닌: 검정/회색/갈색 계통
  - 카로티노이드: 빨강/주황/노랑 계통(보통 먹이에서 얻은 색소가 영향을 줌)
  - 일부 앵무새는 독특한 색소를 자체적으로 만들거나 덧붙여 색을 낼 수 있습니다.
- 구조색에 의한 색깔
  - 깃털의 미세한 구조가 빛을 다르게 산란시켜 파란색이나 자주색, 보랏빛 같은 색을 만듭니다.
  - 초록은 보통 노란 색소와 파란 구조색의 조합으로 보이고, 파란색은 주로 구조색이 주도합니다.
- 유전적 요인과 환경 요인
  - 종마다 다른 유전 형질로 색깔이 달라지고, 같은 종이라도 나이, 계절, 식이에 따라 색의 선명도가 달라집니다.
  - 일부 애완용 앵무새는 유전적 색 변이(mutant color)로 파란색, 노란색, 얼룩무늬 등 다양한 색이 만들어지기도 합니다.
  - 짝짓기나 사회적 신호로도 색이 더 선명하게 보이도록 변하는 경우가 있습니다.

간단히 말하면, 앵무새의 색은 어떤 색소가 어떻게 들어 있고, 깃털의 미세구조가 빛을 어떻게 다루느냐에 좌우되며, 여기에 종별 차이나 먹이/나이 같은 요인이 더해져 여러 가지 색이 나타납니다.

특정 종에 대해 더 알고 싶으면 알려 주세요. 사진이나 종 이름이 있으면 그 색이 왜 그렇게 보이는지 더 구체적으로 설명해 드릴 수 있어요.
```

## 랭체인 시작해보기

### 1) 텍스트 생성
```
!pip install -U langchain
!pip install -U langchain-openai

import os
from langchain.chat_models import init_chat_model

os.environ["OPENAI_API_KEY"] ="{오픈 API KEY}"
# 모듈을 가져와 모델 설정
model = init_chat_model("gpt-5-nano") 

# invoke 메소드를 통해 답변(텍스트) 생성
response = model.invoke("안녕하세요. 당신은 누구입니까?")

# 콘텐츠, 입력 토큰, 아웃풋 토큰 등 출력 가능
AIMessage(content='안녕하세요! 저는 OpenAI가 만든 대화형 인공지능, ChatGPT입니다. GPT-4 아키텍처를 기반으로 질문에 답하고, 글을 쓰고, 아이디어를 정리하고, 코딩 도움까지 다양한 일을 도와드려요. 실제 사람은 아니고 컴퓨터 프로그램이에요. 제 지식은 2024년 6월까지의 정보에 한정되어 있고, 실시간 인터넷 검색은 필요시 확인이 필요합니다.\n\n필요한 게 있으면 말씀해 주세요. 예: 글 다듬기, 아이디어 브레인스토밍, 요약, 번역, 코드 문제 해결 등. 무엇을 도와드릴까요?', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 731, 'prompt_tokens': 15, 'total_tokens': 746, 'completion_tokens_details': {'accepted_prediction_tokens': 0, 'audio_tokens': 0, 'reasoning_tokens': 576, 'rejected_prediction_tokens': 0}, 'prompt_tokens_details': {'audio_tokens': 0, 'cached_tokens': 0}}, 'model_provider': 'openai', 'model_name': 'gpt-5-nano-2025-08-07', 'system_fingerprint': None, 'id': 'chatcmpl-', 'service_tier': 'default', 'finish_reason': 'stop', 'logprobs': None}, id='lc_run--', usage_metadata={'input_tokens': 15, 'output_tokens': 731, 'total_tokens': 746, 'input_token_details': {'audio': 0, 'cache_read': 0}, 'output_token_details': {'audio': 0, 'reasoning': 576}})
```

### 2) 텍스트 생성 - 파라메터 부여
- https://reference.langchain.com/python/langchain/models/?_gl=1*1e1x0z2*_gcl_au*MjgxMzg1MTQ0LjE3NjcxNjAxNjY.*_ga*MjUwODcwMTgwLjE3NjcxNjAxNjY.*_ga_47WX3HKKY2*czE3NjcxNjAxNjYkbzEkZzEkdDE3NjcxNjMwNTckajYwJGwwJGgw#langchain.chat_models.init_chat_model(configurable_fields)
```
model = init_chat_model(
    "gpt-5-nano",
    temperature = 0.7,
    max_tokens = 1000,
    timeout = 30,
    max_retries = 2
)
response = model.invoke("안녕하세요. 당신은 누구입니까?")
response.usage_metadata

{
  'input_tokens': 15,
  'output_tokens': 687,
  'total_tokens': 702,
  'input_token_details': {'audio': 0, 'cache_read': 0},
  'output_token_details': {'audio': 0, 'reasoning': 512}
}
```


### 2) 텍스트 생성 - 스트림
 - https://docs.langchain.com/oss/python/langchain/models#stream
```
for chunk in model.stream("안녕하세요. 당신은 누구입니까?"):
  print(chunk.text, end="")

안녕하세요! 저는 OpenAI가 만든 대화형 인공지능, 이름은 ChatGPT입니다. 다양한 주제에 대해 설명해 드리거나 글쓰기, 아이디어 제안, 코딩 도움 등 여러 방식으로 도와드릴 수 있습니다.

제 한계로는 2024년 6월까지의 정보에 기반해 답변하고, 실시간 웹 검색은 할 수 없다는 점이 있습니다. 필요하신 것이 있으면 말씀해 주세요. 무엇을 도와드릴까요?
```


### 2) 텍스트 생성 - 배치
 - https://docs.langchain.com/oss/python/langchain/models#batch

 ```
inputs = [
    "과적합이 뭔가요? 아주 짧게만 요약해 20자 정도",
    "앵무새의 털 색상이 화려한 이유는? 아주 짧게만 요약해 20자 정도",
    "AI Agent 기반 서비스는 뭐가 있나요? 아주 짧게만 요약해 20자 정도"
]

responses = model.batch(inputs)

for response in responses :
    print(response)
```