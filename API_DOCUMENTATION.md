# Stable Diffusion Web UI - API Documentation

## Overview

The Stable Diffusion Web UI is a comprehensive browser-based interface for AI image generation using Stable Diffusion models. This documentation covers all public APIs, functions, and components available for developers and users.

## Table of Contents

1. [REST API](#rest-api)
2. [Python Processing API](#python-processing-api)
3. [JavaScript Frontend API](#javascript-frontend-api)
4. [Configuration and Settings](#configuration-and-settings)
5. [Extension System](#extension-system)
6. [Utilities and Helper Functions](#utilities-and-helper-functions)

---

## REST API

The web UI provides a comprehensive REST API accessible at `/sdapi/v1/` endpoints.

### Base URL
```
http://localhost:7860/sdapi/v1/
```

### Authentication
If API authentication is enabled, use HTTP Basic Authentication:
```bash
curl -u username:password http://localhost:7860/sdapi/v1/txt2img
```

### Core Generation Endpoints

#### POST /sdapi/v1/txt2img
Generate images from text prompts.

**Request Body (JSON):**
```json
{
  "prompt": "a beautiful landscape, detailed, 4k",
  "negative_prompt": "blurry, low quality",
  "steps": 20,
  "cfg_scale": 7,
  "width": 512,
  "height": 512,
  "sampler_name": "Euler",
  "batch_size": 1,
  "n_iter": 1,
  "seed": -1,
  "restore_faces": false,
  "tiling": false,
  "save_images": false,
  "send_images": true,
  "override_settings": {},
  "script_name": null,
  "script_args": [],
  "alwayson_scripts": {}
}
```

**Response:**
```json
{
  "images": ["base64_encoded_image_data"],
  "parameters": { /* original parameters */ },
  "info": "generation_info_string"
}
```

**Example Usage:**
```bash
curl -X POST "http://localhost:7860/sdapi/v1/txt2img" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "a cat sitting on a table",
    "steps": 20,
    "width": 512,
    "height": 512
  }'
```

#### POST /sdapi/v1/img2img
Generate images from existing images with prompts.

**Request Body (JSON):**
```json
{
  "init_images": ["base64_encoded_image"],
  "prompt": "modify this image",
  "negative_prompt": "",
  "denoising_strength": 0.75,
  "steps": 20,
  "cfg_scale": 7,
  "width": 512,
  "height": 512,
  "sampler_name": "Euler",
  "mask": null,
  "resize_mode": 0,
  "inpainting_fill": 0,
  "inpaint_full_res": false,
  "seed": -1,
  "batch_size": 1,
  "n_iter": 1
}
```

**Response:** Same format as txt2img

**Example Usage:**
```python
import base64
import requests

# Convert image to base64
with open("input.jpg", "rb") as image_file:
    encoded_string = base64.b64encode(image_file.read()).decode()

response = requests.post(
    "http://localhost:7860/sdapi/v1/img2img",
    json={
        "init_images": [encoded_string],
        "prompt": "make it more colorful",
        "denoising_strength": 0.7
    }
)
```

### Image Processing Endpoints

#### POST /sdapi/v1/extra-single-image
Enhance a single image with upscaling and face restoration.

**Request Body:**
```json
{
  "image": "base64_encoded_image",
  "resize_mode": 0,
  "upscaling_resize": 2,
  "upscaler_1": "ESRGAN_4x",
  "upscaler_2": "None",
  "gfpgan_visibility": 0,
  "codeformer_visibility": 0,
  "codeformer_weight": 0
}
```

#### POST /sdapi/v1/extra-batch-images
Process multiple images at once.

**Request Body:**
```json
{
  "imageList": [
    {"data": "base64_image_1", "name": "image1.jpg"},
    {"data": "base64_image_2", "name": "image2.jpg"}
  ],
  "resize_mode": 0,
  "upscaling_resize": 2,
  "upscaler_1": "ESRGAN_4x"
}
```

### Information and Status Endpoints

#### GET /sdapi/v1/progress
Get current generation progress.

**Query Parameters:**
- `skip_current_image` (boolean): Skip returning current image data

**Response:**
```json
{
  "progress": 0.5,
  "eta_relative": 30.5,
  "state": {
    "job_count": 1,
    "job_no": 0,
    "interrupted": false,
    "skipped": false
  },
  "current_image": "base64_image_data",
  "textinfo": "Processing..."
}
```

#### POST /sdapi/v1/png-info
Extract generation parameters from PNG images.

**Request Body:**
```json
{
  "image": "base64_encoded_png"
}
```

**Response:**
```json
{
  "info": "generation_parameters_string",
  "items": {
    "parameters": "detailed_parameters",
    "additional_fields": "values"
  }
}
```

#### POST /sdapi/v1/interrogate
Analyze images to generate prompts.

**Request Body:**
```json
{
  "image": "base64_encoded_image",
  "model": "clip"
}
```

**Response:**
```json
{
  "caption": "generated prompt description"
}
```

### Model and Configuration Endpoints

#### GET /sdapi/v1/sd-models
List available Stable Diffusion models.

**Response:**
```json
[
  {
    "title": "Model Name",
    "model_name": "model_filename",
    "hash": "model_hash",
    "sha256": "full_hash",
    "filename": "path/to/model.safetensors",
    "config": "config_file_path"
  }
]
```

#### GET /sdapi/v1/samplers
List available sampling methods.

**Response:**
```json
[
  {
    "name": "Euler",
    "aliases": ["euler"],
    "options": {}
  },
  {
    "name": "DPM++ 2M",
    "aliases": ["dpm_2m"],
    "options": {}
  }
]
```

#### GET /sdapi/v1/upscalers
List available upscaling models.

**Response:**
```json
[
  {
    "name": "ESRGAN_4x",
    "model_name": "ESRGAN_4x",
    "model_path": "path/to/model",
    "scale": 4
  }
]
```

#### GET /sdapi/v1/options
Get current configuration options.

**Response:**
```json
{
  "setting_name": "setting_value",
  "samples_save": true,
  "samples_format": "png",
  "grid_save": true
}
```

#### POST /sdapi/v1/options
Update configuration options.

**Request Body:**
```json
{
  "setting_name": "new_value",
  "samples_save": false
}
```

### Control Endpoints

#### POST /sdapi/v1/interrupt
Interrupt current generation process.

#### POST /sdapi/v1/skip
Skip current image in batch.

#### POST /sdapi/v1/refresh-checkpoints
Refresh the list of available models.

### Training and Creation Endpoints

#### POST /sdapi/v1/create/embedding
Create a new text embedding.

**Request Body:**
```json
{
  "name": "my_embedding",
  "num_vectors_per_token": 1,
  "overwrite_old": false,
  "init_text": "initialization text"
}
```

#### POST /sdapi/v1/train/embedding
Train an existing embedding.

**Request Body:**
```json
{
  "embedding_name": "my_embedding",
  "learn_rate": 0.005,
  "batch_size": 1,
  "steps": 1000,
  "save_embedding_every": 100,
  "create_image_every": 100,
  "template_file": "template.txt",
  "log_directory": "textual_inversion"
}
```

---

## Python Processing API

The core image processing functionality is provided through Python classes and functions.

### StableDiffusionProcessingTxt2Img

Main class for text-to-image generation.

```python
from modules.processing import StableDiffusionProcessingTxt2Img, process_images
from modules import shared

# Create processing object
p = StableDiffusionProcessingTxt2Img(
    sd_model=shared.sd_model,
    prompt="a beautiful landscape",
    negative_prompt="blurry, low quality",
    steps=20,
    cfg_scale=7,
    width=512,
    height=512,
    sampler_name="Euler",
    batch_size=1,
    n_iter=1,
    seed=-1
)

# Process images
processed = process_images(p)
images = processed.images
```

**Key Parameters:**
- `prompt` (str): Text description of desired image
- `negative_prompt` (str): What to avoid in the image
- `steps` (int): Number of denoising steps (1-150)
- `cfg_scale` (float): Classifier-free guidance scale (1-30)
- `width`, `height` (int): Image dimensions
- `sampler_name` (str): Sampling method name
- `seed` (int): Random seed (-1 for random)
- `batch_size` (int): Number of images per batch
- `n_iter` (int): Number of iterations

**Advanced Parameters:**
- `enable_hr` (bool): Enable high-resolution fix
- `hr_scale` (float): High-res upscale factor
- `hr_upscaler` (str): Upscaler for high-res pass
- `denoising_strength` (float): Strength for high-res denoising
- `restore_faces` (bool): Apply face restoration
- `tiling` (bool): Create tileable images
- `override_settings` (dict): Override default settings

### StableDiffusionProcessingImg2Img

Class for image-to-image generation.

```python
from modules.processing import StableDiffusionProcessingImg2Img, process_images
from PIL import Image

# Load source image
init_image = Image.open("source.jpg")

# Create processing object
p = StableDiffusionProcessingImg2Img(
    sd_model=shared.sd_model,
    prompt="make it more colorful",
    init_images=[init_image],
    denoising_strength=0.7,
    steps=20,
    cfg_scale=7,
    width=512,
    height=512
)

# Process images
processed = process_images(p)
```

**Key Parameters:**
- `init_images` (list): List of PIL Image objects
- `denoising_strength` (float): How much to change the image (0.0-1.0)
- `mask` (Image, optional): Mask for inpainting
- `resize_mode` (int): How to resize input (0=stretch, 1=crop, 2=resize, 3=crop and resize)
- `inpainting_fill` (int): How to fill masked areas
- `inpaint_full_res` (bool): Inpaint at full resolution

### Processing Results

Both processing classes return a `Processed` object:

```python
class Processed:
    images: List[PIL.Image]  # Generated images
    seed: int               # Used seed
    info: str              # Generation info
    parameters: dict       # Used parameters
    comments: str         # Additional comments
```

### Utility Functions

#### Image Processing
```python
from modules import images

# Save image with metadata
images.save_image(
    image=pil_image,
    path="output",
    basename="generated",
    seed=12345,
    prompt="prompt text",
    extension="png",
    info="generation info"
)

# Read metadata from image
geninfo, items = images.read_info_from_image(pil_image)
```

#### Model Management
```python
from modules import sd_models

# List available models
models = sd_models.list_models()

# Load specific model
sd_models.load_model("model_name.safetensors")

# Get current model info
current_model = sd_models.select_checkpoint()
```

#### Style Management
```python
from modules import styles

# Apply styles to prompt
styled_prompt = styles.apply_styles_to_prompt(
    prompt="base prompt",
    styles=["style1", "style2"]
)

# Get available styles
available_styles = styles.get_prompt_styles()
```

---

## JavaScript Frontend API

The frontend provides various JavaScript functions for UI interaction.

### Tab Navigation

```javascript
// Switch between main tabs
switch_to_txt2img();
switch_to_img2img();
switch_to_extras();

// Switch to specific img2img modes
switch_to_sketch();       // Sketch mode
switch_to_inpaint();      // Inpainting mode
switch_to_inpaint_sketch(); // Inpainting sketch mode
```

### Image Gallery Functions

```javascript
// Get all visible gallery buttons
var buttons = all_gallery_buttons();

// Get currently selected gallery button
var selected = selected_gallery_button();

// Get index of selected image
var index = selected_gallery_index();

// Extract image from gallery
var image = extract_image_from_gallery(gallery_array);
```

### Form Submission

```javascript
// Submit txt2img form
submit(/* form arguments */);

// Submit img2img form
submit_img2img(/* form arguments */);

// Create submit arguments (removes large gallery data)
var args = create_submit_args(original_args);
```

### Progress Management

```javascript
// Request progress updates
requestProgress(
    task_id,
    gallery_container,
    gallery_element,
    onComplete_callback
);

// Restore progress for interrupted tasks
restoreProgressTxt2img();
restoreProgressImg2img();
```

### Settings and Configuration

```javascript
// Access current options
console.log(opts.setting_name);

// React to option changes
onOptionsChanged(function() {
    // Handle option changes
});

// Update input elements
updateInput(input_element);
```

### Utility Functions

```javascript
// Set random seed
setRandomSeed("txt2img_seed");

// Switch width/height values
switchWidthHeight("txt2img");

// Update image resize information
updateImg2imgResizeToTextAfterChangingImage();

// Get current image resolution
var [width, height, scale] = currentImg2imgSourceResolution(w, h, scaleBy);
```

### Style Management

```javascript
// Ask for style name
ask_for_style_name(_, prompt_text, negative_prompt_text);

// Confirm prompt clearing
confirm_clear_prompt(prompt, negative_prompt);
```

### Extension Integration

```javascript
// Register callbacks for UI events
onUiLoaded(function() {
    // Initialize extension UI
});

onAfterUiUpdate(function() {
    // Update extension elements
});

onOptionsChanged(function() {
    // React to setting changes
});
```

---

## Configuration and Settings

### Command Line Arguments

The web UI accepts numerous command line arguments for configuration:

```bash
# Basic usage
python webui.py --port 7860 --listen

# Common options
python webui.py \
  --port 7860 \                    # Port number
  --listen \                       # Listen on all interfaces
  --api \                          # Enable API
  --api-auth user:pass \           # API authentication
  --xformers \                     # Enable xformers optimization
  --medvram \                      # Optimize for medium VRAM
  --lowvram \                      # Optimize for low VRAM
  --precision full \               # Use full precision
  --no-half \                      # Disable half precision
  --autolaunch \                   # Auto-launch browser
  --theme dark \                   # UI theme
  --gradio-debug \                 # Enable gradio debug mode
  --share \                        # Create public gradio link
```

### Settings File

Settings are stored in `config.json` and can be accessed via:

```python
from modules import shared

# Get setting value
value = shared.opts.setting_name

# Set setting value
shared.opts.setting_name = new_value

# Save settings
shared.opts.save(shared.config_filename)
```

### Common Settings

```python
# Output directories
shared.opts.outdir_txt2img_samples = "outputs/txt2img-images"
shared.opts.outdir_img2img_samples = "outputs/img2img-images"
shared.opts.outdir_grids = "outputs/grids"

# Image saving
shared.opts.samples_save = True
shared.opts.samples_format = "png"
shared.opts.jpeg_quality = 80
shared.opts.save_images_before_face_restoration = False

# Generation defaults
shared.opts.sd_model_checkpoint = "model.safetensors"
shared.opts.sd_vae = "vae.safetensors"
shared.opts.CLIP_stop_at_last_layers = 1

# UI settings
shared.opts.gradio_theme = "default"
shared.opts.return_grid = True
shared.opts.grid_save = True
```

---

## Extension System

### Creating Extensions

Extensions are located in the `extensions/` directory. Basic extension structure:

```
extensions/
  my-extension/
    javascript/
      my-extension.js
    scripts/
      my-extension.py
    style.css
    preload.py
```

### Extension Script Example

```python
# scripts/my-extension.py
import gradio as gr
from modules import script_callbacks, shared

class MyExtensionScript:
    def __init__(self):
        self.name = "My Extension"
        
    def ui(self, is_img2img):
        with gr.Group():
            with gr.Accordion("My Extension", open=False):
                my_setting = gr.Slider(
                    minimum=0.0,
                    maximum=2.0,
                    value=1.0,
                    label="My Setting"
                )
        
        return [my_setting]
    
    def run(self, p, my_setting):
        # Process the generation
        p.extra_generation_params["My Setting"] = my_setting
        return p

# Register callbacks
def on_ui_settings():
    section = ('my_extension', "My Extension")
    shared.opts.add_option(
        "my_extension_enabled",
        shared.OptionInfo(
            default=True,
            label="Enable My Extension",
            section=section
        )
    )

script_callbacks.on_ui_settings(on_ui_settings)
```

### Extension JavaScript

```javascript
// javascript/my-extension.js
(function() {
    'use strict';
    
    // Extension initialization
    onUiLoaded(function() {
        console.log("My Extension loaded");
    });
    
    // Custom functions
    function myExtensionFunction() {
        // Extension logic
    }
    
    // Register with global scope if needed
    window.myExtensionFunction = myExtensionFunction;
})();
```

---

## Utilities and Helper Functions

### Image Utilities

```python
from modules import images

# Resize image
resized = images.resize_image(
    resize_mode=1,
    image=pil_image,
    width=512,
    height=512
)

# Apply face restoration
restored = images.run_face_restoration(
    pil_image,
    face_restorer_name="GFPGAN"
)

# Create grid of images
grid = images.image_grid(
    images_list,
    rows=2,
    cols=2
)
```

### File Utilities

```python
from modules import paths

# Get various directory paths
models_dir = paths.models_path
outputs_dir = paths.outputs_dir
extensions_dir = paths.extensions_dir

# List files
from modules.shared import listfiles
model_files = listfiles(models_dir)
```

### Device and Memory Management

```python
from modules import devices

# Get device info
device = devices.device
torch_device = devices.torch_device

# Memory management
devices.torch_gc()  # Garbage collect
devices.enable_tf32()  # Enable TF32 if available
```

### Progress Tracking

```python
from modules.shared import state
from tqdm import tqdm

# Check if interrupted
if state.interrupted:
    return

# Update progress
state.job_count = 10
state.job_no = 5
state.textinfo = "Processing..."

# Use with progress bar
with tqdm(total=100) as pbar:
    for i in range(100):
        if state.interrupted:
            break
        pbar.update(1)
```

---

## Error Handling

### Common Exceptions

```python
from modules.errors import NansException
from modules.sd_samplers_common import InterruptedException

try:
    # Processing code
    processed = process_images(p)
except InterruptedException:
    # Handle user interruption
    print("Generation interrupted by user")
except NansException:
    # Handle NaN values in generation
    print("NaN values detected in generation")
except Exception as e:
    # Handle other errors
    print(f"Error during processing: {e}")
```

### API Error Responses

API endpoints return error responses in the following format:

```json
{
  "error": "HTTPException",
  "detail": "Error description",
  "body": "",
  "errors": "Full error message"
}
```

Common HTTP status codes:
- `400`: Bad Request (invalid parameters)
- `401`: Unauthorized (invalid credentials)
- `404`: Not Found (resource not found)
- `422`: Unprocessable Entity (validation error)
- `500`: Internal Server Error

---

## Best Practices

### Performance Optimization

1. **Memory Management**
   ```python
   # Enable memory optimizations
   p.do_not_save_samples = True  # Don't save if not needed
   p.do_not_save_grid = True     # Don't save grid
   
   # Clean up after processing
   devices.torch_gc()
   ```

2. **Batch Processing**
   ```python
   # Process multiple images efficiently
   p.batch_size = 4  # Process 4 images at once
   p.n_iter = 1      # Single iteration
   ```

3. **Model Loading**
   ```python
   # Load model only when needed
   if shared.sd_model is None:
       sd_models.load_model()
   ```

### Error Handling

```python
# Always use try-catch for processing
try:
    processed = process_images(p)
except Exception as e:
    print(f"Processing failed: {e}")
    # Handle error appropriately
```

### Extension Development

1. **Use proper callbacks**
   ```python
   script_callbacks.on_ui_tabs(add_tab)
   script_callbacks.on_ui_settings(add_settings)
   ```

2. **Respect user settings**
   ```python
   if not shared.opts.my_extension_enabled:
       return
   ```

3. **Clean up resources**
   ```python
   def cleanup():
       # Release resources
       pass
   
   script_callbacks.on_app_started(cleanup)
   ```

---

This documentation provides comprehensive coverage of the Stable Diffusion Web UI's public APIs, functions, and components. For the most up-to-date information, always refer to the source code and official documentation.