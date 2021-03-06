# Speech-to-Text Benchmark

Made in Vancouver, Canada by [Picovoice](https://picovoice.ai)

This is a minimalist and extensible framework for benchmarking different speech-to-text engines. It has been developed
and tested on Ubuntu with Python3.

## Table of Contents

* [Background](#background)
* [Data](#data)
* [Metrics](#metrics)
    * [Word Error Rate](#word-error-rate)
    * [Real Time Factor](#real-time-factor)
    * [Memory](#memory)
* [Speech-to-Text Engines](#speech-to-text-engines)
    * [Mozilla DeepSpeech](#mozilla-deepspeech)
    * [Picovoice Cheetah](#picovoice-cheetah)
    * [PocketSphinx](#pocketsphinx)
* [Usage](#usage)
    * [Word Error Rate Measurement](#word-error-rate-measurement)
    * [Real Time Factor Measurement](#real-time-factor-measurement)
    * [Memory Usage Measurement](#memory-usage-measurement)
* [Results](#results)
* [License](#license)

## Background

This framework has been developed by [Picovoice](http://picovoice.ai/) as part of the project
[Cheetah](https://github.com/Picovoice/cheetah). Cheetah is Picovoice's speech-to-text engine specifically designed for
IoT applications. Deep learning has been the main driver in recent improvements in speech recognition. But due to
stringent compute/storage limitations of IoT platforms it is most beneficial to the cloud-based engines. Picovoice's
proprietary deep learning technology enables transferring these improvements to IoT platforms with much lower CPU/memory
footprint. The goal is to be able to run Cheetah on any platform with a C Compiler and a few MB of memory.

This framework enabled us to measure our progress in improving Cheetah and also compare its performance with existing
solutions.

## Data

[Mozilla Common Voice](https://voice.mozilla.org/en) and [LibriSpeech](http://www.openslr.org/12/) datasets are used for
benchmarking. Only the cv-valid-test portion of Common Voice dataset is used to allow engines to use the train portion
of the dataset. Since the Common Voice dataset is community-verified we only use examples that have no downvotes and at
least two upvotes. Similarly, We use the [test-clean](http://www.openslr.org/resources/12/test-clean.tar.gz) portion
of LibriSpeech dataset to allow engines use the train portions.

## Metrics

Three different metrics are measured.

### Word Error Rate

Word error rate is defined as the [Levenstein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
between words in reference transcript and words in the output of the speech-to-text engine to the number of
words in reference transcript.

### Real Time Factor

Real-time factor (RTF) is measured as the ratio of CPU (processing) time in seconds to the length
of the input speech file in seconds. A speech-to-text engine with lower RTF is computationally more efficient

### Memory

The amount of heap memory used.

## Speech-to-Text Engines

All engines below run fully on-device (no cloud connection needed).

### Mozilla DeepSpeech

[Mozilla DeepSpeech](https://github.com/mozilla/DeepSpeech) is an open-source implementation of
[Baidu's DeepSpeech](https://arxiv.org/abs/1412.5567) by Mozilla.

### Picovoice Cheetah

[Cheetah](https://github.com/Picovoice/cheetah) is a speech-to-text engine developed using
[Picovoice's](http://picovoice.ai/) proprietary deep learning technology. It works offline and is supported on a
growing number of mobile/embedded platforms including Android, iOS, and Raspberry Pi.

### PocketSphinx

[PocketSphinx](https://github.com/cmusphinx/pocketsphinx) works offline and can run on embedded platforms such as
Raspberry Pi.

## Usage

Below is information on how to use this framework to benchmark engines mentioned above. First, make sure that you have
already installed DeepSpeech and PocketSphinx on your machine following instructions on their official pages. Then
download Common Voice dataset and [test-clean](http://www.openslr.org/resources/12/test-clean.tar.gz) portion of
LibriSpeech.

### Word Error Rate Measurement

WER can be measured by running the following command from the root of the repository.
`DATASET_TYPE` is either `commonvoice` or `librispeech`. `DATASET_PATH` is the absolute path to the root directory of
dataset. `DEEP_SPEECH_MODELS_PATH` is the absolute path to Mozilla DeepSpeech's model folder.

```bash
python benchmark.py --dataset_type DATASET_TYPE --dataset_root DATASET_PATH --deep_speech_model_path DEEP_SPEECH_MODELS_PATH/output_graph.pbmm \
--deep_speech_alphabet_path DEEP_SPEECH_MODELS_PATH/alphabet.txt --deep_speech_language_model_path DEEP_SPEECH_MODELS_PATH/lm.binary \
--deep_speech_trie_path DEEP_SPEECH_MODELS_PATH/trie
```

The above prints the WER for different engines in console.

### Real Time Factor Measurement

`time` command is used to measure execution time of different engines for a given audio file and then divide
the CPU time by audio length. In order to measure execution time for Cheetah run

```bash
time resources/cheetah/cheetah_demo resources/cheetah/libpv_cheetah.so resources/cheetah/acoustic_model.pv \
resources/cheetah/language_model.pv resources/cheetah/cheetah_eval_linux_public.lic PATH_TO_WAV_FILE
```

The output should have the following format (values will be different)

```bash
real	0m4.961s
user	0m4.936s
sys	0m0.024s
```

then divide `user` by length of the audio file in seconds. The user is the actual CPU time spent in the program.

For DeepSpeech

```bash
time deepspeech --model DEEP_SPEECH_MODELS_PATH/output_graph.pbmm --alphabet PATH_TO_WAV_FILE DEEP_SPEECH_MODELS_PATH/alphabet.txt \
--lm DEEP_SPEECH_MODELS_PATH/lm.binary --trie DEEP_SPEECH_MODELS_PATH/trie --audio PATH_TO_WAV_FILE
```

Finally, for PocketSphinx

```bash
time pocketsphinx_continuous -infile PATH_TO_WAV_FILE
```

### Memory Usage measurement

[Valgrind's](http://valgrind.org/) [massif](http://valgrind.org/docs/manual/ms-manual.html) tool is used to measure
heap memory usage. For example

```bash
valgrind --tool=massif pocketsphinx_continuous -infile PATH_TO_WAV_FILE
```

It creates a file with naming like `massif.out.XXXX`. The file can be read using

```bash
ms_print massif.out.XXXX
```

## Results

Below results are obtained by following the steps above. The benchmarking is performed on a laptop running
Ubuntu 18.04 with 8 GB of RAM and Intel i7-4510U CPU running at 2GHz. WER refers to word error rate and RTF refers to
real time factor.

| Engine | WER (LibriSpeech)| WER (Common Voice) | RTF (Laptop) | RTF (Raspberry Pi 3) | RTF (Raspberry Pi Zero) | Memory |
:---:|:---:|:---:|:---:|:---:|:---:|:---:
Mozilla DeepSpeech (0.3.0) | 0.15 | 0.2 | 0.97  | -- | -- | 1521 MB |
Picovoice Cheetah (v1.0.0) | **0.10** | **0.19** | **0.07** | **0.41** | 2.33 | **71.05 MB** |
PocketSphinx (0.1.15) | 0.33 | 0.55 | 0.32 | 1.87 | **2.04** | 97.8 MB |

Cheetah achieves higher accuracy compared to any other engine on both datasets. Compared to second best performing engine,
Mozilla DeepSpeech, it is 13.9 times faster and consumes 21.4 times less memory. This enables Cheetah to run on small
commodity embedded platforms such as Raspberry Pi while delivering the benefits of large models that need much more
compute/memory resources.

## License

The benchmarking framework is freely-available and can be used under the Apache 2.0 license. Regarding Mozilla DeepSpeech
and PocketSphinx please refer to their respective pages.

The provided Cheetah resources (binary, model, and license file) are the property of Picovoice. They are
only to be used for evaluation purposes and their use in any commercial product is strictly prohibited.

For commercial inquiries regarding Cheetah please contact us at sales@picovoice.ai. For
partnership opportunities contact us at partnerships@picovoice.ai.
