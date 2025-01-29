Windows 환경에서 제작 되었으며 Gradio를 이용해 웹 페이지를 만들었습니다.

해당 파일은  아래의 체크포인트를 활용하여 제작되었습니다.

- Make a new folder named `checkpoints` under this project，and put the downloaded weights files in `checkpoints`。You can download the weights using following URLs：

  - `vit_h`: [ViT-H SAM model](https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth)

또한 Ollama 를 사용하였으며
Ollama 프로그램을 다운 받으셔서 사용하시면됩니다.
> ollama run llava



나눠서 다운로드 받으시는걸 추천합니다 둘다 용량이 좀 큰편이라 하나씩 다운 받는게 오류가 생기더라도 확인하기 편했습니다.
> ollama run llama2

해당 작품의 자세한 설명은 Velog에 올려놓을 계획입니다.

- 해당 작품 설명 Velog
https://velog.io/@bellfollow/%EC%9A%B0%EB%8B%B9%ED%83%95%ED%83%95-%ED%95%99%EA%B5%90-%EC%A1%B8%EC%97%85-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-1-Gradio-AI-%EB%AA%A8%EB%8D%B8-%ED%86%B5%ED%95%A9-%EC%9B%B9%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0




-----------------------------------------------------------------------------------------------

The project was developed in a Windows environment, and the web page was created using Gradio.

The project was made using the following checkpoints:

- Make a new folder named checkpoints under this project, and put the downloaded weights files in checkpoints. You can download the weights using the following URLs:
  
  - `vit_h`: [ViT-H SAM model](https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth)


Additionally, Ollama was used. Please download the Ollama program to use it.

> ollama run llava

- It is recommended to download them separately since both files are quite large. Downloading one at a time will make it easier to check for any errors that may occur.

ollama run llama2

A detailed explanation of this project will be posted on Velog.

Detailed project explanation on Velog: https://velog.io/@bellfollow/%EC%9A%B0%EB%8B%B9%ED%83%95%ED%83%95-%ED%95%99%EA%B5%90-%EC%A1%B8%EC%97%85-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-1-Gradio-AI-%EB%AA%A8%EB%8D%B8-%ED%86%B5%ED%95%A9-%EC%9B%B9%EC%84%9C%EB%B9%84%EC%8A%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0

# 전체적인 구성
![해당 프로젝트의 전체적인 프로세스](https://velog.velcdn.com/images/bellfollow/post/ae56218e-f1af-44cc-8308-e57b6a6afde9/image.png)

1) 먼저 User_image를  Segment를 추출해주는 SAM 모델에 집어넣어 다음과 같은 Masking image를 생성하고
1-2) Llava model에 유저의 이미지를 인풋하여 분위기를 파악하고 키워드를 도출시킵니다.
2) 유저가 배경에 들어가길 원하는 키워드를 User text에 받아 
3) Llava와 user_text의 프롬프트를 Llama2에 집어넣습니다. 
- 해당 LLM은 Blended Latent Diffusion에 프롬프트를 집어넣기전 해당 모델에 더 적합한 형태의 프롬프트를 만들어 넣어줄 것입니다. 

4) 유저가 처음 집어넣은 사진과 SAM 모델이 만들어준 마스킹 이미지, 프롬프트를 집어넣어 Result image를 집어넣습니다.

# 사용된 기술 목록
1. SAM
2. Llava 1.6
3. Llama 2
4. Blended Latent Diffusion
5. Web GUI 구성


## SAM
- Sagment Anything Model의 약자, Meta에서 개발되었음
- 해당 기술은 원하는 사진의 Mask_image를 만들기 위한 Segment를 추출하는데 사용되었습니다.
- 이미지 내의 객체를 정밀하게 분할할 수 있는 고급 컴퓨터 비전 도구입니다. 이 모델은 점, 박스, 또는 프롬프트에서 마스크를 생성할 수 있지만 해당 프로젝트에선 점을 이용하였습니다.

### 해당 모델의 아키텍처

