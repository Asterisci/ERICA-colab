# ERICA-colab

The repo is forked from [thunlp/ERICA](https://github.com/thunlp/ERICA), and then change some wrong codes and adjust codes for using in [google colab](https://colab.research.google.com/drive/1x0ZQF8KZqSczzLCEt3xXkbpWLWmzOgBW?usp=sharing) and other online jupyter notebook.

And this is my blog about ERICA: [ERICA-提升预训练语言模型实体与关系理解的统一框架解读](https://www.asteriscum.cn/2021/07/17/31/59/)

Below is raw repo introduction about ERICA：

# ERICA

Source code and dataset for ACL2021 paper: "[ERICA: Improving Entity and Relation Understanding for Pre-trained Language Models via Contrastive Learning](https://arxiv.org/abs/2012.15022)".

The code is based on huggingface's [transformers](https://github.com/huggingface/transformers), the trained models and pre-training data can be downloaded from [Google Drive](https://drive.google.com/drive/folders/19SxYoDeKZg4Ho_FIrDYpcifCtpsl5u3K?usp=sharing).

### Quick Start

You can quickly run our code by following steps:

- Install dependencies as described in following section. 
- cd to `pretrain` or `finetune` directory then download and pre-process data for pre-training or finetuning.    

### 1. Dependencies

Run the following script to install dependencies.

```shell
pip install -r requirement.txt
```

**You need to install transformers and apex manually.**

**transformers**
We use huggingface transformers to implement Bert and RoBERTa, and the version is 2.5.0. For convenience, we have downloaded transformers into `code/pretrain/` so you can easily import it, and we have also modified some lines in the class `BertForMaskedLM` in `src/transformers/modeling_bert.py` while keeping the other codes unchanged.

You just need run 
```
pip install .
```
to install transformers manually.

**apex**
Install [apex](https://github.com/NVIDIA/apex) under the offical guidance.

### process pretraining data
In folder prepare_pretrain_data, we provide the codes for processing pre-training data.

### 2. Pretraining

To pretrain ERICA_bert:

```shell
cd code/pretrain

python -m torch.distributed.launch --nproc_per_node 8  main.py  \
    --model DOC  --lr 3e-5 --batch_size_per_gpu 16 --max_epoch 105  \
    --gradient_accumulation_steps 16    --save_step 500  --temperature 0.05  \
    --train_sample  --save_dir ckpt_doc_dw_f_alpha_1_uncased --n_gpu 8  --debug 1  --add_none 1 \
    --alpha 1 --flow 0 --dataset_name none.json  --wiki_loss 1 --doc_loss 1 \
    --change_dataset 1  --start_end_token 0 --bert_model bert \
    --pretraining_size -1 --ablation 0 --cased 0
```

some explanations for hyper-parameters: temperature (\tau used in loss function of contrastive learning); debug (whether to debug (we provide an example_debug file for pre-training); add_none (whether to add no_relation pair in RD loss); alpha (the proportion of masking (1 means no masking, in experiments, we find masking is not helpful as is described in the main paper, so for all models, we do not mask in the pre-training phase. However, we leave this function here for further research explorations.)); flow (if masking, whether to use a linear decay); wiki_loss (whether to add ED loss); doc_loss (whether to add RD loss); start_end_token (use another entity encoding method); cased (whether to use cased version of BERT).

### 3. Fine-tuning

Enter each folder for downstream task (document-level / sentence-level relation extraction, entity typing and question answering) fine-tuning. Before fine-tuning, we assume you have already pre-trained an ERICA model. Excecute the bash in each folder for reimplementation.
