# [Solution Summary] Prostate cANcer graDe Assessment (PANDA) Challenge

categories: Competition
tags: Summary

## 대회 소개 *[(참조)](https://www.kaggle.com/rohitsingh9990/panda-eda-better-visualization-simple-baseline)*

### 전립선암이란?

: 남성에게 가장 흔한 암 종류 중 하나입니다. 보통 전립선암은 천천히 자라고 전립선 안에만 있어 큰 위험요소는 아니지만 특정 종류의 전립선 암은 매우 빠르게 자라고 퍼집니다.

- 전립선 암 검사
1. Digital rectal exam (DRE): 의사가 장갑을 끼고 직장에 손을 넣어 전립선의 질감, 모양, 크기를 통해 이상을 탐지합니다.
2. Prostate-specific antigen (PSA) test: 정맥의 혈액검사를 통해 PSA라는 전립선에서 분비되는 물질 검사를 합니다. 보통 적은 양이 분비되지만 이보다 많다면 전립선 감염, 염증 혹은 암이 의심됩니다.
3. Ultrasound: 직장에 작은 크기의 도구를 넣어 음파를 통해 전립선 사진을 촬영합니다.
4. Collecting a sample of prostate tissue: 모든 검사에서 암이 의심된다면 생체(조직) 검사를 통해 암을 판별합니다. 작은 침을 전립선에 넣어 조직을 수집하고 분석합니다.
- GLEASON score
: 생체 검사를 통해 암이라고 확정받으면 암의 aggrerssive level을 판단합니다. 병리학자는 암 세포가 정상 세포에 비해 얼마나 다른지를 보고 판단하고 점수가 높을 수록 심각한 암입니다.
Gleason score는 일반적으로 사용되는 전립선 암 점수입니다. 두 점수를 합산하여 범위는 2부터 10입니다. 2보다 낮은 점수는 보통 사용되지않습니다.
- ISUP grade
: GLEASON score를 합산하여 ISUP grade를 나타냅니다. 범위는 1~5입니다.
    - Gleason score 6 = ISUP grade 1
    - Gleason score 7 (3 + 4) = ISUP grade 2
    - Gleason score 7 (4 + 3) = ISUP grade 3
    - Gleason score 8 = ISUP grade 4
    - Gleason score 9-10 = ISUP grade 5
- Gleason score 채점방식
: 전립선 조직은 haematoxylin & eosin(H&E) 염색을 통해 염색됩니다. 조직은 내외분비샘 조직(glandular tissue)과 결합 조직(connective tissue)으로 구성됩니다.
분비샘 조직은 하얗거나 가지친 구멍처럼 보이는 구조로 되어있습니다. 분비샘의 모습은 GLEASON score 채점의 기반이 됩니다. 건강한 분비샘 구조는 암이 심각해질 수록 모양을 잃게 되며 점수는 3, 4, 5로 증가합니다.

    > 한글 의료 용어는 정확하지 않을 수 있습니다. 수정 사항이 있다면 알려주세요.

    [A]Benign prostate glands with folded epithelium(상피 접힌 전립선): 세포질은 창백하고 핵은 작고 규칙적입니다. 분비샘들은 함께 뭉쳐있습니다.

    [B]Prostatic adenocarcinoma(정위선암종): Gleason Pattern 3은 분비샘 모습의 손실이 없습니다. 작은 분비샘이 양성 분비샘 사이에 침투합니다. 세포질은 종종 어둡고 핵은 어두운 색소와 몇몇 두드러진 뉴클레오로 확대됩니다. 각 상피부에는 별도의 lumen이 있습니다.

    [C]Prostatic adenocarcinoma(정성 선종암): Gleason Pattern 4는 분비샘 모습 손실을 가지고 있습니다. Luminar를 형성하려는 시도가 있었지만 종양이 완전하고 잘 발달된 분비샘을 형성하는데 실패했습니다. 이 현미경 영상은 불규칙적인 암, 즉 다발성 luminar를 가진 상피부를 보여줍니다. 또한 잘 형성되지 않은 작은 분비샘과 퓨전된 분비샘도 있습니다. 이 모든 것이 Gleason Pattern 4에 포함되어 있습니다.

    [D]Prostatic adenocarcinoma(전립선 선암종): Gleason Pattern 5는 거의 완전한 분비샘 모습 손실을 가지고 있습니다. 분산되어 있는 단일 암세포가 stroma에서 발견됩니다. Gleason Pattern 5는 또한 암세포의 고체 시트나 가닥을 포함할 수 있습니다. 모든 현미경 사진에는 20배 렌즈 확대 시 hematoxylin & eosin stains이 표시됩니다.

    ![%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled.png](%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled.png)

