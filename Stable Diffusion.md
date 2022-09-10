Saved some notes [from here](https://replicate.com/blog/run-stable-diffusion-on-m1-mac) about how to run them on my Mac.

I needed Python 3.10; using the 3.11 (beta) version didn't work. Got a bunch of OpenBLAS-related compilation errors. I just used `pyenv` to create a virtual environment.

```bash
brew update
brew install Cmake protobuf rust

pyenv virtualenv stable-diffusion
pyenv activate stable-diffusion

git clone -b apple-silicon-mps-support https://github.com/bfirsh/stable-diffusion.git
cd stable-diffusion
pip install -r requirements.txt

# Download sd-v1-4.ckpt from here
#
#    https://huggingface.co/CompVis/stable-diffusion-v-1-4-original
#
# and place it in this folder as
#
#    models/ldm/stable-diffusion-v1/model.ckpt
#
mkdir -p models/ldm/stable-diffusion-v1/

# Run!
python scripts/txt2img.py \
    --prompt "a red juicy apple floating in outer space, like a planet" \
    --n_samples 1 --n_iter 1 --plms

# Results will be here
ls -l outputs/txt2img-samples/
```

### Explanation

Here's [a nice basic overview of diffusion](https://www.youtube.com/watch?v=RGBNdD3Wn-g). [This video](https://www.youtube.com/watch?v=J87hffSMB60) goes a bit more into depth.

### UIs

I found [two](https://github.com/sd-webui/stable-diffusion-webui) web [UIs](https://github.com/sd-webui/stable-diffusion-webui) that might work well.
