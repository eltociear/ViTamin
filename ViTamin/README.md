# ViTamin pre-training in CLIP

This folder contains the implementation of the ViTamin pre-training using OpenCLIP.

## 🔥 Model Zoo
Here we provide 14 best-performing models. See [MODEL_HUB.md](./ViTamin/MODEL_HUB.md) for the additional 48 benchmarked short-schedule models.

| image encoder | image size | num patches | text encoder depth/width | seen samples (B) | params Image+Text (M) | MACs Image+Text (G) | ImageNet Acc. | avg. 38 datasets | download |
|---------------|------------|-------------|--------------------------|-------------------|----------------------------------|----------------------|---------------|------------------|-----------------------|
| ViTamin-S     | 224        | 196         | 12/384                   | 1.28              | 22.0+40.4                       | 5.50+1.64            | 62.2          | 53.2             | [[checkpoint]](google.com)                |
| ViTamin-S-LTT  | 224       | 196         | 12/384                   | 1.28              | 22.0+40.4                       | 5.50+1.64            | 63.4          |54.6              | [[checkpoint]](google.com)                | 
| ViTamin-B     | 224        | 196         | 12/512                   | 1.28              | 87.5+63.4                       | 21.8+2.9             | 68.9          | 57.7             | [[checkpoint]](google.com)                  |
| ViTamin-B-LTT  | 224       | 196         | 12/512                   | 1.28              | 87.5+63.4                       | 21.8+2.9             | 70.8          | 59.4             | [[checkpoint]](google.com)                  |
| ViTamin-L     | 224        | 196         | 12/768                   | 12.8              | 333.3+123.7                     | 72.6+6.6             | 80.8          | 66.7             | [[checkpoint]](google.com)                  |
| ViTamin-L     | 256        | 256         | 12/768                   | 12.8+0.2          | 333.4+123.7                     | 94.8+6.6             | 81.2          | 67.0             | [[checkpoint]](google.com)                  | 
| ViTamin-L     | 336        | 441         | 12/768                   | 12.8+0.2          | 333.6+123.7                     | 163.4+6.6            | 81.6          | 67.0             | [[checkpoint]](google.com)                  | 
| ViTamin-L     | 384        | 576         | 12/768                   | 12.8+0.2          | 333.7+123.7                     | 213.4+6.6            | 81.8          | 67.2             | [[checkpoint]](google.com)                  | 
| ViTamin-L2    | 224        | 196         | 24/1024                  | 12.8              | 333.6+354.0                     | 72.6+23.3            | 80.9          | 66.4             | [[checkpoint]](google.com)                  | 
| ViTamin-L2    | 256        | 256         | 24/1024                  | 12.8+0.5          | 333.6+354.0                     | 94.8+23.3            | 81.5          | 67.4             | [[checkpoint]](google.com)                  | 
| ViTamin-L2    | 336        | 441         | 24/1024                  | 12.8+0.5          | 333.8+354.0                     | 163.4+23.3           | 81.8          | 67.8             | [[checkpoint]](google.com)                  | 
| ViTamin-L2    | 384        | 576         | 24/1024                  | 12.8+0.5          | 334.0+354.0                     | 213.4+23.3           | 82.1          | 68.1             | [[checkpoint]](google.com)                  | 
| ViTamin-XL    | 256        | 256         | 27/1152                  | 12.8+0.5          | 436.1+488.7                     | 125.3+33.1           | 81.9          | 67.7             | [[checkpoint]](google.com)                  | 
| ViTamin-XL    | 384        | 576         | 27/1152                  | 12.8+0.5          | 436.1+488.7                     | 125.3+33.1           | 82.6          | 68.1             | [[checkpoint]](google.com)                  | 
## Usage
### Install

- Clone this repository:

```bash
git clone https://github.com/Beckschen/ViTamin.git
cd ViTamin
```

- Create a conda virtual environment and activate it:

```bash
conda create -n vitamin python=3.9 -y
conda activate vitamin
```

- Install `PyTorch` and `torchvision`; We recommend using the  `PyTorch>=2.1.0`  with `CUDA>=12.1`.


```bash
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

- Install 'OpenCLIP' requirements

```bash
pip3 install 'open_clip_torch[training]'
```

### Data preparation

We trained models on the [DataComp-1B](https://github.com/mlfoundations/datacomp). Please follow the instruction of [DataComp-1B](https://github.com/mlfoundations/datacomp) to download and prepare the data. 

We can set up the enviroment using vitamin/setup_eval.sh and run the evaluation with vitamin/eval.sh

### Training
Here is an example to use 8 GPUs to train ViTamin-B use 128 seen sample on 14M data size.

```bash
cd vitamin
torchrun --nnodes=1 --nproc_per_node=8 --master_addr=127.0.0.1 --master_port=9999 --node_rank=0 \
        -m training.main \
        --config='./configs/vitamin_base_d14ms128m_bs8k_lr5e4.yaml'
```



### Evaluation

To evaluate a pre-trained `ViTamin` on zero-shot classification and retrieval tasks, see the folder `datacomp` for details

- Setup enviroment:
```bash
bash setup_eval.sh
```

- Download 38 test set:
```bash
cd vitamin/datacomp
python3 download_evalsets.py
```

- Run evaluation
```bash
bash eval.sh
```

### Pretrained Model Interface

We offer a simple model interface to instantiate both pre-trained and untrained models.

- build the local open_clip first
```bash
cd open_clip
python3 setup.py install
```
- play the interface
```python
>>> import open_clip
>>> model, _, preprocess = open_clip.create_model_and_transforms('ViTamin-L2', pretrained='~/vitmain_l_datacomp1b_s13b_b90k.bin')
```

### Huggingface

We support huggingface interface.

```python
import torch
from PIL import Image
from transformers import AutoModel, CLIPImageProcessor

model = AutoModel.from_pretrained(
    'ViTamin/ViTamin-L-224px',
    torch_dtype=torch.bfloat16,
    low_cpu_mem_usage=True,
    trust_remote_code=True).cuda().eval()

image = Image.open('./vitaminicon.png').convert('RGB')

image_processor = CLIPImageProcessor.from_pretrained('ViTamin/ViTamin-L-224px')

pixel_values = image_processor(images=image, return_tensors='pt').pixel_values
pixel_values = pixel_values.to(torch.bfloat16).cuda()

outputs = model(pixel_values)
```

## Citing ViTamin

```
@inproceedings{chen2024vitamin,
  title={ViTamin: Design Scalable Vision Models in the Vision-language Era},
  author={Chen, Jieneng and Yu, Qihang and Shen, Xiaohui and Yuille, ALan and Chen, Liang-Chieh},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year={2024}
}
```