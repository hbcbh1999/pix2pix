# pix2pix
[[Project]](https://phillipi.github.io/pix2pix/)   [[arxiv]](https://arxiv.org/pdf/1611.07004v1.pdf)

Torch implementation for learning a mapping from input images to output images, for example:

<img src="imgs/examples.png" width="900px"/>

"Image-to-Image Translation Using Conditional Adversarial Networks"  
 [Phillip Isola](http://web.mit.edu/phillipi/), [Jun-Yan Zhu](https://people.eecs.berkeley.edu/~junyanz/), [Tinghui Zhou](https://people.eecs.berkeley.edu/~tinghuiz/), [Alexei A. Efros](https://people.eecs.berkeley.edu/~efros/)   
 arxiv, 2016.

On some tasks, decent results can be obtained fairly quickly and on small datasets. For example, to learn to generate facades (example shown above), we trained on just 400 images for about 2 hours (on a single Pascal Titan X GPU).

## Setup

### Prerequisites
- Linux or OSX
- Python with numpy
- NVIDIA GPU + CUDA CuDNN (CPU mode and CUDA without CuDNN may work with minimal modification, but untested)

### Getting Started
- Install torch and dependencies from https://github.com/torch/distro
- Clone this repo:
```bash
git clone git@github.com:phillipi/pix2pix.git
cd pix2pix
```
- Download the dataset (e.g. [CMP Facades](http://cmp.felk.cvut.cz/~tylecr1/facade/)):
```bash
bash ./datasets/download_dataset.sh facades
```
- Train the model
```bash
DATA_ROOT=./datasets/facades name=facades_generation which_direction=BtoA th train.lua
```
- (Optionally) start the display server to view results as the model trains. (This requires torch package `display`. See `Display UI` for more details):
```bash
th -ldisplay.start 8000 0.0.0.0
```
- Finally, test the model:
```bash
DATA_ROOT=./datasets/facades name=facades_generation which_direction=BtoA phase=val th test.lua
```
The test results will be saved to an html file here: `./datasets/facades/results/facades_generation/latest_net_G_val/index.html`.

## Train
```bash
DATA_ROOT=/path/to/data/ name=expt_name which_direction=AtoB th train.lua
```
Switch `AtoB` to `BtoA` to train translation in opposite direction.

Models are saved to `./checkpoints/expt_name` (can be changed by passing `checkpoint_dir=your_dir` in train.lua).

See `opt` in train.lua for additional training options.

## Test
```bash
DATA_ROOT=/path/to/data/ name=expt_name which_direction=AtoB phase=val th test.lua
```

This will run the model named `expt_name` in direction `AtoB` on all images in `/path/to/data/val`.

Result images, and a webpage to view them, are saved to `./results/expt_name` (can be changed by passing `results_dir=your_dir` in test.lua).

See `opt` in test.lua for additional testing options.


## Datasets
Download the datasets using the following script (more datasets are coming soon!):
```bash
bash ./datasets/download_dataset.sh dataset_name
```
- `facades`: 400 images from [CMP Facades dataset](http://cmp.felk.cvut.cz/~tylecr1/facade/).


### Setup Training and Test data
We require training data in the form of pairs of images {A,B}, where A and B are two different depicitions of the same underlying scene. For example, these might be pairs {label map, photo} or {bw image, color image}. Then we can learn to translate A to B or B to A:

Create folder `/path/to/data` with subfolders `A` and `B`. `A` and `B` should each have their own subfolders `train`, `val`, `test`, etc. In `/path/to/data/A/train`, put training images in style A. In `/path/to/data/B/train`, put the corresponding images in style B. Repeat same for other data splits (`val`, `test`, etc).

Corresponding images in a pair {A,B} must be the same size and have the same filename, e.g. `/path/to/data/A/train/1.jpg` is considered to correspond to `/path/to/data/B/train/1.jpg`.

Once the data is formatted this way, call:
```bash
python data/combine_A_and_B.py --fold_A /path/to/data/A --fold_B /path/to/data/B --fold_AB /path/to/data
```

This will combine each pair of images (A,B) into a single image file, ready for training.


## Display UI
Optionally, for displaying images during training and test, use the [display package](https://github.com/szym/display).

- Install it with: `luarocks install https://raw.githubusercontent.com/szym/display/master/display-scm-0.rockspec`
- Then start the server with: `th -ldisplay.start`
- Open this URL in your browser: [http://localhost:8000](http://localhost:8000)

By default, the server listens on localhost. Pass `0.0.0.0` to allow external connections on any interface:
```bash
th -ldisplay.start 8000 0.0.0.0
```
Then open `http://(hostname):(port)/` in your browser to load the remote desktop.

## Citation
If you use this code for your research, please cite our paper <a href="https://arxiv.org/pdf/1611.07004v1.pdf">Image-to-Image Translation Using Conditional Adversarial Networks</a>:

```
@article{pix2pix2016,
  title={Image-to-Image Translation with Conditional Adversarial Networks},
  author={Isola, Phillip and Zhu, Jun-Yan and Zhou, Tinghui and Efros, Alexei A},
  journal={arxiv},
  year={2016}
}
```

## Acknowledgments
Code borrows heavily from [DCGAN](https://github.com/soumith/dcgan.torch) and [Context-Encoder](https://github.com/pathak22/context-encoder).