![](https://velog.velcdn.com/images/bellfollow/post/9ed02f0a-cccf-40aa-b0f8-76e0056e4457/image.png)

![](https://velog.velcdn.com/images/bellfollow/post/7b18eae2-f129-43e2-bcf2-14a21e09a09a/image.png)

- Image Encoder :  image을 입력으로 image embedding 출력

![](https://velog.velcdn.com/images/bellfollow/post/2f6e1678-9f9c-4d75-bb54-54d4d0dd1418/image.png)

- Dense Prompt: 이미지와 공간적으로 대응되는 정보(Mask), Image Embedding에 Convoltion 수행

![](https://velog.velcdn.com/images/bellfollow/post/e472f1b4-b33a-466e-b480-6798ea297e05/image.png)

- Prompt Encoder: prompt를 입력으로 prompt embedding 출력, Points, box, text 이 3가지를 프롬프트로 취급합니다. 이번 프로젝트에서는 points를 프롬프트로 받아서 사용했습니다.

![](https://velog.velcdn.com/images/bellfollow/post/c57aa1b4-6629-4485-81ba-d3ae4c260e36/image.png)

- Mask Decode: Prompt와 Dence embedding 값을 입력으로 segmentation mask 추출

### 마스크 추출 방식 - fully automatic

![](https://velog.velcdn.com/images/bellfollow/post/8522a8a8-ff11-48b4-a60e-843d8fd322b0/image.png)

- 미리 만들어진 마스크를 어느정도 모델이 학습한 후, input 이미지에 grid point를 찍어서 모든걸 알아서 masking 하는 방법


### 헤딩 모델의 사전 학습 데이터셋 

![](https://velog.velcdn.com/images/bellfollow/post/99eba9c4-027f-4f0c-9234-0ae6218a778f/image.png)

- 11M개의 고해상도 이미지
- 1.1B개의 Mask

#### 결과물 

- 입력 사진
![](https://velog.velcdn.com/images/bellfollow/post/8b4b74c5-1ecb-480e-99c1-ca0fb13be868/image.png)

- 출력 사진
![](https://velog.velcdn.com/images/bellfollow/post/0cc3b71b-b5d9-4d2c-bb29-0a8ecf0c26f3/image.png)

- 출력 사진 결과물은 SAM에서 그대로 출력해주는 사진이 아닌, 선택된 프롬프트를 위주로 사진을 이진화 시킨 결과물입니다. 
- 해당 이미지를 이진화 시킨 이유는 후에 사용될 Blended Latent Diffusion를 위해 마스크 이미지를 만들어 주는 것입니다.
## Llava 1.6
- LLaVA(Large Language and Vision Assistant)
- 언어와 이미지 간의 복합적인 상호 작용에 중점을 둔 오픈소스 멀티모달 모델
- 해당 모델은 해당 프로젝트의 input_image의 전반적인 분위기 키워드를 출력하는데 사용되었습니다.

### 해당 모델의 아키텍처

![](https://velog.velcdn.com/images/bellfollow/post/efff63d3-292b-44d2-93f6-673ffeb0b273/image.png)

- 이미지 Xv를 입력받아 시각적 특징을 비전 인코더를 통해 Zv로 추출합니다. 
- Zv는 프로젝션 레이어 W를 통해 언어 모델의 임베딩 공간으로 변환되어 Hv로 매핑됩니다. 
	- 이 과정을 통하여 시각적 정보를 언어 모델이 이해할 수 있는 형식으로 변환합니다.
- 사용자로부터 언어 명령 Xq를 입력받고 해당 언어 임베딩 공간으로 변환시킵니다.
- Image와 word Embedding의 결합은 Linear Layer를 사용하여 결합합니다.
- 결합된 임베딩을 언어 모델에 넘기는 것으로 모델은 결과물을 도출해줍니다.


### Llava 1.6의 AnyRes

![](https://velog.velcdn.com/images/bellfollow/post/0b112a39-6aca-4388-bd05-3ced1109c50d/image.png)

- 사진을 전체적으로 나누고, 수평으로, 수직으로 한번더 나누고 이미지를 임베딩, 이미지를 작게도 리사이징한 다음 임베딩한 후 전달하게 되면 복잡한 세부사항을 인식하는 모델의 능략이 크게 향상된다고 합니다.
- 저해상도 이미지에서 종종 발생하는 모델의 "할루시네이션" 현상을 줄이는 것을 목표만든 기술입니다.


### 해당 사진의 역할
![](https://velog.velcdn.com/images/bellfollow/post/afea6a93-4a14-49ff-9aad-3be932a69544/image.png)


## Llama 2
- LLaMA(Large Language Model Meta AI) 대규모 AI 언어 모델
- Meta에서 개발
- 해당 기술은 Llava_prompts 와 User_prompt의 promp를 결합하여 Blended Latent Diffusion의 SDXL 모델에 가장 잘 맞는 프롬프트를 생성하는 역할을 합니다.

### 모델 아키텍처

![](https://velog.velcdn.com/images/bellfollow/post/74a3d0fe-6f86-4185-b9de-1f8affcf9019/image.png)

1. 사전 훈련 (Pretraining)
- Llama2는 Meta 내부 데이터를 활용하지 않고 온라인으로 공개된 데이터셋만을 이용해 pre-training을 했습니다, 2조개의 데이터를 사용했다고 합니다.
- Pretrain Corpus의 언급이 있는것으로 보아 해당 데이터셋을 사용한 것으로 생각됩니다.
	- Wikipedia: 방대한 양의 백과사전식 데이터 
    - Common Crawl: 전 세계 웹 페이지에서 수집한 방대한 텍스트 데이터 
    - BookCorpus: 다양한 주제의 책 텍스트의 모음

- 자가 지도 학습 (Self-Supervised Learning): 모델이 대규모 텍스트 데이터를 통해 기본 언어 패턴을 학습하는 과정.


2. (Human Feedback)파트
-휴먼피드백 파트에서 Human Preference 데이터도 수집하여 사용하였습니다. 해당 데이터셋은 모델이 인간의 취향과 의견을 더 잘 이해하고 반영할 수 있도록 돕습니다.

![](https://velog.velcdn.com/images/bellfollow/post/63c4d169-e84b-4b4a-9909-598a739f9695/image.png)

- SFT(Supervised fine tuning) 공개 데이터셋을 Meta 측에서 Annotation 업체를 통해 자체 데이터셋 구축하여 고품질의 소량 데이터(28000개 정도)의 데이터셋 확보 하였습니다. 
- 해당 데이터셋은 Helpfulness와 Safety 두가지 측면의 데이터 별로 구축하였습니다.
	- Helpfulness: 매우 창의적인 생성을 요구하는 입력 예) 시 만들어줘
	- Safety: 유해한 응답을 생성할 수 있는 Prompt에 대해선 응답 거부 , 예) 널 굽고싶어
라는 내용을 거부

3. 미세 조정 (Fine-Tuning)파트 
- RLHF (Reinforcement Learning with Human Feedback): 인간의 피드백을 통해 모델을 강화 학습하는 과정. 
- 거부 샘플링 (Rejection Sampling): RewardModel을 이용한 SFT데이터셋 생성 및 필터링, 여러 응답 중 가장 적절한 응답을 선택하여 모델을 강화.
- Reward Model: RLHF 훈련 과정에서 사용되는 모델, 학습 과정에서만 사용, 적절한 대답에 대한 점수를 주는 것 , 해당 모델을 사용하기 위해선 Human Preference Data구축 되어야 함,
- 근접 정책 최적화 (Proximal Policy Optimization, PPO): 강화 학습 기법을 통해 모델을 최적화하는 방법.
- 지도 학습 미세 조정 (Supervised Fine-Tuning): 
- 고품질의 지도 학습 데이터를 수집하여 모델을 학습시킵니다.예시로, 명령 조정 데이터(Instruction Tuning Data)를 사용하여 모델의 응답이 사용자 명령에 잘 따르도록 합니다.

## Blended Latent Diffusion
- Blended Latent Diffusion은 텍스트 기반의 지역 이미지 편집기술로 해당 프로젝트에서 배경의 분위기를 전환시키는 역할을 맡았습니다.

### 모델의 기본 이해 Diffusion
![](https://velog.velcdn.com/images/bellfollow/post/cd975588-79c9-4f53-8c87-971badb93316/image.png)

- Diffusion model은 사진에서 보이는 것처럼 일반적으로 임의의 Gaussian noise로 인해 손상된 이미지의 noise를 제거하도록 점진적으로 학습된다. 제거되면서 사진이 만들어 지면 

### 모델의 아키텍처 
![](https://velog.velcdn.com/images/bellfollow/post/3dd4b637-d9b4-4624-8017-bd77cb419683/image.png)

- input_image는 Encoder로 들어갑니다 해당 Encoder는 VAE(Variaitional AutoEncoder)입니다.
>여기서 VAE의 역할은? encoder와 decoder를 활용해 latent space를 도출하고, 이 결과물을 Decoding함으로써 데이터 생성을 진행시키는 것입니다. 
*latent space: 고차원의 데이터를 저차원으로 변환한 저차원 벡터가 위치하는 공간

- 노이즈를 주입시키고, 텍스트 프롬프트 d에 따라 노이즈를 제거하여 Latent Representation zfg(forground)를 생성 
- 노이즈를 주입시켜 Latent Representation zdg를 생성합니다.
- 생성된 zfg와 zbg를 Mask 를 이용해 블렌딩합니다. 블랜딩된 Latent Representation zt가 다음단계로 전달되어 노이즈를 주입하고 제거하는 동일한 과정이 반복됩니다. 
- 최종 잠재표현 z0를 VAE의 디코더를 통해 디코딩하여 최종 이미지를 생성합니다.

좀더 직관적인 이해를 위한 사진

![](https://velog.velcdn.com/images/bellfollow/post/caccc6ce-d014-44be-91e6-cd07ac02e78a/image.png)


### 결과물
- 입력 사진
![](https://velog.velcdn.com/images/bellfollow/post/8b4b74c5-1ecb-480e-99c1-ca0fb13be868/image.png)

- 출력사진
![](https://velog.velcdn.com/images/bellfollow/post/84eda1a2-5bfe-4e04-bfc5-1b7a909d65bf/image.png)

- 입력 프롬프트는 Galaxy


## Web GUI 구성

- Gradio는 Python 기반 라이브러리로 해당 프로젝트의 웹의 UI를 만드는데 사용되었습니다.
Ollama는 거대 LLM의 API사용을 용이하게 만들어주는 프로그램입니다.

![](https://velog.velcdn.com/images/bellfollow/post/57514d93-fae2-463e-a03a-0b38c5d468c6/image.png)

# 어떤 프롬프트를 사용해야 하는가?

Please explain the atmosphere of the entered picture in as much detail as possible
-> Llava에서 해당 사진의 분위기를 설명해달라고 했을때 

출력 "Create a stunning fashion image showcasing a person standing in front of a warm-toned background. The outfit consists of a bold green turtleneck sweater, a classic plaid blazer, a mini skirt, and black knee-high boots. Accessorize with a blue choker around the neck for an edgy touch. The person poses casually yet poised, with one hand tucked into the pocket of the blazer and the other resting on their hip. The lighting is soft and even, highlighting the textures of the clothing without creating harsh shadows. The background is plain but warm-toned, allowing the focus to remain solely on the outfit and the person wearing it. Create an image that exudes a blend of classic and modern fashion styles, with an emphasis on bold colors and patterns."

-> 분위기에 해당하는 말이 아닌 사진의 자체적인 설명만을 출력함 

- 해당 프롬프트를 이용한 결과

![](https://velog.velcdn.com/images/bellfollow/post/7b24cbd4-4847-49c0-ad4b-0dc4bef87864/image.png)


## 프롬프트 조정 후

Llava 1.6
-> "Please express the mood keyword of the image,words",
- 키워드를  단어로 표현해 달라는 요구를 하는 이유는 후에 Llama2에 쓰이는 프롬프트 예시를 보면 알 수 있습니다. 

->llama2 prompt
Combine '{llava_prompts}' and '{user_prompt}' to create a new atmospheric background prompt that best suits SDXL. The maximum number of tokens that can be used in the generated sentence is 77. 
Describe like this example. example : breathtaking origami, close up portrait of a mature irish man with flushed skin swimming in the cold sea at dawn, soft light, eerie serene photograph by annie leibowitz, 2 0 1 9, thomas kinkade, bouguereau, award winning, paper art pleated paper folded origami art pleats cut and fold“
-> 77token 제한 이유 -> Blended Latent Diffusion 의 CLIP 모델이 77token의 문장밖에 받아들이지 못해서 제한함

>허나 이런 프롬프트를 전해준다고 77token으로 제한되진 못합니다. 추가적인 코딩으로 출력을 조절해야합니다. 

- llava_prompt는 위의 Llava 1.6의 질문과 input_image를 넣어나온 대답으로 
-> Stylish, modern, contemporary 
- user_prompt는 사용자가 입력하는 promp로 forest를 입력하였다.

![](https://velog.velcdn.com/images/bellfollow/post/f2dac8dc-54a4-49bb-822c-4a6bd7491718/image.png)
- 해당 프롬프트는 Civitai의 SDXL 모델의 프롬프트 예시를 가져왔습니다.

- Llama2의 프롬프트에서 예시를 집어넣은 이유는 In-Context Learning의 Few-shot 방식을 활용하여 예시를 집어넣어, 해당 모델에 가장 알맞은 프롬프트를 만들기 위함입니다. 
- 숨막히는 종이 접기, 새벽에 차가운 바다에서 수영하는 홍조 피부를 가진 성숙한 아일랜드 남자의 클로즈업 초상화, 부드러운 빛,등 사람들이라면 흔치 않은 단어의 나열이지만 모델이 이해하기에는 가장 적절한 프롬프트를 만들기위한 작업이었다 라고 이해하시면 좋습니다.


# 실행결과물
![](https://velog.velcdn.com/images/bellfollow/post/45a84951-9127-45bd-82a5-14da5acf44a8/image.png)


![](https://velog.velcdn.com/images/bellfollow/post/e00364f1-338f-4357-a0f7-d8d6fd9cd6e8/image.png)


# 후기
> 프로젝트의 완성도가 생각한것과 좀 달라서 좀 더 발전시키고 싶은 마음이 큽니다. 그래도 열심히 했으니 좀 더 공부해서 더 나은 결과를 만들고 싶네요
