# TrustyAI Detoxify
Detoxifying large language models is challenging. Training data is usually scraped from the internet which often contains toxic content. Without proper guardrails, the model can learn undesirable properties and in turn generate toxic text. Filtering out toxic content from training data can be expensive as it usually requires data labelers to identify samples that align with human values. We aim to lower the costs detoxifying LLMs during training by using [TrustyAI Detoxify](https://github.com/trustyai-explainability/trustyai-explainability-python/tree/main/src/trustyai/language/detoxify), a library that rephrases toxic text, in conjuction with HuggingFace's [SFT Trainer](https://huggingface.co/docs/trl/en/sft_trainer) or Supervised Fine-Tuning Trainer. HuggingFace's SFT Trainer streamlines the process of finetuning models and allows for effecient memory usage. Training LLMs are memory-intensive and inaccessible for users with consumer hardware. We can reduce memory usage by using QLoRA, an  efficient finetuning technique allows us to compress a model through 4-bit quantization. SFT Trainer already has built-in integrations for training a model using QLoRA, making memory and resource efficient training accessible with only a few lines of code.

The `/notebooks` directory contains Jupyter notebooks that demonstrate an end-to-end example from model training to deployment, using [facebook/opt-350m](https://huggingface.co/facebook/opt-350m). In `1-sft.ipynb` and  `2-eval.ipynb`, we compare the model using two different data preprocessing and training approaches, the first being full finetuning *without detoxifying* the training data and the second, supervised finetuning *with detoxification*. In `3-save_convert-model.ipynb` and `4-inference-request.ipynb`, we prepare the detoxed model for deployment onto a Caikit-TGIS Serving Runtime and then make an inference request.

## Repository Structure
```
├── manifests/
|
├── notebooks
|      ├── 1-sft.ipynb <- Train models with and without detoxification
|      ├── 2-eval.ipynb <- Comparison of "toxic" and "detoxed" models
|      ├── 3-save_convert_model.ipynb <- Save model to S3 storage
│      ├── 4-inference_request.ipynb <- Send inference request to model
|      ├── instructions.md
|      └── requirements.txt
|
├── slides/ # background information on tech stack
|
├── instructions.md
|
├── .gitignore
|
└── README.md <- You are here
```

## References
[QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314)


