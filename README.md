# Qwen2.5-0.5B-Instruct Fine-tuned on ChatDoctor
![image](/assets/project%20img.png)
This repository contains a medical-domain adapter for the **Qwen2.5-0.5B-Instruct** model. The model was fine-tuned using **QLoRA (4-bit quantization)** on the **ChatDoctor-HealthCareMagic-100k** dataset to enhance its ability to provide professional and empathetic medical advice.

## Project Overview
The goal of this project was to adapt a lightweight Small Language Model (SLM) for specialized medical Q&A tasks. By leveraging Parameter-Efficient Fine-Tuning (PEFT), we transformed a general-purpose instruct model into a medical assistant capable of understanding patient symptoms and suggesting potential diagnoses and treatments.

## Model Details
- **Base Model:** [Qwen/Qwen2.5-0.5B-Instruct](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct)
- **Dataset:** [lavita/ChatDoctor-HealthCareMagic-100k](https://huggingface.co/datasets/lavita/ChatDoctor-HealthCareMagic-100k)
- **LoRA Adapter:** [justjuu/qwen2.5-0.5b-chatdoctor-qlora-adapters](https://huggingface.co/justjuu/qwen2.5-0.5b-chatdoctor-qlora-adapters)
- **Fine-tuning Method:** QLoRA (4-bit NF4 quantization)
- **Training Samples:** 39,500 (Subset)
- **Evaluation Samples:** 500

## Hyperparameters
The following hyperparameters were used during training:
- **LoRA Rank (r):** 16
- **LoRA Alpha:** 32
- **LoRA Dropout:** 0.05
- **Learning Rate:** 2e-4
- **Batch Size:** 4
- **Gradient Accumulation Steps:** 4
- **Optimizer:** `paged_adamw_8bit`
- **Max Sequence Length:** 512 tokens
- **Epochs:** 1

## Performance Summary
Fine-tuning resulted in a significant improvement in ROUGE metrics compared to the base model:
- **ROUGE-1:** +0.0400 improvement
- **ROUGE-L:** +0.0402 improvement

For a detailed breakdown of metrics and training plots, see [results.md](./results.md).

## Usage
You can load this adapter using the `peft` library:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# Load base model and tokenizer
base_model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-0.5B-Instruct",
    device_map="auto",
    torch_dtype=torch.bfloat16,
)
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-0.5B-Instruct")

# Load LoRA adapter
model = PeftModel.from_pretrained(base_model, "justjuu/qwen2.5-0.5b-chatdoctor-qlora-adapters")

# Generate response
messages = [
    {"role": "system", "content": "You are a helpful medical assistant."},
    {"role": "user", "content": "What are the symptoms of diabetes?"},
]
prompt = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_new_tokens=256)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

## Disclaimer
This model is for educational purposes only. It is not a substitute for professional medical advice, diagnosis, or treatment. Always consult a qualified healthcare provider for medical concerns.

---