### 데이터

- 11,000 전립선 생체(조직)검사 whole-slide images (WSI)
- 각 WSI의 ISUP grade, GLEASON score
- 제공: Radboud University Medical Center, Karolinska Institute

![%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%201.png](%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%201.png)

- Sample Images
    1. 이미지 크기는 x축과 y축 모두 5,000~40,000 px 사이로 큽니다.
    2. 이미지는 레벨 3까지 존재합니다 (1배, 4배, 16배 축소)
    3. 조직은 서로 다르게 회전되어있고 이에 큰 의미는 없습니다.
    4. 병리조직들의 색 차이는 매우 크고 이는 연구 절차 차이때문입니다. 

![%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%202.png](%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%202.png)

- Label masks
: pixel-label mask 정보가 주어져있고 각 기관마다 다르게 표시하였습니다.
    - **Radboudumc**: Prostate glands are individually labelled. Valid values are:
        - 0: background (non tissue) or unknown
        - 1: stroma (connective tissue, non-epithelium tissue)
        - 2: healthy (benign) epithelium"
        - 3: cancerous epithelium (Gleason 3)
        - 4: cancerous epithelium (Gleason 4)
        - 5: cancerous epithelium (Gleason 5)
    - **Karolinska**: Regions are labelled. Valid values:
        - 0: background (non tissue) or unknown
        - 1: benign tissue (stroma and epithelium combined)
        - 2: cancerous tissue (stroma and epithelium combined)

    반자동적으로 생성되었기때문에 noisy합니다.

    ![%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%203.png](%5BSolution%20Summary%5D%20Prostate%20cANcer%20graDe%20Assessmen%20bdd3887f74dd4402af7b47e9a7c77145/Untitled%203.png)

## 상위 Solutions

- [1st Place Solution](https://www.kaggle.com/c/prostate-cancer-grade-assessment/discussion/169143)
    - Data Preprocessing
        - Geting cleaned labels
        1. efficientnet-b1을 k-fold로 학습시킨 후, validation set에 대해 예측값을 낸다.
        2. ground truth와 isup grade의 차이가 클 경우 해당 데이터를 삭제한다.

        ```bash
        def remove_noisy(df, thresh):
            gap = np.abs(df["isup_grade"] - df["probs_raw"])
            df_removed = df[gap > thresh].reset_index(drop=True)
            df_keep = df[gap <= thresh].reset_index(drop=True)
            return df_keep, df_removed

        df_keep, df_remove = remove_noisy(df, thresh=1.6)
        show_keep_remove(df, df_keep, df_remove)
        ```

        - Split: stratifed k-fold with [imghash](https://www.kaggle.com/yukkyo/imagehash-to-detect-duplicate-images-and-grouping) or stratified k-fold with gleason-score and imghash similarity
        - Tile Method: [iafoss tile method](https://www.kaggle.com/iafoss/panda-concat-tile-pooling-starter-0-79-lb) (tile size 192, tile num 64)
    - Model
        1. Resnext50_32x4d: head(3 * reg_head + 1 * softmax head)
        2. Effnet-b1 + generalized-mean pooling
    - Training
        1. Local train & pred
        2. Remove noisy label
        3. Re-train
    - Not worked
        - Remove noisy by confident-learning
        - Cycle GAN augmentation(karolinska radboud)
        - test with AdaBN & Freezing BN at train
        - CutMix, Mixup (before denoising)
- [2nd Place Solution](https://www.kaggle.com/c/prostate-cancer-grade-assessment/discussion/169108)

## Reference

- [PANDA - EDA + Better Visualization+Simple Baseline](https://www.kaggle.com/rohitsingh9990/panda-eda-better-visualization-simple-baseline)
- [1st Place Solution [PND]](https://www.kaggle.com/c/prostate-cancer-grade-assessment/discussion/169143)
- [2nd Place Solution [Save the Prostate]](https://www.kaggle.com/c/prostate-cancer-grade-assessment/discussion/169108)