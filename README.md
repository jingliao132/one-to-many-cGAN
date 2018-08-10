# One-to-many-cGAN
Pytorch implementation of one-to-many cGAN for multiple image-to-image translations.

We provide both realizations of the Synthesize adversarial training. The training and testing scripts files are in the folder hybrid_fake and cascade_learn. Copy train.lua and test.lua of the corresponding desired realization to the main directory.

## Requirements
- Linux or OSX
- NVIDIA GPU + CUDA + CuDNN

- A self-contained Torch7 and dependencies from https://github.com/torch/distro

- Torch packages nngraph and display by:
```bash
luarocks install nngraph
luarocks install https://raw.githubusercontent.com/szym/display/master/display-scm-0.rockspec
```

## Getting started
- Clone this repo:
```bash
git clone https://github.com/jingliao132/one-to-many-cGAN.git
cd one-to-many-cGAN
```

- Download and Prepare your datasets
Original UT-Zappos50K dataset from https://github.com/phillipi/pix2pix#datasets

Original MVOD5K dataset from paper [Mobile multi-view object image search] (Çalışır F, Baştan M, Ulusoy Ö, Güdükbay U (2017) Mobile multi-view object image search. Multimedia Tools & Applications 76(10):12433–12456)

Prepare training and testing data: refer to project [pix2pix](https://github.com/phillipi/pix2pix#setup-training-and-test-data)

- Train the model
```bash
DATA_ROOT=./datasets/facades name=facades_generation which_direction=BtoA th train.lua
```
- (Optionally) start the display server to view results as the model trains (See [Display UI](#display-ui) for more details)
```bash
th -ldisplay.start 8000 0.0.0.0
```

- Finally, test the model:
```bash
DATA_ROOT=./datasets/facades name=facades_generation which_direction=BtoA phase=val th test.lua
```
The test results will be saved to an html file here: `./results/facades_generation/latest_net_G_val/index.html`.

## Commands
### Train
```bash
DATA_ROOT=/path/to/data/ name=expt_name which_direction=AtoB th train.lua
```
Switch `AtoB` to `BtoA` to train translation in opposite direction.

Models are saved to `./checkpoints/expt_name` (can be changed by passing `checkpoint_dir=your_dir` in train.lua).

See `opt` in train.lua for additional training options.

### Test
```bash
DATA_ROOT=/path/to/data/ name=expt_name which_direction=AtoB phase=val th test.lua
```

This will run the model named `expt_name` in direction `AtoB` on all images in `/path/to/data/val`.

Result images, and a webpage to view them, are saved to `./results/expt_name` (can be changed by passing `results_dir=your_dir` in test.lua).

See `opt` in test.lua for additional testing options.

### Generating image Pairs
We modified the python script in [pix2pix](https://github.com/phillipi/pix2pix) to generate training data in the form of pairs of images {A,B,B2}, where B and B2 are two different depicitions of the same underlying scene to be translated from A.

Create folder `/path/to/data`(e.g. /one-to-many-cGAN/rawdata) with subfolders `A`,`B` and `B2`. `A`,`B` and `B2` should each have their own subfolders `train`, `val`, `test`, etc. In `/one-to-many-cGAN/rawdata/A/train`, put training images in style A. In `/one-to-many-cGAN/rawdata/B/train`, put the corresponding images in style B. In `/one-to-many-cGAN/rawdata/B2/train`, put the corresponding images in style B2. Repeat same for other data splits (`val`, `test`, etc).

Corresponding images in a pair {A,B,B2} must be the same size and have the same filename, e.g. `/one-to-many-cGAN/rawdata/A/train/1.jpg` is considered to correspond to `/one-to-many-cGAN/rawdata/train/1.jpg`.

Once the data is formatted this way, call:
```bash
python scripts/combine_A_and_B.py --fold_A /path/to/data/A --fold_B /path/to/data/B --fold_B2 /path/to/data/B2 --fold_AB /path/to/data
```

This will combine each pair of images (A,B,B2) into a single image file, ready for training.

### Display UI
Optionally, for displaying images during training and test, use the [display package](https://github.com/szym/display).

- Install it with: `luarocks install https://raw.githubusercontent.com/szym/display/master/display-scm-0.rockspec`
- Then start the server with: `th -ldisplay.start`
- Open this URL in your browser: [http://localhost:8000](http://localhost:8000)

By default, the server listens on localhost. Pass `0.0.0.0` to allow external connections on any interface:
```bash
th -ldisplay.start 8000 0.0.0.0
```
Then open `http://(hostname):(port)/` in your browser to load the remote desktop.

L1 error is plotted to the display by default. Set the environment variable `display_plot` to a comma-seperated list of values `errL1`, `errG` and `errD` to visualize the L1, generator, and descriminator error respectively. For example, to plot only the generator and descriminator errors to the display instead of the default L1 error, set `display_plot="errG,errD"`.

## Citation
Please cite our paper if you use code of this repo:
```
@article{Chai2018A,
  title={A one-to-many conditional generative adversarial network framework for multiple image-to-image translations},
  author={Chai, Chunlei and Liao, Jing and Zou, Ning and Sun, Lingyun},
  journal={Multimedia Tools & Applications},
  pages={1-28},
  year={2018},
}
```
## Acknowledgments
Code and this README instructions borrow heavily from [pix2pix](https://github.com/phillipi/pix2pix). The script combine_A_and_Bs.py is modified from combine_A_and_B.py in [pix2pix](https://github.com/phillipi/pix2pix).
