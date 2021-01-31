# 유사 대회 솔루션

Assign: Jiyeon Lee
Status: In Progress
날짜: Jan 24, 2021 → Jan 30, 2021
열: https://www.kaggle.com/c/vinbigdata-chest-xray-abnormalities-detection/discussion/207694
주차: 2주차

- 리뷰
- 저희 대회에 적용할만한 것
- 시도하면 안될 것들

---

# 1위 솔루션

## tl; dr (Too long; Didn't read)

(대충 길고 장황해서 읽기 힘들텐데 죄송하다는 뜻...)

- RetinaNet
- Deformable R-FCN
- Deformable Relation Networks
- 최종 예측 길이와 너비를 87.5%로 조정함.
- 박스는 여기 코드를 통해서 앙상블했다.
- 전체 코드 링크 :
- 1개의 InceptionResNetV2와 5개의 Deformable Relation Networks로 6개 모델을 앙상블 해 stage 2의 private leaderboard에서 0.253을 달성했다!

## Classification, 분류

- Keras2.2를 사용. 분류 모델은 아래와 같은 개별 모델로 구성. [형식 : modelArcitecture(numClasses)(imgSize)]
- 모델은 2개 클래스 (opacity vs. not) 또는 3개 클래스(opacity vs. not normal, no opacity vs. normal )에서 학습. 각 모델은 다른 방식으로 학습되었습니다.
    - InceptionResNetV2 (2) (256) , InceptionResNetV2 (2) (320)
    - InceptionResNetV2 (3) (256) , InceptionResNetV2 (3) (320)
    - Xception (2) (384) , Xception (2) (448)
    - Xception (3) (384) , Xception (3) (448)
    - DenseNet169 (2) (512) , DenseNet169 (3) (512)
    - 먼저 15개의 클래스를 사용하여 NIH Chest X-ray14 데이터셋에서 ImageNet 사전 훈련된 네트워크를 훈련시켰습니다. (14개 결과 + 비정상 vs. 정상) 그런 다음 폐렴 데이터 세트에서 해당 가중치를 미세 조정 했습니다. 이로 인해 ImageNet 가중치를 사용한 훈련 결과가 로컬에서 약 1% 향상되었습니다.
- 모델은 색이 반전될 확률이 50%, 뒤집힐 확률이 50%, 다른 확대로 (예. 대비 향상, 자르기, 회전 등) 확대 될 확률이 50%로 학습되었습니다. 최종 예측을 위해 모델당 15x TTA를 사용하고 최종 분류 점수에 대해 150개의 예측을 평균했습니다. 1단계 테스트 라벨이 출시되었을 때 분류 앙상블의 AUC는 0.93이었습니다.
- 훈련 (단일 읽기) 데이터와 테스트 (삼중 읽기) 데이터의 분포는 상당히 달랐습니다. 우리는 STR의 흉부 펠로우십 훈련을 받은 방사선 전문의와 독자의 증가가 유병률을 증가시킨 불투명도에 대한 민감도 증가 또는 임상적 의심의 증가에 기여했다고 생각합니다. (??)

## 시도하면 안될 것들

- IoU > 0.4 및 <0.4로 분류하기 위해 감지 모델에의해 생성된 접을 수 없는 경계 상자예측에 대해 다른 분류기를 훈련했었습니다.
- 최종 제출에서 겹치는 경계 상자의 NMS : 1 단계 및 2단계 LB에 대한 최종 제출의 여러 이미지에 겹치는 상자가 있습니다. 이를 억제하면 실제로 점수가 감소했습니다.

### 전체 코드

