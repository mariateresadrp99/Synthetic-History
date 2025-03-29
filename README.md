## Code for "Synthetic History: Evaluating Visual Representations of the Past in Diffusion Models"

![Evaluation Methodology](./evalutation_methodology.png)

Text-To-Image (TTI) models have become powerful tools for artistic creation and design. However, while existing research predominantly examines their embedded demographic and cultural biases, their ability to accurately represent historical contexts remains largely unexplored. In this work, we introduce a systematic and reproducible methodology for evaluating how TTI systems depict different historical periods across multiple dimensions. Our approach is grounded in HistVis, a curated dataset of 30,000 synthetic images generated by three state-of-the-art diffusion models using carefully designed prompts that depict universal human activities across diverse historical contexts. We evaluate historical depiction along three key dimensions: (1) Implicit Stylistic Associations: examining how models default to certain visual styles for specific periods; 
(2) Historical Consistency: detecting anachronisms such as the depiction of modern objects in historical scenes; and (3) Demographic Representation: comparing generated racial and gender distributions against historically plausible baselines derived from Large Language Models. We find that TTI models frequently stereotype past eras by adding visual stylistic properties not defined in the prompt, while also introducing anachronisms at notable rates and failing to reflect historically plausible demographic patterns. By providing a structured evaluation methodology and empirical insights, this work highlights critical gaps in the historical reasoning of TTI models. We release both the HistVis dataset and the accompanying tools needed to replicate our analysis and support the evaluation of additional TTI systems, laying the foundation for more historically responsible generative models.


## A. The HistVis Dataset

### A1. Dataset Overview
The HistVis dataset consists of 30,000 synthetic images generated from prompts describing 100 universal human activities across 10 historical time periods using three state-of-the-art diffusion models (Stable Diffusion XL, Stable Diffusion 3, FLUX.1-schnell). Each prompt follows the format "A person [activity] in the [historical period]", combining 100 activities (drawn from 20 domains such as art, work, celebration, and communication) with five centuries (17th–21st) and five 20th-century decades (1910s, 1930s, 1950s, 1970s, 1990s). For each activity–period pair, 10 images were generated per model. All images are annotated with the associated prompt, activity category, time period, and model identifier.



### A2. Dataset Access
The dataset is publicly available on [Hugging Face](https://huggingface.co/datasets/mariateresadrp/HistVis) and can be downloaded using `huggingface_hub`:

```python
from huggingface_hub import snapshot_download

dataset_path = snapshot_download(
    repo_id="mariateresadrp/HistVis",
    repo_type="dataset"
)

```
## Licence
The HistVis dataset is released under the Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0) license.

## B. Evaluation Methods

### B1. Visual Style Prediction

This module predicts visual styles in images generated by TTI models. We use a VGG16-based classifier fine-tuned on six style categories: drawings, engravings, illustrations, paintings, photography (color or b&w). The script also computes a colorfulness score to help distinguish between black-and-white and color photography.

## Files

- `predict_visual_style.py`: Main script for running style prediction
- `utils.py`: Helper functions

## Model Weights

The pretrained model is available on [Hugging Face](https://huggingface.co/mariateresadrp/visual_style_predictor) and can be downloaded using `huggingface_hub`:

```python
from huggingface_hub import hf_hub_download
from tensorflow.keras.models import load_model

model_path = hf_hub_download(
    repo_id="mariateresadrp/visual_style_predictor",
    filename="best_vgg16_only_last.keras"
)

model = load_model(model_path)

```

### B2. Anachronism Detection 

This module detects and analyzes anachronisms in synthetic images generated by text-to-image (TTI) models. It combines LLM-generated question prompts with vision-language model (VLM) analysis and computes anachronism frequency & severity metrics.
## Files

- `llm_anachronism_proposal.py`: Given a prompt with a historical condition, an LLM (GPT-4o) proposes potential anachronisms in the generated images, as well as identification questions for a VLM model.
- `anachronism_detection.py`: Runs anachronism detection using a VLM (GPT-4 Turbo).
- `compute_anachr_freq_and_sever.py`: Compute anachronism frequency and severity metrics.
- `19th_century.json`: Includes a sample JSON input with prompts and questions for the VLM (GPT-4 Turbo).

## Usage
```bash

# LLM Anachronism Detection
python generate_anachronism_questions.py \
  --input_txt path/to/prompts.txt \

# VLM Anachronism Detection
python detect_anachronisms.py \
  --image_root path/to/images \
  --json_file path/to/prompts.json \
  --output_file results.json

# Compute Frequency and Severity Metrics
python analyze_anachronisms.py \
  --json_path results.json \
  --output_csv anachronism_stats.csv
```

### B3. Demographic Representation

This method evaluates whether TTI models generate historically plausible demographic distributions in their outputs, focusing on race and gender. Rather than assuming modern demographic expectations, we compare generated outputs against contextual historical estimates provided by an LLM (GPT-4o). We then compute over- and underrepresentation scores by comparing model outputs with LLM predictions.

## Files: 
- `generate_demographic_estimates.py`: Uses an LLM (GPT-4o) to infer plausible demographic ratios (gender and race) for each prompt
- `evaluate_demographic_alignment.py`: Compares FairFace-predicted outputs to LLM predictions and computes over-/underrepresentation metrics

## Usage:
```bash
# Generate historical demographic estimates from prompts
python generate_demographic_estimates.py \
  --input_txt path/to/prompts.txt \
  --output_json llm_demographics.json

#  Evaluate demographic alignment
python evaluate_demographic_alignment.py \
  --model_outputs_csv fairface_outputs.csv \
  --llm_json llm_demographics.json \
  --output_csv demographic_metrics.csv



