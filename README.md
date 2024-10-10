# Team Project #3: ThreeKcal
## Overview
ML 어플리케이션 서비스를 위한 기본 리포지토리

팀 프로젝트 #3: 팀 ThreeKcal



`DistilRoBERTa` 기반의 text classifier 모델인 [michellejieli/emotion_text_classifier](https://huggingface.co/michellejieli/emotion_text_classifier) 을 통해:
- `Streamlit` 기반 웹 어플리케이션을 통해 사용자 입력을 받고, 해당 문장에 대한 sentiment analysis/prediction 실행 (🤬🤢😀😐😭😲)
- 해당 prediction에 대해 실제 sentiment label 및 피드백 코멘트 역시 입력
- Model 부분을 더 알고 싶다면 [이 리포지토리](https://github.com/ThreeKcal/model/tree/main) 확인
- Airflow 부분을 더 알고 싶다면 [이 리포지토리](https://github.com/ThreeKcal/dags/tree/main) 확인
- Pyspark 부분을 더 알고 싶다면 [이 리포지토리](https://github.com/ThreeKcal/pyspark/tree/main)  확인
<br></br>
## 목차
- [기술 스택](#기술-ㅅ택)
- [Model Features](#Model-Features)
- [Airflow Features](#Airflow-Features)
- [pyspark Features](#pyspark-Features)
- [Usage](#Usage)
- [개발 관련 사항](#개발-관련-사항)
<br></br>
## 기술 스택
<img src="https://img.shields.io/badge/Python-3.11-3776AB?style=flat&logo=Python&logoColor=F5F7F8"/>  <img src="https://img.shields.io/badge/Spark-3.5.1-E25A1C?style=flat&logo=apachespark&logoColor=F5F7F8"/>  <img src="https://img.shields.io/badge/Airflow-2.7.0-017CEE?style=flat&logo=apacheairflow&logoColor=F5F7F8"/>  <img src="https://img.shields.io/badge/Mariadb-003545?style=flat&logo=mariadb&logoColor=F5F7F8"/>  <img src="https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=F5F7F8"/>  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&logoColor=F5F7F8"/>
<br></br>
## Model Features
### `streamlit` [어플리케이션](http://54.180.132.11:8002/) 시연 모습
- `텍스트 업로드` 페이지: 이용자가 `username`과 `comment`를 입력해 데이터베이스로 전송시킵니다
![text_uploadpage](https://github.com/user-attachments/assets/1099ff86-8491-4002-b375-5f0dbe3e8bfc)

- `코멘트 라벨` 페이지: 전체 혹은 `username` 기준으로 추려낸 코멘트에 관리자가 실제 `label` 값 및 추가 사항을 입력할 수 있습니다
![commentlabelpage](https://github.com/user-attachments/assets/b2c8be3b-54a2-4366-bcf9-5943f40c5569)

- `결과 통계` 페이지: 위 두 페이지를 통해 형성된 데이터베이스에 대한 각종 통계 자료를 볼 수 있습니다. 새로고침할 때마다 새롭게 변경사항을 반영합니다.
![statistic_dynamic](https://github.com/user-attachments/assets/a4f7656e-9a57-46e8-a85b-e6be9c187305)


### Model Structure
![Blank_diagram_-_Page_1_2](https://github.com/user-attachments/assets/2c2cfbd5-fa7e-4cee-858b-57ccb84e6715)

본 어플리케이션은 `fastapi`와 `airflow`, `maradb`, `pyspark`를 필요로 하는 `streamlit` 웹 어플리케이션입니다.

사용자가 우선 `streamlit`의 텍스트 업로드 페이지에서 입력을 보내면, 이는 
1. `streamlit`에서 `fastapi`로 전달되어 `mariadb` 데이터베이스에 저장되고,
2. 이 데이터베이스를 `airflow`가 주기적으로 읽어들여 모델을 적용하고 업데이트합니다.
3. 해당 과정의 로그파일은 `pyspark`로 관리되고,
4. 이렇게 완성된 데이터베이스의 값을 다시 `streamlit`으로 읽어들여 코멘트 입력 및 통계 페이지에서 확인할 수 있는 구조입니다.

데이터베이스 내 테이블은 다음과 같이 형성되어 있습니다.
- `num`: 입력된 각 데이터에 매겨지는 인덱스 번호
- `comments`: 이용자로부터 입력된 코멘트 내용
- `request_user`: 이용자가 입력한 `username`
- `request_time`: 해당 이용자의 입력 요청이 보내진 시각
- `prediction_result`: 모델을 통해 예측한 해당 코멘트의 감정. anger (분노), disgust (경멸), fear (두려움),	joy (기쁨),	neutral (중립),	sadness (슬픔),	surprise (놀람) 의 7가지로 분류됩니다.
- `prediction_score`: 모델이 자체적으로 반환한 예측 스코어 입니다. 본 어플리케이션은 특히 통계 데이터 분석에 사용됩니다.


## Airflow Features
### Airflow Structure
![Blank_diagram_-_Page_1_2](https://github.com/user-attachments/assets/2c2cfbd5-fa7e-4cee-858b-57ccb84e6715)

본 에어플로우 어플리케이션은 `predict.py`, `pyspark_db.py`, `pyspark_pj3.py`로 이루어져 있습니다.

- `prediction.py` : 실제 모델 적용 및 예측을 실행하는 DAG 입니다. 해당 예측 프로세스에 대한 로그파일 역시 본 DAG에서 실행합니다.
![image](https://github.com/user-attachments/assets/dce759a2-cf03-4b02-89e5-e44340c9c44e)

- `pyspark_db.py` : `prediction.py` 의 로그파일이 생성된 후 이를 받아 시간변수를 추가해 `pyspark_pj3.py`로 전송합니다.

- `pyspark_pj3.py` : 전송된 값 및 변수를 기반으로 `pyspark`과 연동, `mariadb` 데이터베이스를 업데이트합니다.

 
### 생성된 에어플로우 로그 파일 디렉토리 및 실제 내부 값
![image](https://github.com/user-attachments/assets/c733df6d-e212-4565-8dfb-28b1963bc901)

![image](https://github.com/user-attachments/assets/982106c8-cfbc-42dc-aadb-9c74c00ac2a9)


## pyspark Features
![pyspark_proj](https://github.com/user-attachments/assets/c678225e-e5c0-4da0-9cac-b8025b5a8a74)

`airflow`로 저장된 `predict.log` 파일을 읽어서 `MariaDB`에 있는 테이블을 업데이트합니다.
상세한 사항은 상단의 `pyspark` 리포지토리의 README를 참조해 주세요.


## Usage
- `fastapi` 서버 런칭
```bash
$ uvicorn src/threekcal_model/api:app --host 0.0.0.0 --port 8001
```

- `steamlit` 서버 런칭
```bash
$ streamlit run src/threekcal_model/streamlit/main.py --server.port 8501
```

- 에어플로우 폴더의 `airflow.cfg` 파일을 수정해 `dags_folder` 값을 본 리포지토리 경로로 바꿉니다.
```bash
# airflow.cfg
#...
[core]
#...
dags_folder=<THIS_REPOSITORY_PATH>
```

## 개발 관련 사항
### 타임라인
![스크린샷 2024-10-10 010952](https://github.com/user-attachments/assets/7bed00cb-272e-49e1-83f4-3986dd6bfcff)

※ 권한이 있는 이용자는 [프로젝트 schedule](https://github.com/orgs/ThreeKcal/projects/1/views/4)에서 확인할 수 있습니다.

### troubleshooting
- 각 리포지토리들의 `issues`, `pull request` 쪽을 참조해 주세요.
