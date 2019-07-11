# LakhNES: Generate 8-bit music with machine learning

## Get the data

LakhNES is first trained on [Lakh MIDI](https://colinraffel.com/projects/lmd/) and then fine tuned on [NES-MDB](https://github.com/chrisdonahue/nesmdb). The MIDI files from these datasets are first converted into a list of musical *events* (words), so that the data is easily ingested into NLP-focused pipelines.

The NES-MDB dataset has been preprocessed into two event-based formats: `TX1` and `TX2`. The [`TX1` format](#tx1-format) only has *composition* information: the notes and their timings. The [`TX2` format](#tx2-format) has *expressive* information: dynamics and timbre information.

You can get the data in `TX1` (used in our paper) and `TX2` (*not* used in our paper) formats here:

* (10 MB) [Download](https://drive.google.com/open?id=1WO6guGagqaw22LH32_NEBeavtbaWy_ar "94763d90ba98ca457b64af3c49b1ed7d9e1434e5ef534be34e836c71d4693cbe") NES-MDB in [TX1 Format](#tx1-format)
* (20 MB) [Download](https://drive.google.com/open?id=1ko3LXvotfubZ-C8Xq_K01bWd5NWiduH9 "c787a4e04bcf6439a8674683e66f2b87062383296199f3ae3c0ef66de23c50e4") NES-MDB in [TX2 Format](#tx2-format)

The remainder of the instructions assume you have moved (at least one of) these bundles to the `LakhNES/data` folder and `tar xvfz` them there.

### TX1 Format

TODO

### TX2 Format

TODO

## Generation environment

```
cd LakhNES
virtualenv -p python3 --no-site-packages LakhNES-gen
source LakhNES-gen/bin/activate
pip install torch torchvision
```

## Synthesis environment

LakhNES requires the Python package `nesmdb` to synthesize chiptune audio. Unfortunately, `nesmdb` does not support Python 3 (which the rest of this codebase depends on).

We *strongly* recommend using `virtualenv` to install `nesmdb` and run it is a local RPC server. To do this, run the following commands from this repository:

```
cd LakhNES
virtualenv -p python2.7 --no-site-packages LakhNES-synth
source LakhNES-synth/bin/activate
pip install nesmdb
pip install pretty_midi
python data/synth_server.py 1337
```

This will expose an RPC server on port `1337` with a two methods `tx1_to_wav` and `tx2_to_wav`. Both take a `TX1/TX2` input file path, a `WAV` output file path`, and optionally a `MIDI` downsample rate downsample rate. A lower rate speeds up synthesis but will mess up the rhythms (full rate if not specified).

Once you [have the data](#get-the-data) and have both your [generation](#generation-environment) and [synthesis](#synthesis-environment) ready, you can test your synthesis environment from another pane:

```
source LakhNES-gen/bin/activate
python data/synth_client.py data/nesmdb_tx1/train/191_Kirby_sAdventure_02_03PlainsLevel.tx1.txt plains_tx1.wav 48
aplay plains_tx1.wav
python data/synth_client.py data/nesmdb_tx2/train/191_Kirby_sAdventure_02_03PlainsLevel.tx2.txt plains_tx2.wav 48
aplay plains_tx2.wav
```

## Download checkpoints

Here we provide all of the Transformer-XL checkpoints used for the results in our paper. All of them use TX1. We recommend using the `LakhNES` checkpoint (which was pretrained on Lakh MIDI for 400k batches before fine tuning on NES-MDB), but the others can also produce interesting results (in particular *NESAug*).

* (147 MB) **Recommended** [Download](https://drive.google.com/open?id=1ND27trP3pTAl6eAk5QiYE9JjLOivqGsd "856e2ec6db1568d6712d73703804a518616174aaf6eb419ea763bf7490b0b61c") `LakhNES` (400k steps Lakh pre-training)
* (147 MB) [Download](https://drive.google.com/open?id=19SN-1vxbNhm_i3lMb_swMVeg5PYiQmkF "b4cec0333e30be6bea04fddfef807ca426e7367c64688619b2da085ff5d1fcfb") `Lakh200k` (200k steps Lakh pre-training)
* (147 MB) [Download](https://drive.google.com/open?id=1dmqCQ7qqjfyJF-wK8AYgPqgNRRDjyYmR "1fe9606306e9d1e8511a730ab2e67909a86d76a79e96aa90d49be90e0de75a18") `Lakh100k` (100k steps Lakh pre-training)
* (147 MB) [Download](https://drive.google.com/open?id=13lCurR-OWpqCAu_KehogkAU18-jVm3lU "d137ddc03796bd247d5b200512c0464c1ab33772e7c7511de9e9b2bc7d4a2d83") `NESAug` (No Lakh pre-training but uses data augmentation)
* (147 MB) [Download](https://drive.google.com/open?id=1qgN1PxOdSZ8T-zwLqSRvjIXmdV062J3- "b297c5afedd6f11e5d2d4a57e89887161f29c9dada97af0d367da76b06c43e65") `NES` (No Lakh pre-training or data augmentation)

If you download all of the checkpoints and `tar xvfz` them under `LakhNES/model/pretrained`, you can reproduce the exact numbers from our paper (Table 2 and Figure 3):

```
source LakhNES-gen/bin/activate
cd model
./reproduce_paper_eval.sh
```

## Generate new chiptunes

To generate new chiptunes, first [set up your generation environment](#generation-environment), [download a checkpoint](#download-checkpoints), and [start your synthesis server](#synthesis-environment). Then, run the following:

```
source LakhNES-gen/bin/activate
python generate.py \
	<MODEL_DIR> \
	--out_dir ./generated \
	--num 1
python data/synth_client.py ./generated/0.tx1.txt ./generated/0.tx1.wav
aplay ./generated/0.tx1.wav
```

## Train LakhNES

I (Chris) admit it. My patch of [the official Transformer-XL codebase](https://github.com/kimiyoung/transformer-xl) (which lives under the `model` subdirectory) is just about the ugliest code I've ever written. Instructions about how to use it are forthcoming. For now, I focused on making the [pretrained checkpoints](#download-a-pre-trained-checkpoint) easy to use. I hope that will suffice for now.

One asset of our training pipeline, the code which adapts Lakh MIDI to NES MIDI for transfer learning, is somewhat more polished. It can be found at `LakhNES/data/adapt_lakh_to_nes.py`.

## Attribution

If you use this work in your research, please cite us via the following BibTeX:

```
@inproceedings{donahue2019lakhnes,
  title={LakhNES: Improving multi-instrumental music generation with cross-domain pre-training},
  author={Donahue, Chris and Mao, Huanru Henry and Li, Yiting Ethan and Cottrell, Garrison W. and McAuley, Julian},
  booktitle={ISMIR},
  year={2019}
}
```
