
# Focus-Read(독서 도우미)
'문해력 향상을 위한 ai 독서 도우미 프로젝트'입니다.

![image](https://github.com/guscldns/Focus-Read/assets/130722839/357f285c-ec62-4eb2-9045-d50c9baa8ff1)
***

## 사용한 모델
model | train | used
----|------|---|
KoBART-summarization | Y | Y |
skt/kobert-base-v1 | N | Y |
clovaai/donut | Y | N |
***


## 사용한 데이터
data   | explanation |
-----------|----------|
[요약문 및 레포트 생성 데이터](https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&aihubDataSe=realm&dataSetSn=582) | 다양한 한국어 원문 데이터로부터 정제된 추출 및 생성 요약문을 도출하고 검증한 한국어 문서요약 AI 데이터셋  |
[저작권만료 어문데이터](https://gongu.copyright.or.kr/gongu/wrt/wrtCl/listWrtText.do?menuNo=200019&pageIndex=1&sortSe=date&licenseCd=97&searchWrd=&pageUnit=12) | 저작권 권리처리가 된 콘텐츠를 공유하는 공유마당에서 저작권이 만료된 어문데이터 PDF | 
[한국어기초사전](https://krdict.korean.go.kr/download/downloadPopup) |  국립국어원의 한국어 학습용 웹 사전  |
자체 제작 데이터  |  생성형 AI 서비스들을 이용하여 요약문 생성 후 검수 (총 300개)|
***


## OCR 
+ ### [clovaai/donut](https://github.com/clovaai/donut) 모델 Fine-tuning
  + How to train   
    ```c
    ! python /home/alpaco/final_Project/donut/train.py --config /home/alpaco/final_Project/donut/config/train_cord.yaml \
        --pretrained_model_name_or_path "naver-clova-ix/donut-base-finetuned-cord-v2" \
        --dataset_name_or_paths '["/home/alpaco/final_Project/donut/synthdog/outputs/KoreanData"]'\
        --resume_from_checkpoint_path "/home/alpaco/final_Project/donut/result/train_cord/test01/" \
        --result_path "/home/alpaco/final_Project/donut/result/" \
        --exp_version "test01"
    ```

  +  경로는 자신의 경로로 변경 필수
  +  레퍼런스
      + https://github.com/clovaai/donut
      + [OCR-free Document Understanding Transformer](https://arxiv.org/abs/2111.15664)
  + [1020 학습한 모델 bin](https://drive.google.com/file/d/151luqrFE7TqRaEOJS0GucITZAlduobsk/view?usp=share_link)

+ ### Google API
    +  사용방법   
    ```c
        from google.cloud import vision
    
        os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "API_key.json"
        # API 가져오기
        client = vision.ImageAnnotatorClient()
    
        # 주석을 추가할 이미지 파일 이름
        file_name = os.path.abspath(image_path)
    
        # 이미지 로드
        with io.open(file_name, 'rb') as image_file:
            content = image_file.read()
    
        image = vision.Image(content=content)
    
        # 이미지 OCR
        response = client.document_text_detection(image=image)
    
        # 이미지 OCR 텍스트 전문
        full_text = response.full_text_annotation.text
    
    ```

---

## summary 
+ ###  [KoBART-summarization](https://github.com/seujung/KoBART-summarization ) 모델 Fine-tuning
  + How to train  
     ```c
    !CUDA_VISIBLE_DEVICES=0 python /home/alpaco/hw/KoBART-summarization/train.py --gradient_clip_val 1.0 \
        --train_file '/home/alpaco/hw/KoBART-summarization/data/train.tsv' \
        --test_file '/home/alpaco/hw/KoBART-summarization/data/test.tsv' \
        --max_epochs 1001 \
        --checkpoint checkpoint \
        --accelerator gpu \
        --num_gpus 1 \
        --batch_size 26 \
        --num_workers 4
    ```
---
## Keyword
+  KeyBERT
   + How to used
       ```c
      # BERT 모델 로드
      model = BertModel.from_pretrained('skt/kobert-base-v1')
      
      # KeyBERT 모델 초기화
      kw_model = KeyBERT(model)
      
      # 키워드 추출
      keywords = kw_model.extract_keywords(text, keyphrase_ngram_range=(1, 1), stop_words=None, top_n=10)
       ```
---
## 배운점
1. 장문 Task일 경우 많은 데이터와 좋은 성능의 GPU에 따라 학습성능이 달라진다
2. 모델이 한 번 볼수 있는 데이터 범위가 넓을 수록 학습이 더 잘된다
3. batch_size는 클 수록 좋고 learning_rate를 처음부터 미세하게 주는 건 학습에 적합하지 않다
4. 한 종류의 많은 데이터보다 여러가지 종류의 데이터를 보는 것이 일반화에 더 효과적이다
5. 모델이 클수록 적게 학습해도 성능이 나오지만 출력시 속도를 체크 후 자신의 Task에 맞는지 확인이 필요하다. 
---
<p align="center">
  <img src="https://github.com/guscldns/Focus-Read/assets/130722839/fb47ecf9-528c-4ed6-91be-b651a42ad510">
</p>
