# Microbiome-Analysis

## Index
```
0. Introduction
1. Qiime2 Analysis
  1.0 Setup Qiime2
  1.1 Analysis Environment Setup
  1.2 Import raw data to QIIME2
  1.3 DADA2 Denoising and ASV Processing
  1.4 Feature Table and ASVs Summary
  1.5 Taxonomic Classification of Observed ASVs
2. Unspecified Taxonomy Reassignment
```

<br/>

## 1. Qiime2 Analysis
### 1.0 Setup Qiime2
#### 1.0.1 Qiime2 설치
```bash
wget https://data.qiime2.org/distro/amplicon/qiime2-amplicon-2023.9-py38-linux-conda.yml
```
#### 1.0.2 Conda 환경 생성 
```bash
conda env create -n qiime2-amplicon-2023.9 --file qiime2-amplicon-2023.9-py38-linux-conda.yml
```
#### 1.0.3 Conda 환경 활성화
```bash
conda activate qiime2-amplicon-2023.9
```
- Qiime2 실행할 때마다 conda의 Qiime2 환경 활성화시켜야 함

#### 1.0.4 Qiime2 설치 확인
```bash
qiime --help
```
- 위 커맨드 실행 시, qiime 사용 설명이 나오면 잘 설치가 된 것

<br/>

### 1.1 Analysis Environment Setup
#### 1.1.1 Set Working Directory
```bash
mkdir beetle_microbiome
cd beetle_microbiome
```

#### 1.1.2 Download the Raw data
```bash
mkdir raw_data
cd raw_data
```
```bash
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium5.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium5.R2.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium6.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium6.R2.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium7.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium7.R2.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium8.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Intestine-microbium8.R2.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Treecre-microbium1.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Treecre-microbium1.R2.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Treecre-microbium2.R1.fq.gz
wget http://175.207.144.135/SD/2024HC079_20240623142430/Treecre-microbium2.R2.fq.gz
```
- raw_data 디렉토리에 raw data 다운로드 받기

#### 1.1.3 Make Meatadata
```bash
vi sample-metadata.tsv
```
```
SampleId	DevelopmentalStage	Environment Feed
Intestine-microbium5	5th instar	artificial	sawdust
Intestine-microbium6	6th instar	artificial	sawdust
Intestine-microbium7	7th instar	artificial	sawdust
Intestine-microbium8	8th instar	artificial	sawdust
Treecre-microbium1  8th instar	nature	oak tree
Treecre-microbium2	8th instar	natrue	oak tree
```
- Qiime2에서 metadata를 인식하기 위해서는 SampleID 컬럼은 필수적으로 들어가야함
- 추가로 Qiime2에서 인식 가능한 카테고리 정보를 분석 목적에 맞게 적절히 지정

<br/>

### 1.2 Import raw data to QIIME2
#### 1.2.1 Qiime2에 raw data 불러오기
```bash
qiime tools import \
--type 'SampleData[PairedEndSequencesWithQuality]' \
--input-format CasavaOneEightSingleLanePerSampleDirFmt \
--input-path raw_data \
--output-path demultiplexed-sequences.qza
```
- 위 커맨드 실행 시 아래 에러가 발생하면 **1.4.2** 방법으로 raw data import 하기
<br/>
:warning:
<br/>
  ```
    There was a problem importing raw_data:

    Missing one or more files for CasavaOneEightSingleLanePerSampleDirFmt: '.+_.+_L[0-9][0-9][0-9]_R[12]_001\\.fastq\\.gz'
  ```
- 이 에러 메세지는 Qiime2에서 Casava 1.8 형식의 파일을 찾을 수 없거나, 파일명이 올바르지 않은 경우 발생

#### 1.2.2 Manifest 파일로 raw 데이터 불러오기
1. Manifest 파일 생성
- Manifest 파일은 각 샘플의 ID, forward read 파일 경로, reverse read 파일 경로를 포함
```bash
vi manifest.tsv
```
```bash
sample-id	forward-absolute-filepath	reverse-absolute-filepath
Intestine-microbium5	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium5.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium5.R2.fq.gz
Intestine-microbium6	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium6.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium6.R2.fq.gz
Intestine-microbium7	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium7.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium7.R2.fq.gz
Intestine-microbium8	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium8.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Intestine-microbium8.R2.fq.gz
Treecre-microbium1	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Treecre-microbium1.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Treecre-microbium1.R2.fq.gz
Treecre-microbium2	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Treecre-microbium2.R1.fq.gz	/home/biotech/BI_2024/Beetle_Microbiome/Group2/Qiime2/raw_data/Treecre-microbium2.R2.fq.gz
```
```bash
:wq
```
2. Manifest 파일로 raw data 불러오기
```bash
qiime tools import \
  --type 'SampleData[PairedEndSequencesWithQuality]' \
  --input-path manifest.tsv \
  --output-path demultiplexed-sequences.qza \
  --input-format PairedEndFastqManifestPhred33V2
```
- manifest 파일에서 forward read 파일 경로, reverse read 파일 경로를 명시하고 있으므로, **PairedEnd** input format을 사용

