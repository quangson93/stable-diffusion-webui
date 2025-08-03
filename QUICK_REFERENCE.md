# Stable Diffusion Web UI - Quick Reference

## REST API Endpoints

### Core Generation
- **POST** `/sdapi/v1/txt2img` - Generate images from text
- **POST** `/sdapi/v1/img2img` - Generate images from images
- **POST** `/sdapi/v1/extra-single-image` - Upscale/enhance single image
- **POST** `/sdapi/v1/extra-batch-images` - Process multiple images

### Information
- **GET** `/sdapi/v1/progress` - Get generation progress
- **POST** `/sdapi/v1/png-info` - Extract image metadata
- **POST** `/sdapi/v1/interrogate` - Generate prompts from images

### Models & Configuration
- **GET** `/sdapi/v1/sd-models` - List available models
- **GET** `/sdapi/v1/samplers` - List samplers
- **GET** `/sdapi/v1/upscalers` - List upscalers
- **GET** `/sdapi/v1/options` - Get settings
- **POST** `/sdapi/v1/options` - Update settings

### Control
- **POST** `/sdapi/v1/interrupt` - Stop generation
- **POST** `/sdapi/v1/skip` - Skip current image

## Python API Quick Examples

### Text-to-Image
```python
from modules.processing import StableDiffusionProcessingTxt2Img, process_images
from modules import shared

p = StableDiffusionProcessingTxt2Img(
    prompt="a cat",
    steps=20,
    width=512,
    height=512,
    sampler_name="Euler"
)
processed = process_images(p)
```

### Image-to-Image
```python
from modules.processing import StableDiffusionProcessingImg2Img, process_images
from PIL import Image

init_image = Image.open("input.jpg")
p = StableDiffusionProcessingImg2Img(
    prompt="make it blue",
    init_images=[init_image],
    denoising_strength=0.7
)
processed = process_images(p)
```

### Save Images
```python
from modules import images

images.save_image(
    image=pil_image,
    path="output",
    basename="generated",
    seed=12345,
    prompt="prompt text"
)
```

## JavaScript API Quick Examples

### Navigation
```javascript
switch_to_txt2img();
switch_to_img2img();
switch_to_extras();
```

### Gallery
```javascript
var selected = selected_gallery_button();
var index = selected_gallery_index();
var buttons = all_gallery_buttons();
```

### Form Submission
```javascript
submit(args);           // txt2img
submit_img2img(args);   // img2img
```

### Utilities
```javascript
setRandomSeed("txt2img_seed");
switchWidthHeight("txt2img");
updateInput(input_element);
```

## Common Settings

### Command Line
```bash
python webui.py --port 7860 --listen --api --xformers
```

### Python Settings
```python
from modules import shared

# Output paths
shared.opts.outdir_txt2img_samples = "outputs/txt2img-images"
shared.opts.outdir_img2img_samples = "outputs/img2img-images"

# Image format
shared.opts.samples_format = "png"
shared.opts.jpeg_quality = 80

# Model settings
shared.opts.sd_model_checkpoint = "model.safetensors"
shared.opts.sd_vae = "vae.safetensors"
```

## Extension Development

### Basic Extension Structure
```
extensions/my-extension/
├── scripts/my-extension.py
├── javascript/my-extension.js
├── style.css
└── preload.py
```

### Extension Script Template
```python
import gradio as gr
from modules import script_callbacks, shared

class MyExtension:
    def __init__(self):
        self.name = "My Extension"
    
    def ui(self, is_img2img):
        with gr.Group():
            setting = gr.Slider(0, 2, 1, label="My Setting")
        return [setting]
    
    def run(self, p, setting):
        p.extra_generation_params["My Setting"] = setting
        return p

def on_ui_settings():
    shared.opts.add_option(
        "my_extension_enabled",
        shared.OptionInfo(True, "Enable My Extension", section=('my_ext', "My Extension"))
    )

script_callbacks.on_ui_settings(on_ui_settings)
```

### Extension JavaScript
```javascript
onUiLoaded(function() {
    console.log("Extension loaded");
});

onOptionsChanged(function() {
    // React to setting changes
});
```

## Error Handling

### Python
```python
from modules.sd_samplers_common import InterruptedException

try:
    processed = process_images(p)
except InterruptedException:
    print("Generation interrupted")
except Exception as e:
    print(f"Error: {e}")
```

### API Error Response
```json
{
  "error": "HTTPException",
  "detail": "Error description",
  "errors": "Full error message"
}
```

## Memory Management

### Python
```python
from modules import devices

# Garbage collect
devices.torch_gc()

# Check device
print(devices.device)
```

### Processing Options
```python
p.do_not_save_samples = True  # Don't save to disk
p.do_not_save_grid = True     # Don't save grid
```

## Common Parameters

### txt2img Parameters
- `prompt` (str): Text description
- `negative_prompt` (str): What to avoid
- `steps` (int): Denoising steps (1-150)
- `cfg_scale` (float): Guidance scale (1-30)
- `width`, `height` (int): Image dimensions
- `sampler_name` (str): Sampling method
- `seed` (int): Random seed
- `batch_size` (int): Images per batch
- `n_iter` (int): Number of iterations

### img2img Additional Parameters
- `init_images` (list): Input images
- `denoising_strength` (float): Change amount (0.0-1.0)
- `mask` (Image): Inpainting mask
- `resize_mode` (int): Resize method
- `inpainting_fill` (int): Fill method

## Samplers
- Euler, Euler a
- DPM++ 2M, DPM++ 2M Karras
- DPM++ SDE, DPM++ SDE Karras
- LMS, LMS Karras
- DDIM, PLMS

## Common Resolutions
- 512x512 (SD 1.5 default)
- 768x768 (SD 2.x default)
- 1024x1024 (SDXL default)
- 512x768, 768x512 (portrait/landscape)

## File Paths
- Models: `models/Stable-diffusion/`
- VAE: `models/VAE/`
- Embeddings: `embeddings/`
- Hypernetworks: `models/hypernetworks/`
- Loras: `models/Lora/`
- Outputs: `outputs/`
- Extensions: `extensions/`

## Useful Modules
- `modules.shared` - Global state and settings
- `modules.processing` - Image generation
- `modules.images` - Image utilities
- `modules.sd_models` - Model management
- `modules.devices` - Device/memory management
- `modules.scripts` - Script system
- `modules.paths` - Path utilities