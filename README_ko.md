# HuDiff

## 인간화 과정.
![pipeline](doc/process.svg)


이 패키지는 HuDiff-Ab 및 HuDiff-Nb에 대한 학습 세부 정보 구현과 두 모델에 대한 추론 파이프라인을 제공합니다. 또한 다음을 제공합니다.

1. 마우스 항체를 인간화하도록 설계된 미세 조정된 항체 인간화 확산 모델과 나노바디를 인간화하기 위한 미세 조정된 나노바디 인간화 확산 모델.
2. 논문에서 사전 처리한 학습 세트와 구축한 테스트 세트.

이 소스 코드 또는 모델 매개변수를 사용하여 얻은 결과를 제시하는 모든 출판물은 HuDiff 논문을 인용해야 합니다.

문의 사항은 HuDiff 팀(fm924507@gmail.com)에 문의하십시오.

## Conda 환경
<!-- [mirrors](mirrors.tencent.com/ma_env/antidiff:new)에서 사용 가능한 Docker 환경을 만들었습니다. 환경을 직접 구축하려면 environment.yaml 파일을 사용하여 빌드할 수 있습니다. -->
environment.yaml 파일을 사용하여 환경을 구성할 수 있습니다.
```
conda env create -f environment.yaml
```

## HuDiff-Ab
### 데이터 준비
HuDiff에 대한 페어링된 데이터 세트의 lmdb 파일을 업로드했습니다([Hugging Face](https://huggingface.co/cloud77/HuDiff/tree/main) 참조). 이러한 데이터 세트를 직접 사용하여 HuDiff-Ab 모델을 학습시킬 수 있습니다. 또는 제공하는 다운로드 목록을 확인할 수 있습니다.
### 사전 학습
```
antibody_scripts/antibody_run.sh
```
제공된 bash 스크립트를 실행하기 전에 환경 변수 `config_path`, `data_path` 및 `log_path`를 수정해야 합니다. `data_path`는 제공된 LMDB 파일의 디렉터리 경로로 설정해야 합니다.

### 미세 조정
```
antibody_scripts/antibody_finetune.sh
```
환경 변수 `config_path`, `data_path`, `log_path` 및 `ckpt_path`를 수정해야 합니다. `ckpt_path`는 미세 조정할 체크포인트를 지정해야 합니다. 릴리스된 데이터에는 사전 학습된 체크포인트를 제공하지 않으므로 처음부터 학습해야 합니다. 그러나 미세 조정 후 체크포인트는 제공합니다. `data_path`는 사전 학습 단계와 동일하게 유지되며 제공된 LMDB 파일을 활용합니다.

### 평가
항체 분석을 위해 제공된 코드를 평가하고 실행하려면 다음 단계를 따르십시오.
1. 해당 GitHub 리포지토리([BioPhi](https://github.com/Merck/BioPhi))를 복제하여 필요한 라이브러리 OAsis를 설치합니다.
2. 실험실에서 수집한 특허 데이터 세트의 경우 다음 명령을 실행하여 샘플을 만듭니다.
```
python antibody_scripts/sample.py
--ckpt --ckpt checkpoints/antibody/hudiffab.pt
--data_fpath ./data/antibody_eval_data/HuAb348_data/humanization_pair_data_filter.csv
```
3. 그런 다음 다음 명령을 사용하여 샘플을 평가합니다.
```
python antibody_scripts/patent_eval.py
```
4. 공개된 25쌍 항체 데이터 세트의 경우 다음 명령을 실행합니다.
```
python antibody_scripts/sample.py
--ckpt --ckpt checkpoints/antibody/hudiffab.pt
--data_fpath ./data/antibody_eval_data/Humab25_data/parental_mouse.csv
```
5. 마지막으로 다음 명령을 사용하여 공개 데이터 세트를 평가합니다.
```
python antibody_scripts/humab25_eval.py
```
추정되는 인간화 과정의 경우 평가 스크립트를 제공하지 않지만 위에서 설명한 것과 동일한 단계를 직접 따를 수 있습니다.

### 인간화
항체를 인간화하는 방법에는 항원-항체 서열을 포함하는 복잡한 fasta 파일을 사용하거나 개별 중쇄 및 경쇄 서열을 제공하는 두 가지 방법이 있습니다.
```
python antibody_scripts/sample_for_anti_cdr.py
--ckpt checkpoints/antibody/hudiffab.pt
--anti_complex_fasta data/fasta_file/7k9i.fasta
```
또는 개별 중쇄 및 경쇄 서열 제공
```
python antibody_scripts/sample_for_anti_cdr.py
--ckpt checkpoints/antibody/hudiffab.pt
--heavy_seq HEAVY_SEQUENCE
--light_seq LIGHT_SEQUENCE
```
7k9i.fasta, HEAVY_SEQUENCE 및 LIGHT_SEQUENCE를 각각 해당 사슬 서열로 바꿔야 한다는 점에 유의하십시오.


## HuDiff-Nb
### 데이터 준비
데이터 전처리에서 모든 LMDB 파일을 HuDiff용 [Hugging Face](https://huggingface.co/cloud77/HuDiff/tree/main)에 업로드했습니다. 학습 데이터 세트를 처음부터 처리하려면 중쇄 파일을 다운로드하십시오(다운로드 목록 제공).
### 사전 학습
제공된 명령은 학습을 위해 직접 실행할 수 있습니다. 그러나 사전에 환경 변수를 지정하는 것이 중요합니다.
```
nanobody_scripts/nanotrain_run.sh
```
데이터 파일(`unpair_data_path`)과 구성 파일(`config_path`)의 경로를 모두 수정해야 합니다.
### 미세 조정
사전 학습된 모델을 설정한 후 미세 조정을 위해 선택해야 합니다. YAML 구성 파일을 수정하여 사전 학습된 체크포인트의 경로를 지정합니다. 미세 조정 전에 [AbNatiV](https://gitlab.developers.cam.ac.uk/ch/sormanni/abnativ) 모델을 설치하고 AbNatiV 모델이 구성 YAML 파일에 지정되어 있는지 확인합니다.
```
nanobody_scripts/nanofinetune_run.sh
```
### 평가
이 스크립트는 사용자 지정할 수 있는 다양한 샘플링 방법을 제공합니다(예: 체크포인트를 학습된 모델로 교체).
```
python nanobody_scripts/nanosample.py
--ckpt checkpoints/nanobody/hudiffnb.pt
--data_fpath data/nanobody_eval_data/abnativ_select_vhh.csv
--model pretrain
--inpaint_sample False
```
```
python nanobody_scripts/nanosample.py
--ckpt checkpoints/nanobody/hudiffnb.pt
--data_fpath data/nanobody_eval_data/abnativ_select_vhh.csv
--model finetune_vh
--inpaint_sample True
```
샘플링 후 다음 스크립트를 사용하여 샘플링 결과를 평가할 수 있습니다.
```
python nanobody_scripts/nano_eval.py # 샘플 경로를 지정해야 합니다.
```
### 인간화
나노바디의 fasta 파일을 사용하여 모델은 이를 인간화할 수 있습니다. 더 높은 수준의 인간화가 필요한 경우 배치 크기 또는 샘플링 횟수를 늘리는 것을 고려하십시오.
```
python nanobody_scripts/sample_for_nano_cdr.py
--ckpt checkpoints/nanobody/hudiffnb.pt
--nano_complex_fasta data/fasta_file/7x2l.fasta
--model finetune_vh
--inpaint_sample True
```

# HuDiff 인용
연구에 HuDiff를 사용하는 경우 논문을 인용하십시오.
```BibTex
@article{ma2024adaptive,
title={An adaptive autoregressive diffusion approach to design active humanized antibody and nanobody},
author={Ma, Jian and Wu, Fandi and Xu, Tingyang and Xu, Shaoyong and Liu, Wei and Yan, Divin and Bai, Qifeng and Yao, Jianhua},
journal={bioRxiv},
year={2024},
publisher={Cold Spring Harbor Laboratory}
}
```
