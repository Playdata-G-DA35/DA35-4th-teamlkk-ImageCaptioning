# 모델링 기법 선정 보고서

## 1. 프로젝트 주제
- 프로젝트의 목적 및 목표
  > 본 프로젝트는 강의 시간에 학습한 CNN 모델(VGG, ResNet 등)과 RNN 모델(LSTM, GRU)을 활용하여, 입력으로 이미지를 제공하면 해당 이미지의 내용을 텍스트로 추출하는 기능을 구현하는 것을 목표로 함.
- 프로젝트가 해결하고자 하는 문제와 그 중요성
  > - 박물관이나 전시관과 같은 그림 설명을 다루는 장소에서는 시각장애인들이 정보 접근에 어려움을 겪고 있음.
  > - 이러한 불편을 해소하기 위해 이미지 정보를 텍스트로 설명함으로써 정보 접근에 제한이 있는 사용자에게 포용성을 증대시키고자 함.
- 예상되는 결과와 영향
  > - 글로벌 사용자에 맞춰 다양한 언어로 이미지 설명을 제공할 수 있음.
  > - 시각 장애인을 포함한 각 국의 사용자에게 다양한 이미지의 접근성과 이해도를 높임.
  

## 2. 데이터 설명
- 사용 데이터 설명
  - 데이터 출처 - Kaggle
  ![alt text](image.png)
  - 데이터 타입: 이미지-텍스트 쌍
  - 데이터 크기: 총 8,091개의 Image와 40,455개의 caption.<br/>
    이미지당 5개의 정답 캡션(문장)
    - Train set : 32,000개
    - Validation set : 4,000개
- 데이터 전처리 과정 및 방법
  - Image Resize: 이미지를 크기를 조정
  - Tensor 변환: 이미지를 텐서 형태로 변환
  - Normalization: 데이터를 정규화하여 모델 입력에 적합하게 변환
    ![alt text](image-7.png)

## 3. 모델링 기법 후보
- 고려된 모델링 기법
  - **Encoder** : 이미지 특징을 추출하는 `CNN(VGG, ResNet)`모델을 적용할 것을 고려함.
  - **Decoder** : 텍스트 데이터를 순차적으로 처리하는 `RNN(LSTM, GRU)`모델을 적용할 것을 고려함.
  - 모델의 예상 아키텍처
    - `Encoder`
      - Image를 입력받아 CNN 모델을 통해 Feature Map을 추출
      - 추출된 Feature Map을 Embedding 처리하여 Context Vector로 변환
    - `Decoder`
      - 추출된 Context Vector를 LSTM의 입력값으로 넣어 예상 텍스트로 변환
    ![alt text](image-9.png)
  - Pretrained 모델을 사용한 경우 고려한 모델들
    - CNN: BLIP, VGG16, ResNet101 등
    - RNN: BERT, BLIP
    

## 4. 모델 선정 기준 및 모델링 기법들 비교
- 성능 지표
    > ## **BLEU**
    > - 예측값과 실제값의 n-gram 일치도를 기반으로 동일한 가중치를 두어 유사도를 계산.
    > - 예측한 문장의 길이가 짧을 경우 큰 패널티가 부여됨.
    ![alt text](image-1.png)<br/>

    > ## **NIST**
    > - BLEU와 유사한 방식으로 n-그램을 사용하지만, 희귀한 n-그램에 더 큰 가중치를 부여.
    > - 문장 길이에 따른 영향을 적게 받음.

- 학습 시간 및 리소스 요구 사항 비교
  > **VGG + LSTM 모델**
    > - 학습 시간: 1 epoch 당 파라미터 설정 별로 약 90~200초 소요
    > - 파라미터 설정별 early stopping : **30~50 epoch**
    > - 학습 환경(GPU) : Google Colab(A100, L4), GTX3060ti
- 모델의 장단점 분석
  - **VGG + LSTM 모델**
    - 장점: 학습 시간과 자원 소모가 상대적으로 적으며, 기본적인 이미지와 텍스트 처리에 적합함.
    - 단점: 복잡한 데이터나 대규모 데이터 처리에는 한계가 있음.
  - **ResNet + GRU 모델**
    - 장점: 복잡한 이미지 특징을 효율적으로 학습할 수 있음.
    - 단점: 학습 시간이 비교적 길고, 고성능의 GPU를 필요로 함.


## 5. 선정된 기법
- a. 최종 선정 모델링 기법
  > Encoder : VGG 모델의 Feature Extract Layer를 간소화하여 사용
  ![alt text](image-2.png)
  
  > Decoder :  Embedding Layer 1개와 LSTM Layer 1개로 구성
  ![alt text](image-6.png)
- b. 선정 이유
  - 여러 파라미터들을 조정해가며 학습하는데에 시간 소요가 많기 때문에 비교적 가벼운 레이어로 모델을 구성하여 사전 학습된 모델을 사용하지 않고, 주어진 데이터로 직접 학습을 진행하기 위함.
- c. 선정 모델의 구조
  - 모델의 아키텍처
    - Encoder
      ![alt text](image-10.png)
      출처 : Git Hub(https://github.com/sgrvinod/a-PyTorch-Tutorial-to-Image-Captioning)




  - 레이어 수
    - Encoder : MaxPooling Layer 1개
    - Decoder :  Embedding Layer 1개, LSTM Layer 1개

  -  파라미터
      > **파라미터 수 : 15개**<br/>
    is_attn": 0<br/>
    "img_size": 224<br/>
    "enc_hidden_size": 14<br/>
    "vocab_size": 10000,"max_len": 32<br/>
    "dec_hidden_size": 512<br/>
    "dec_num_layers": 1<br/>
    "dropout": 0.5<br/>
    "batch_size": 64<br/>
    "epochs": 100<br/>
    "enc_lr": 0.0001<br/>
    "dec_lr": 0.0004<br/>
    "regularization_lambda": 1<br/>
    "result_num": 10<br/>
    "topk": 5<br/>
    "early_stop_criterion": 5

  - 사용된 기술
    - Encoder : VGG 모델의 Feature Extract Layer를 간소화하여 사용
    - Decoder : Embedding Layer 1개와 LSTM Layer 1개
    - 평가 지표 - 총 4가지 기준으로 평가
        - BLEU: n-gram 2, n-gram 4인 경우.
        - NIST: n-gram 2, n-gram 4인 경우.


---


# 테스트 설계 보고서

## 1. 모델 평가 기준
- 성능 지표
- 처리 속도 (추론 시간, 학습 시간)

## 2. 테스트 환경
- 하드웨어(GPU, 메모리, CPU 환경)
- 소프트웨어 (프레임워크 등)

## 3. 테스트 결과


---


# 프로세스 검토 결과 보고서

## 1. 프로젝트 진행 프로세스 설명
- 프로젝트 흐름도 및 단계 별 설명
  - 단계별 목표 및 주요 활동
- 각 단계에 참여자
  - 역할 및 책임
- 각 단계에서 사용된 도구 및 시스템
  - 사용된 software, framework 등

## 2. 각 프로세스 결과 분석
- 성과
  - 주요 성과 지표 (성과물, 시간)
  - 성과 평가
- 문제점
  - 진행 중 발생한 문제점, 어려웠던 점
  - 그 문제가 프로젝트 또는 그 프로세스 진행에 미친 영향도
  - 해결 방식
- 개선점
  - 이번에는 적용하지 못했으나 차후 프로젝트를 한다면 개선할 점