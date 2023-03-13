---
layout: post
hero-bg-color: "#000"
title: "Using ANTs package on Google Colaboratory"
date: 2019-03-28
category: Misc
github: Brainy
thumb: "ANTs.png"

# subtitle: ""
# worktype: ""
summary: "I couldn't get ANTs to install on the Colab environment. There are not many helpful articles on the internet, and the existing pre-built binaries were too outdated. After some iterations, I was able to build ANTs from the source on Colab itself."
# progress: 100
---

> Click [Here](#Binaries) to jump directly to the pre-built binaries (Link updated on: 13-03-2023)

## A Primer on ANTs and Google Colaboratory
**[Advanced Normalization Tools](http://stnava.github.io/ANTs/)** (or simply, _ANTs_) is one of the few packages available in the domain of brain MRI study. It is _the_ state-of-the-art package currently available for various tasks related to brain MRI scans like registration, normalization and bias field correction (including both N3 and N4 bias field correction). Though the package hasn't been updated with a major release in a while, it works pretty well on PCs.

**[Google Colaboratory](https://colab.research.google.com)** (or simply, _Colab_) has quickly risen to become one of the most commonly used platforms by machine learning researchers and students all over the globe. Through Colab, Google provides users free access to cloud resources for machine learning. With a Google account, you get access to a Google's version of IPython Notebook backed by either CPU, GPU, or even TPU runtime (you get to choose).

I am currently working on a project dubbed _[Brainy](https://github.com/IAmSuyogJadhav/Brainy)_ which is based on brain tumor segmentation from MRI scans of the patient's brain. At the time of writing this article, I couldn't get ANTs to install on the Colab environment. There are not many helpful articles on the internet, and the existing pre-built binaries were too outdated. After some iterations, I was able to build ANTs from the source on Colab itself. The binaries included in the next section were built specifically for Colab and should work fine on Colab. However, if you run into an error, do let me know by raising an issue [here](https://github.com/IAmSuyogJadhav/Brainy/issues/new).

<a name="Binaries"></a>
## Pre-Built Binaries
I will try to keep this section updated by posting a new binary based on the latest master branch of [ANTsX/ANTs](https://github.com/ANTsX/ANTs) after a major release.

- [ANTs-28-03-2019](https://drive.google.com/file/d/1JwiDXv1plwWXCc6kM_XG5VaI-BYwy26K/view?usp=sharing)

<!-- - [ANTs-28-03-2019](https://drive.google.com/file/d/1N1Qx-R5tLCX5EhXPoPdyg6YvEkDtf-cD/view?usp=sharing) -->

## Installation Instructions
1. Open the latest build from above links. It is a Google Drive link. Add the `.7z` file to your drive by clicking the _Add to Drive_ icon.
2. Open a new Colab notebook.
3. Mount the drive by running the following cell.
```python
from google.colab import drive
drive.mount('/gdrive')
```
4. Drive is now mounted at `/gdrive`. Copy the archive from your drive using `cp` and extract it using `7z`.
```bash
!cp '/gdrive/My Drive/ANTS_built_for_colab.7z' ./
!7z x ANTS_<dd-mm-yyyy>.7z
```
5. Now copy the entire contents of the newly created `bin` folder to `/usr/local/bin/`
```bash
!cp bin/* /usr/local/bin
```
6. Test the installation by running:
```bash
!which antsRegistration
```
it should output full path to antsRegistration (which should be `/usr/local/antsRegistration` in our case)

That's it! If you face any errors, [please let me know](https://github.com/IAmSuyogJadhav/Brainy/issues/new).

## Building on your own
If you are curious to know the steps to build ANTs on Colab, view the notebook used to build the above binaries by clicking the button below. Keep in mind that it takes a LOT of time to compile. If you build a more recent version than available above, hit me up and I will add it above.

<a href="https://colab.research.google.com/drive/1mCRy-A4qve8QaYmYRWjKR2MyycrloDEt#scrollTo=kZA_FAp1BOM-" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>
