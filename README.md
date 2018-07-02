<p align="center">
<a href="https://github.com/himsR/check/blob/master/logos/layer6ai-logo.png" width="180"></a>
</p>


# ACM RecSys Challenge 2018 Team vl6

Team members: Maksims Volkovs, Himanshu Rai, Zhaoyue Cheng, Yichao Lu, Wu Ga

[Layer6 AI](https://layer6.ai/) and [Vector Institute](https://vectorinstitute.ai/)

## Table of Contents  
0. [Introduction](#intro)  
1. [Environment](#env)
2. [Dataset](#dataset)
2. [Executing](#executing)
4. [Results](#results)

<a name="intro"/>

## Introduction
This repository contains the Java implementation of our entries for both main and creative tracks. Our approach consists of a two-stage model where in the first stage a blend of collaborative filtering methods is used to quickly retrieve a set of candidate songs for each playlist with high recall. Then in the second stage a pairwise playlist-song gradient boosting model is used to re-rank the retrieved candidates and maximize precision at the top of the recommended list.

<a name="env"/>

## Environment
The JAVA code is developed and tested on the following environment:
* Intel(R) Xeon(R) CPU E5-2620 v4 @ 2.10GHz
* 256GB RAM
* Nvidia Titan V
* Java Oracle 1.8.0_171
* Apache Maven 3.3.9
* CUDA 8.0 and CUDNN 8.0
* Intel MKL 2018.1.038
* XGBoost and XGBoost4j 0.7

<a name="dataset"/>

## Dataset
To run the model, download the Data from [here](https://s3.amazonaws.com/public.layer6.ai/RecSys2018/Data.tar.gz)
and dependency jar from [here](https://s3.amazonaws.com/public.layer6.ai/RecSys2018/jar.tar.gz)
and extract them. Note that for consistency we have serialized all relevant objects (including a java-loaded copy of the challenge dataset and our train/valid split) in Data so this file is ~20GB in size. For the creative track we extracted extra song features from the [Spotify Audio API].
(https://developer.spotify.com/documentation/web-api/reference/tracks/get-several-audio-features/). We were able to match most songs from the challenge Million Playlist Dataset, and used the following fields for further feature extraction: `[acousticness, danceability, energy, instrumentalness, key, liveness, loudness, mode, speechiness, tempo, time_signature, valence]`. These additional features are also included in Data.

Once extracted, you should see the following structure for Data and jar:
```
Data
  ├─ recsys2018
  │      └─ models
  │       └─ blend
  │          └─ blend_uu_als_ii_cnn_5K_0.1865_0.3728_1.4064.out
  │       └─ latent_album
  │          └─ album_200.bin
  │          └─ album_U_200.bin
  │          └─ album_V_200.bin
  │       └─ latent_artist
  │          └─ artist_200.bin
  │          └─ artist_U_200.bin
  │          └─ artist_V_200.bin
  │       └─ latent_name
  │          └─ name_200.bin
  │          └─ name_U_200.bin
  │          └─ name_V_200.bin
  │      └─ iatent_song
  │          └─ matching_name_U_200.bin
  │          └─ matching_name_V_200.bin
  │          └─ matching_CNN_v2_U_200.bin
  │          └─ matching_CNN_v2_V_200.bin
  │       └─ xgb
  │          └─ run1
  │              └─ 0150.model.creative
  │              └─ 0150.model.regular
  │    └─ submissions
  │    └─ mpd.parsed
  │    └─ split_recsys_matching.out

```
```
    Jar
     └─  ml_core-cpu_0.0.2-SNAPSHOT-assembly.jar 
```
The `ml_core-cpu_0.0.2-SNAPSHOT-assembly.jar` jar needs to be coppied to:
```
{codebasedir}/src/main/resources/
```

<a name="executing"/>

## Executing
All models are executed from the `src/main/java/recsys2018/maks/XGBModel.java`, the main function has examples on 
how to do main and creative track training, evaluation and submission. Important functions are `extractFeatures2Stage()`, `inference2Stage()` and `submission2Stage()`. We have included pre-trained xgboost models for main (`0150.model.regular`) and creative (`0150.model.creative`) tracks that were used to generate our current submissions on the leader board. To run validation:

* Set the path to extracted Data directory in `XGBModel.java` by setting the `dataPath` variable, for example:
```
String dataPath = "/home/recsys2018";
```

* To run validation for main track:
```
params.doCreative = false;
params.xgbModel = xgbModelPath + "/run1/0150.model.regular";
XGBModel model = new XGBModel(data, params, latents, split, false);
model.inference2Stage();
```
* To run validation for creative track:
```
params.doCreative = true;
params.xgbModel = xgbModelPath + "/run1/0150.model.creative";
XGBModel model = new XGBModel(data, params, latents, split, false);
model.inference2Stage();
```
Replace `inference2Stage()` with `submission2Stage()` to generate the submission for the respective track. To execute with maven once the appropriate code is set:

```
export MAVEN_OPTS="-Xms150g -Xmx150g"
mvn clean install
mvn exec:java -Dexec.mainClass="recsys2018.maks.XGBModel" 
```

Note that for this project we prioritized speed over memory so you'll need at least 100GB of RAM to load data and execute inference.


## Inference Results

You should see the following inference results for main and creative tracks:

```
MAIN
BLEND nEval: 9000, b-ndcg: 0.2539 0.2981 0.3317 0.3553 0.3728
BLEND nEval: 9000, r-precision-artist: 0.2118
BLEND nEval: 9000, r-precision: 0.1866
BLEND nEval: 9000, clicks: 1.4064
2STAGE nEval: 9000, b-ndcg: 0.2661 0.3103 0.3444 0.3674 0.3844
2STAGE nEval: 9000, r-precision-artist: 0.2238
2STAGE nEval: 9000, r-precision: 0.1977
2STAGE nEval: 9000, clicks: 1.2098

```
```
CREATIVE
BLEND nEval: 9000, b-ndcg: 0.2539 0.2981 0.3317 0.3553 0.3728
BLEND nEval: 9000, r-precision-artist: 0.2118
BLEND nEval: 9000, r-precision: 0.1866
BLEND nEval: 9000, clicks: 1.4064
2STAGE nEval: 9000, b-ndcg: 0.2663 0.3107 0.3447 0.3677 0.3847
2STAGE nEval: 9000, r-precision-artist: 0.2239
2STAGE nEval: 9000, r-precision: 0.1978
2STAGE nEval: 9000, clicks: 1.2057

```
These are validation results on our internal validation set which was generated in a similar fashion to the test set. BLEND shows the first stage collaborative filtering candidate generator model accuracy, and 2STAGE shows the gain from the second stage re-ranking. There is a small gain by including Spotify features in the creative track, however, we were unable to achieve significant improvement with any external data.