[i-pan/kaggle-rsna18](https://github.com/i-pan/kaggle-rsna18)

1. Keras
2. MXNet
3. Keras-RetinaNet
4. Data
    - `SETTINGS.json` 에 지정된 디렉토리를 사용.
5. Training
    - 앙상블의 모든 모델을 학습하는 커맨드 : `sh train.sh`
    - **Trained Models** : 이 솔루션에서 학습한 모델.
    `models/` 디렉토리를 지우고 아래로 대체하면 된다. (22GB)

        ```jsx
        wget --no-check-certificate \
                                 -r \
        'https://docs.google.com/uc?export=download&id=12abFXy7-FOwoKxFSJ__IbOGm9oQDU7CQ' \
        -O models.tar.gz
        ```

    - **Pretrained Model** : NIHY ChestXray14데이터셋의 InceptionResNetV2, Xception, DenseNet169를 사용. 훈련시키는 데에 이 pretrained 모델들을 필요로 한다. 아래 명령어로 다운로드 가능하고 `models/pretrained` 에서 unzip해주면 된다.

        ```jsx
        wget --no-check-certificate \
                                 -r \
        'https://docs.google.com/uc?export=download&id=1rI_WSlot6ZNa_ERdLSCsGquUXEK_ikYb' \
        -O pretrained.zip
        ```

6. Multiple GPU
    - `CUDA_VISIBLE_DEVICES` 를 수정. (training script에서)
    - deformable R-FCN/relation 네트워크에서는 YAML config파일의 `gpus` 옵션을 수정해야한다. 이 옵션은 아래 경로에 위치해있다.

        `$TOP/models/<DeformableConvNets|RelationNetworks>/skpeep/<unfreeze|default>/cfgs` 

    - 이 친구는 각 training fold마다 다른 YAML 파일이 있고 `gpus` 옵션은 각각 다르게 설정해주어야 한다.
    - multiple GPU로 트레이닝 : `gpus : '0,1,2'`
7. Model Checkpoints
    - 앙상블을 훈련한 후 선택한 체크포인트에 따라 fold당 1개의 모델 체크포인트를 새 폴더로 이동.
    - 모델 체크포인트는 `$TOP/models/classifiers/snapshots/<binary|multiclass>` 에 저장.
    - 그 다음 디렉토리를 만든다. `mkdir $TOP/models/classifiers/binary $TOP/models/classifiers/multiclass/`
8. predict
    - `sh predict.sh`
    - 최종 submission 파일을 생성한다.
9. Simple Model
    - 1개의 InceptionResNetV2분류기와 5개의 deformable relation network로 학습한 모델.
    - 스테이지 2의 private LB에서 0.253을 기록했다.

### 몰라서 정리해보았던 개념들

- Deformable Convolutional Networks

    Reference

    - 원본 논문 : [https://arxiv.org/abs/1703.06211](https://arxiv.org/abs/1703.06211)
    - 설명해주는 블로그 (한글) : [https://eehoeskrap.tistory.com/406](https://eehoeskrap.tistory.com/406)
    - 깃허브 링크 : [https://github.com/i-pan/kaggle-rsna18/tree/master/models/DeformableConvNets](https://github.com/i-pan/kaggle-rsna18/tree/master/models/DeformableConvNets)

    말 그대로 변형이 가능한 CNN이라는 개념인데, 기존 CNN에서 사용하는 여러 연산(convolution, pooling, roi pooling 등)이 기하학적으로 일정한 패턴을 가정하고 있기 때문에 복잡한 transformation에 유연하게 대처하기 어렵다는 한계를 지적했다. 

    그래서 이 논문은 **어떤 데이터 x를 뽑을 것인지에 초점을 맞췄다**는 것이 포인트이다.

    예시를 들자면, 3x3 convolution이 있다고 할 때 가운데 (0, 0) 픽셀을 중심으로 (-1, -1), (-1, 0), (-1, 1), (0, -1)... (1, 1) 등 9개의 픽셀을 가지고 컨볼루션 연산을 하는데, 여기에 오프셋을 도입해 (-1, -1+dx) 등의 샘플로부터 convolution을 하나즌 이야기이다. 이럴 경우 샘플의 위치가 꼭 정방향이 아닌 약간 회전된 위치에서도 나올 수 있게 된다.

    기존 CNN의 문제점을 해결할 수 있는 Deformable Convolution 및 Deformable ROI Pooling 방법을 제시한다.

    1. **Deformable Convloution**

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled.png)

        - Deformable Convolution은 위 그림에서 convolution에서 사용하는 sampling grid에 2D offset을 더한다는 아이디어에서 출발한다.
        - 그림 (a)에서 초록색 점이 일반적인 convolution의 sampling grid라면 b, c, d처럼 다양한 패턴으로 변형시켜 사용할 수 있다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%201.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%201.png)

        - 위 그림은 3x3 deformable convolution의 예시인데, 위 그림과 같이 기존 conv layer외에도 초록색 그림의 conv가 더 존재한다. 이 레이어는 각 입력에 2D Offset을 학습시키기 위한 것으로, 이 오프셋은 정수값이 아니라 fractional number이기 때문에 소수값이 가능하며 실제 계산은 선형보간법으로 이루어진다. (2차원이기 때문에 bilinear interpolation). 즉, 필터 사이즈를 학습하여 object 크기에 맞게 변화하도록 한 것!
        - 학습 과정에서 (1)output feature를 만드는 convolution kernel과 (2)offset을 정하는 convolution kernel을 동시에 학습할 수 있다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%202.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%202.png)

        - 위 그림은 deformable convolution filter에서 학습한 offset을 반영한 sampling location.
        - 초록색 사각형은 filter의 output의 위치. 일정하게 샘플링 패턴이 고정되어있지 않고, 큰 오브젝트에서는 receptive field가 더 커졌다.

    2. **Deformable ROI Pooling**
        - ROI (Region Of Interest) Pooling : 크기가 변하는 사각형 입력 region을 고정된 크기의 feature로 변환하는 과정. (아래 그림 참고)

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%203.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%203.png)

        - Deformable ROI Pooling에서는 (1) ROI Pooling Layer와 (2)offset을 학습하기 위한 레이어로 구성된다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%204.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%204.png)

        - offest을 학습하는 뿐에 컨볼루션 연산이 아니라 Fully connected layer를 사용했다. 마찬가지로 학습 과정에서 offset을 결정하는 fc layer도 backpropagation을 통해 학습된다.
