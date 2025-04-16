Search Video 보완 진행

데이터 로드 및 FAISS 인덱스 구축: 영상 세그먼트를 JSON 파일로부터 읽어와 SBERT 임베딩과 FAISS 인덱스로 유사도 검색을 수행

LangChain 기반 Vector Store 생성: 캡션 텍스트와 함께 세그먼트 메타데이터(시작시간, 얼굴 정보 등)를 함께 저장

LLM 및 RetrievalQA 체인 구성: HuggingFace의 FLAN-T5-small 모델과 LangChain을 사용해, retrieval된 문맥(context)과 질문을 기반으로 영상 장면 요약을 생성

프롬프트 템플릿 및 디버깅: 프롬프트 템플릿을 수정하여, retrieval된 context 내의 캡션, 시작시간, 인물정보 등을 모델이 충분히 반영하도록 유도

Agent 도구 준비: 필요 시 영상 클립 생성 도구를 LangChain Agent로 등록하여 추가 기능 구현 (현재는 간단하게 영상의 특정 구간을 클립을 생성하는걸 만들어볼 예정 ffmpeg 사용)





