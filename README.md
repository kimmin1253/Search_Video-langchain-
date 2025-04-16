Search Video 보완 진행    
url: https://www.kaggle.com/code/kimmin1253/llm-test

데이터 로드 및 FAISS 인덱스 구축: 영상 세그먼트를 JSON 파일로부터 읽어와 SBERT 임베딩과 FAISS 인덱스로 유사도 검색을 수행

LangChain 기반 Vector Store 생성: 캡션 텍스트와 함께 세그먼트 메타데이터(시작시간, 얼굴 정보 등)를 함께 저장

LLM 및 RetrievalQA 체인 구성: HuggingFace의 FLAN-T5-small 모델과 LangChain을 사용해, retrieval된 문맥(context)과 질문을 기반으로 영상 장면 요약을 생성

프롬프트 템플릿 및 디버깅: 프롬프트 템플릿을 수정하여, retrieval된 context 내의 캡션, 시작시간, 인물정보 등을 모델이 충분히 반영하도록 유도

Agent 도구 준비: 필요 시 영상 클립 생성 도구를 LangChain Agent로 등록하여 추가 기능 구현 (현재는 간단하게 영상의 특정 구간을 클립을 생성하는걸 만들어볼 예정 ffmpeg 사용)



구성 및 코드 설명
1. 데이터 로드 및 FAISS 인덱스 구축
설명:
JSON 파일에서 영상 캡션 및 메타데이터를 불러와 각 세그먼트 정보를 리스트(segments_meta, captions_list)에 저장합니다.
SBERT 모델("bongsoo/kpf-sbert-128d-v1")을 이용해 각 캡션의 임베딩을 생성한 후, FAISS 인덱스를 구축합니다.

2. LangChain 기반 Vector Store 생성
설명:
HuggingFaceEmbeddings를 통해 동일 모델 기반의 임베딩 함수를 정의하고, LC_FAISS.from_texts()를 사용하여 captions_list와 segments_meta를 함께 저장한 vectorstore를 생성합니다.


3. LLM 및 RetrievalQA 체인 구성
설명:
FLAN-T5-small 모델을 HuggingFace pipeline을 통해 로드하고, LangChain의 HuggingFacePipeline으로 래핑하여 LLM 객체를 생성합니다.
PromptTemplate을 사용하여 retrieval된 context와 질문을 결합한 프롬프트로 RetrievalQA 체인을 생성합니다.


4. run_qa 함수 및 디버깅 출력
설명:
run_qa 함수는 입력된 질문을 기반으로 retrieval된 Document들의 문맥(context)을 구성하고, 이를 QA 체인에 전달하여 답변을 생성합니다.
디버깅용으로 retrieval된 context를 출력하여, 캡션과 함께 시작시간, 등장인물 정보가 제대로 포함되었는지 확인합니다.

5. CLI 실행 및 에이전트 통합 (Agent 테스트는 추후 구현)
설명:
메인 실행 부분에서는 CLI 선택 메뉴를 통해 QA 체인 테스트와 Agent 테스트 모드 중 하나를 선택할 수 있습니다.

향후 개선 사항
빈 응답(빈 result)의 원인:
프롬프트 템플릿의 구성, retrieval된 context의 품질, 모델의 decoding 파라미터(temperature, max_new_tokens 등) 조정 사항을 점검.
빈 응답이 발생할 경우 디버그 로그(최종 프롬프트, retrieval context)를 확인할 것.
현재 google/flan-t5-small 모델을 사용중인데 같은 프롬프트를 줬을때 google/flan-t5-base 를 사용하였을때 result에 빈 캡션을 생성하는 문제
small 모델을 사용하고 질문하고 대답시
1: QA 체인 테스트, 2: Agent 테스트
/usr/local/lib/python3.11/dist-packages/transformers/generation/configuration_utils.py:631: UserWarning: `do_sample` is set to `False`. However, `temperature` is set to `0.7` -- this flag is only used in sample-based generation modes. You should set `do_sample=True` or unset `temperature`.
  warnings.warn(
RetrievalQA 체인 응답:
Kim Minji (member: Kim Minji) started_times: 147.0, 538.0, 76.0, 146.0) 
이러한 답을 하고 있는데 지금 prompt template에서 query에 인물정보인 member, 영상 시작시간인 start_time을 묶어서 주고나서 이후에 분리해서 처리중인데
지금 이러한 대답을 내뱉는 원인 파악 필요



프롬프트 튜닝:
원하는 답변(예: 인물의 행동, 캡션 해석 등)을 생성하도록 프롬프트 템플릿을 재구성하고, 필요 시 영어와 한국어 버전 모두를 비교 실험.
같은 모델에서 한국어로 프롬프트를 주었을때 답은 하였던것도 영어로 번역해서 프롬프트를 줬을때 빈캡션을 출력하는 문제도 발생했음
영어 한국어 프롬프트의 답변 차이가 명확히 된다면 추후에 한국어 프롬프트를 영어 프롬프트로 전환후 프롬프트에 입력해주는 파이프라인을 추가하는것도 고려중


현재 상태:
gemini를 프로젝트에서나 사용가능하지 실제 사용 가능한 모델로 적용하기 위해 오픈소스에 있는 LLM을 가져와서 RAG, langchain에 있는 agents, RetrievalQA 을 사용하려고 했으나 
문제점이 많이 발견되고 아직 해결한게 하나도 없음