- Inception ResNet Model

    Reference

    - 텐서플로우 블로그 : [https://tensorflow.blog/2016/09/01/new-convnet-model-inception-resnet-v2/](https://tensorflow.blog/2016/09/01/new-convnet-model-inception-resnet-v2/)
    - POD_Deep-Learning 블로그 : [https://poddeeplearning.readthedocs.io/ko/latest/CNN/Inception.v4/](https://poddeeplearning.readthedocs.io/ko/latest/CNN/Inception.v4/)
    - PAPER 블로그 : [https://norman3.github.io/papers/docs/google_inception.html](https://norman3.github.io/papers/docs/google_inception.html)

    Inception의 여러 버전 중 하나가 GoogLeNet이라고 구글이 밝혔다. 

    GoogLeNet은 VGGNet보다 훨씬 복잡한 구조로 저조한 활용도를 보이고 이듬해 ResNet이 2배의 격차를 내면서 우승하게 되면서 새롭게 탄생한 모델입니다.

    1. 인수분해

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%205.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%205.png)

        - 5x5 convolution을 적용하면 한번에 넓은 특징을 추출해낼 수 있지만 25개의 파라미터를 학습해야 한다.
        - 5x5 convolution을 3x3 convolution을 두번에 걸쳐 연산한다! (5x5 한번 : 25, 3x3 두번 : 18)
        - 크고 무거운 컨볼루션을 여러개의 3x3으로 쪼개어 더 깊고 가벼우면서도 구조적으로는 간단한 모델을 작성하자는 것!
        - 여기에서 좀 더 나아가서 커널을 비대칭적으로 인수분해하는 고안도 등장했다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%206.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%206.png)

    2. 그리드(grid)를 줄이기 위한 고찰
        - 그리드, 즉 해상도를 감소시키는 기술
        - 연산량을 감소키기 위해 이미지의 사이즈를 줄이는데, 이를 위한 방법으로는 두가지가 있다. (1) 풀링을 하거나 (2) convolution의 스트라이드를 크게 하거나.
        - 여기서는 병렬 연산을 통해 해결하려 하였다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%207.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%207.png)

        - 스트라이드 2의 컨볼루션 연산이 풀링 연산과 함께 처리되고 있다.

        ![%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%208.png](%E1%84%8B%E1%85%B2%E1%84%89%E1%85%A1%20%E1%84%83%E1%85%A2%E1%84%92%E1%85%AC%20%E1%84%89%E1%85%A9%E1%86%AF%E1%84%85%E1%85%AE%E1%84%89%E1%85%A7%E1%86%AB%204bdd61bb8f4e4cd89456cf490a83cb37/Untitled%208.png)