#### 1.2.3 Raw data 요약 및 시각화
```bash
qiime demux summarize \
--i-data demultiplexed-sequences.qza \
--o-visualization V1_demultiplexed-sequences-summ.qzv
```
- 생성된 시각화 파일 ```.qzv```는 [Qiime2 View](https://view.qiime2.org/)에서 확인 가능
- ```V1_demultiplexed-sequences-summ.qzv``` 파일에서 샘플별 시퀀스 개수, 시퀀스 길이 분포, 품질 점수를 확인할 수 있음
- 이 파일에서 확인한 Quality Score Plot으로 이후 진행할 denoising 과정의 trimming 변수값 조정
![Image](https://github.com/user-attachments/assets/bbfbf923-ba91-4f1f-8d77-81c96ddecf24)

<br/>

### 1.3 DADA2 Denoising and ASV Processing
#### 1.3.1 DADA2 Denoising
```bash
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demultiplexed-sequences.qza \
  --p-trunc-len-f 250 \
  --p-trim-left-f 20 \
  --p-trunc-len-r 220 \
  --p-trim-left-r 10 \
  --o-representative-sequences asv-sequences.qza \
  --o-table feature-table.qza \
  --o-denoising-stats dada2-stats.qza
```
|옵션|기본값|설명|
|------|---|---|
|--p-trunc-len-f/r|필수 입력|Forward/Reverse reads의 3'말단(오른쪽)에서 잘라낼 길이|
|--p-trim-left-f/r|0|Forward/Reverse reads의 5'말단(왼쪽)에서 잘라낼 길이|
|--p-max-ee-f/r|2.0|Forward/Reverse reads의 허용 가능한 최대 예상 오류|
|--p-min-overlap|12|Forward/Reverse reads 병합 시 최소 오버랩 길이|
|--p-min-fold-parent-over-abundance|1.0|키메라 후보 서열과 부모 서열 간 최소 풍부도 비율
|--p-n-threads|1|사용할 CPU 스레드 수|
- **--p-trunc-len** 옵션은 ```V1_demultiplexed-sequences-summ.qzv```의 Quality 그래프 확인 후, Quality Score가 30 이하로 떨어지는 bp로 지정
- **--p-trim-left** 옵션은 read 앞쪽에 제거되지 않고 남아있을 가능성이 있는 프라이머나 어댑터 등을 제거하기 위함
- **-p-trunc-len**와 **--p-trim-left**로 트리밍된 forward와 reverse reads의 길이가 
- **-p-max-ee** 옵션은 값을 낮추면 더 엄격한 품질 필터링, 보통 1.0~2.0 사이 값 사용
- **p-min-overlap** 옵션은 값을 낮추면 overlap 길이가 짧아도 병합이 가능하게 함
- **p-min-fold-parent-over-abundance** 옵션은 값을 낮추면 Chimeric reads를 더 엄격하게 제거함

#### 1.3.1 DADA2 denoising 결과 시각화
```bash
qiime metadata tabulate \
  --m-input-file dada2-stats.qza \
  --o-visualization dada2-stats-summ.qzv
```

<br/>

### 1.4 Feature Table and ASVs Summary
#### 1.4.1 Feature Table 요약 및 시각화
```bash
qiime feature-table summarize \
--i-table feature-table.qza \
--m-sample-metadata-file sample-metadata.tsv \
--o-visualization feature-table-summ.qzv
```

#### 1.4.2 ASV representative 서열 요약 및 시각화
```bash
qiime feature-table tabulate-seqs \
--i-data asv-sequences.qza \
--o-visualization asv-sequences-summ.qzv
```

<br/>

### 1.5 Taxonomic Classification of Observed ASVs
#### 1.5.1 SILVA classifier 다운로드
```bash
wget \
  -O silva-138-99-515-806-nb-classifier.qza \
  https://data.qiime2.org/2023.7/common/silva-138-99-515-806-nb-classifier.qza
```
- 16S rRNA Taxonomic Classifier
  - SIVLA (silva-138-99-515-806-nb-classifier.qza)
  - Greengens (gg-13-8-99-nb-classifier.qza)
- ITS Taxonomic Classifier
  - UNITE Database (unite-ver8-99-classifier.qza)

#### 1.5.2 SILVA classifier를 이용한 Taxonomy 할당
```bash
qiime feature-classifier classify-sklearn \
  --i-classifier silva-138-99-515-806-nb-classifier.qza \
  --i-reads asv-sequences.qza \
  --o-classification silva-taxonomy.qza
```

#### 1.5.3 Taxonomy Classification 결과 요약 및 시각화
```bash
qiime metadata tabulate \
  --m-input-file silva-taxonomy.qza \
  --o-visualization silva-taxonomy.qzv
```

#### 1.5.4 SILVA 분류 결과를 기반으로 Feature Table 필터링
```bash
qiime taxa filter-table \
  --i-table feature-table.qza \
  --i-taxonomy silva-taxonomy.qza \
  --p-mode contains \
  --p-include p__ \
  --p-exclude 'p__;,Chloroplast,Mitochondria' \
  --o-filtered-table silva-filtered-table-01.qza
```
- **--p-include p__** : 포함할 taxonomic 그룹 지정, __p는 모든 분류군을 의미
- **--p-exclude 'p__;,Chloroplast,Mitochondria'** : 엽록체, 미토콘드리아 서열 제거

<br/>

### 1.6 Filtering samples with low sequence counts
```bash
qiime feature-table filter-samples \
--i-table silva-filtered-table-01.qza \
--p-min-frequency 10000 \
--o-filtered-table silva-filtered-table-02.qza
```
```bash
qiime feature-table filter-seqs \
--i-data asv-sequences.qza \
--i-table silva-filtered-table-02.qza \
--o-filtered-data asv-filtered-sequences.qza
```
<br/>

### 1.7 Generate taxonomic composition barplots
```bash
qiime taxa barplot \
--i-table silva-filtered-table-02.qza \
--i-taxonomy silva-taxonomy.qza \
--m-metadata-file sample-metadata.tsv \
--o-visualization plot_output/taxa-bar-plots.qzv
```