---

## 2위 솔루션

- Retinanet 모델을 사용. 하나의 모델을 4개의 fold에 나누어 학습시킨 후 앙상블했다.
    - Retinanet을 사용한 이유 : 모델이나 SSD와 같은 Faster RCNN에 비해 훨씬 간단하면서도 비슷한 결과를 얻을 수 있기 때문.
    - 여기 깃허브 코드를 참고함. : [https://github.com/yhenon/pytorch-retinanet](https://github.com/yhenon/pytorch-retinanet)
    - **Modifications pytorch retinanet implementation**
        - 베이스 모델로 se-resnext101이 가장 잘 나왔다. (se-resnext50은 약간 더 결과나 나빴다.)
        - 더 작은 상자를 처리하기 위해 더 작은 anchor에 대한 추가 출력을 추가했다.
        - 전체 이미지의 등급을 예측하는 또 다른 분류 출력을 추가했다. (No lung Opacity, Not normal, normal, lung opacity)
            - 이것을 사용하진 않았지만 다른 관련 기능을 예측하는 모델을 만들어 결과가 향상되었다.
        - 원래 pytorch-retinanet 구현은 상자가 없는 이미지를 무시하는데, 그 친구들도 손실을 계산하도록 바꾸었다.
        - 분류 출력이 앵커 위치, 크기 회귀 출력에 비해 훨씬 더 빠리 과적합되므로 앵커 및 전체 이미지 클래스 출력에 드롭아웃ㅇ들 추가했다.
    - **Training**
        - Stratified Split 한 4개의 fold를 사용.
            - Stratified K Fold란

                Stratified K-fold 교차 검증 방법은 원본 데이터에서 레이블 분포를 먼저 고려한 뒤, 이 분포와 동일하게 학습 및 검증 데이터 세트를 분배한다.

- **Augmentation**
    - 약간의 회전 (최대 6도), shift, scale, shear and h_flip 일부 이미지 흐림 및 노이즈 및 감마를 변경했다.
    - 라벨을 무효화하지 않는지 확인하기가 어려웠기때문에 밝기/감마 증가량을 제한했다.
    - 회전이 bounding box 크기에 미치는 영향을 줄이기 위해 모서리를 회전하는 대신 모서리에서 1/3 및 2/3 모서리 길이로 각 모서리에서 두 포인트를 회전하고 총 8포인트로 새 경계 상자를 최소/최대로 계산했다.
- **Resize**
    - 원본 이미지를 512x512 해상도로 조정했다. (256으로 하면 결과가 저하되고 전체 해상도를 사용하는건 실용적이지 않았음)
- **Submission**
    - 1080ti GPU에서 약 1시간동안 12epoch 훈련.
    - 그 외로 이미지에서 다른 메타 데이터를 사용하지 않았고, 방향 필트가 누출되는 경계선을 느꼈다.

**train set과 test set 데이터 레이블의 불균형**

- 해당 데이터는 train set과 test set의 레이블 분포가 달랐는데, 다른 라벨링 방법으로 인해 그러했다.
- 여러 방사선 전문의가 유사한 레이블에 사용되는 교차로 각 이미지에 레이블을 지정.
- 이것이 더 많은 상자를 예측하긴 하지만 복잡잡한 경우에는 작은 크기로 이루어질 것이라고 생각함.
- 다른 폴드의 출력을 사용하여 이 프로세스를 대략적으로 시뮬레이션 하려고 했다.
- 앵커 크기의 평균 출력을 사용하는 대신 20 백분위수 값을 사용하고 모델간 80 백분위수와 20백분위수 차이에 비례하게 값을 줄였다.

**시도하려고 했던 것들**

- **NIH 데이터 세트**
    - 더 큰 데이터세트지만 레이블의 품질이 낮다.
    - 기존 데이터셋에 끼워서 사용하면 좋은 결과를 보여줄 것 같고, 그렇지 않더라도 최소한 기본 모델을 사전훈련하는데 사용하는 것이 좋을 것 같다.