# ffmpeg-normalize

[![PyPI version](https://img.shields.io/pypi/v/ffmpeg-normalize.svg)](https://pypi.org/project/ffmpeg-normalize)
![Docker Image Version](https://img.shields.io/docker/v/slhck/ffmpeg-normalize?sort=semver&label=Docker%20image)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/slhck/ffmpeg-normalize/python-package.yml)

<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-20-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END -->

A utility for batch-normalizing audio using ffmpeg.

This program normalizes media files to a certain loudness level using the EBU R128 loudness normalization procedure. It can also perform RMS-based normalization (where the mean is lifted or attenuated), or peak normalization to a certain target level.

Batch processing of several input files is possible, including video files.

**A very quick how-to:**

1. Install a recent version of [ffmpeg](https://ffmpeg.org/download.html)
2. Run `pip3 install ffmpeg-normalize`
3. Run `ffmpeg-normalize /path/to/your/file.mp4`
4. Done! 🎧 (the file will be in a folder called `normalized`)

Read on for more info.

**Contents:**

- [Requirements](#requirements)
  - [ffmpeg](#ffmpeg)
- [Installation](#installation)
  - [Shell Completions](#shell-completions)
- [Usage with Docker](#usage-with-docker)
- [High LeveL Introduction](#high-level-introduction)
- [Basic Usage](#basic-usage)
- [Examples](#examples)
- [Detailed Options](#detailed-options)
  - [File Input/Output](#file-inputoutput)
  - [General](#general)
  - [Normalization](#normalization)
  - [EBU R128 Normalization](#ebu-r128-normalization)
  - [Audio Encoding](#audio-encoding)
  - [Other Encoding Options](#other-encoding-options)
  - [Input/Output Format](#inputoutput-format)
  - [Environment Variables](#environment-variables)
- [API](#api)
- [FAQ](#faq)
  - [My output file is too large?](#my-output-file-is-too-large)
  - [What options should I choose for the EBU R128 filter? What is linear and dynamic mode?](#what-options-should-i-choose-for-the-ebu-r128-filter-what-is-linear-and-dynamic-mode)
  - [The program doesn't work because the "loudnorm" filter can't be found](#the-program-doesnt-work-because-the-loudnorm-filter-cant-be-found)
  - [Should I use this to normalize my music collection?](#should-i-use-this-to-normalize-my-music-collection)
  - [Why are my output files MKV?](#why-are-my-output-files-mkv)
  - [I get a "Could not write header for output file" error](#i-get-a-could-not-write-header-for-output-file-error)
  - [The conversion does not work and I get a cryptic ffmpeg error!](#the-conversion-does-not-work-and-i-get-a-cryptic-ffmpeg-error)
  - [What are the different normalization algorithms?](#what-are-the-different-normalization-algorithms)
  - [Couldn't I just run `loudnorm` with ffmpeg?](#couldnt-i-just-run-loudnorm-with-ffmpeg)
  - [What about speech?](#what-about-speech)
  - [After updating, this program does not work as expected anymore!](#after-updating-this-program-does-not-work-as-expected-anymore)
  - [Can I buy you a beer / coffee / random drink?](#can-i-buy-you-a-beer--coffee--random-drink)
- [Related Tools and Articles](#related-tools-and-articles)
- [Contributors](#contributors)
- [License](#license)

-------------

## Requirements

You need Python 3.9 or higher, and ffmpeg.

### ffmpeg

- ffmpeg 5.x is required, ffmpeg 6.x is recommended (it fixes [a bug for short files](https://github.com/slhck/ffmpeg-normalize/issues/87))
- Download a [static build](https://ffmpeg.org/download.html) for your system
- Place the `ffmpeg` executable in your `$PATH`, or specify the path to the binary with the `FFMPEG_PATH` environment variable in `ffmpeg-normalize`

For instance, under Linux:

```bash
wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
mkdir -p ffmpeg
tar -xf ffmpeg-release-amd64-static.tar.xz -C ffmpeg --strip-components=1
sudo cp ffmpeg/ffmpeg /usr/local/bin
sudo cp ffmpeg/ffprobe /usr/local/bin
sudo chmod +x /usr/local/bin/ffmpeg /usr/local/bin/ffprobe
```

For Windows, follow [this guide](https://www.wikihow.com/Install-FFmpeg-on-Windows).

For macOS and Linux, you can also use [Homebrew](https://brew.sh):

```bash
brew install ffmpeg
```

Note that using distribution packages (e.g., `apt install ffmpeg`) is not recommended, as these are often outdated.

## Installation

For Python 3 and pip:

```bash
pip3 install ffmpeg-normalize
```

Or download this repository, then run `pip3 install .`.

To later upgrade to the latest version, run `pip3 install --upgrade ffmpeg-normalize`.

### Shell Completions

This tool provides shell completions for bash and zsh. To install them:

<!--
Note to self: Generate the shtab ones with:

  shtab --shell=bash -u ffmpeg_normalize.__main__.create_parser > completions/ffmpeg-normalize-shtab.bash
  shtab --shell=zsh -u ffmpeg_normalize.__main__.create_parser > completions/ffmpeg-normalize-shtab.zsh

but these are not properly working yet.
-->

#### Bash

If you have [`bash-completion`](https://github.com/scop/bash-completion) installed, you can just copy your new completion script to the `/usr/local/etc/bash_completion.d` directory.

```bash
curl -L https://raw.githubusercontent.com/slhck/ffmpeg-normalize/master/completions/ffmpeg-normalize-completion.bash \
  -o /usr/local/etc/bash_completion.d/ffmpeg-normalize
```

Without bash-completion, you can manually install the completion script:

```bash
# create completions directory if it doesn't exist
mkdir -p ~/.bash_completions.d

# download and install completion script
curl -L https://raw.githubusercontent.com/slhck/ffmpeg-normalize/master/completions/ffmpeg-normalize-completion.bash \
  -o ~/.bash_completions.d/ffmpeg-normalize

# source it in your ~/.bashrc
echo 'source ~/.bash_completions.d/ffmpeg-normalize' >> ~/.bashrc
```

#### Zsh

Download the completion script and place it in the default `site-functions` directory:

```bash
curl -L https://raw.githubusercontent.com/slhck/ffmpeg-normalize/master/completions/ffmpeg-normalize.zsh \
  -o /usr/local/share/zsh/site-functions/
```

You may choose any other directory that is in your `$FPATH` variable.
Make sure your `.zshrc` file contains `autoload -Uz compinit && compinit`.

## Usage with Docker

You can use the pre-built image from Docker Hub:

```bash
docker run -v "$(pwd):/tmp" -it slhck/ffmpeg-normalize
```

Alternatively, download this repository and run

```bash
docker build -t ffmpeg-normalize .
```

Then run the container with:

```bash
docker run  -v "$(pwd):/tmp" -it ffmpeg-normalize
```

This will mount your current directory to the `/tmp` directory inside the container. Everything else works the same way as if you had installed the program locally. For example, to normalize a file:

```bash
docker run  -v "$(pwd):/tmp" -it ffmpeg-normalize /tmp/yourfile.mp4 -o /tmp/yourfile-normalized.wav
```

You will then find the normalized file in your current directory.

## High LeveL Introduction

Please read this section for a high level introduction.

**What does the program do?**

The program takes one or more input files and, by default, writes them to a folder called `normalized`, using an `.mkv` container. All audio streams will be normalized so that they have the same (perceived) volume according to the EBU R128 standard. This is done by analyzing the audio streams and applying a filter to bring them to a target level. Under the hood, the program uses ffmpeg's `loudnorm` filter to do this.

**How do I specify the input?**

Just give the program one or more input files as arguments. It works with most media files, including video files.

**How do I specify the output?**

You don't have to specify an output file name (the default is `normalized/<input>.mkv`), but if you want to override it, you can specify one output file name for each input file with the `-o` option. In this case, the container format (e.g. `.wav`) will be inferred from the file name extension that you've given.

Example:

```bash
ffmpeg-normalize 1.wav 2.wav -o 1-normalized.wav 2-normalized.wav
```

Note that if you don't specify the output file name for an input file, the container format will be MKV, and the output will be written to `normalized/<input>.mkv`. The reason for choosing the MKV container is that it can handle almost any codec combination.

Using the `-ext` option, you can supply a different output extension common to all output files, e.g. `-ext m4a`. However, you need to make sure that the container supports the codecs used for the output (see below).

**What will get normalized?**

By default, all streams from the input file will be written to the output file. For example, if your input is a video with two language tracks and a subtitle track, both audio tracks will be normalized independently. The video and subtitle tracks will be copied over to the output file.

**How will the normalization be done?**

The normalization will be performed according to the EBU R128 algorithm with the [`loudnorm` filter](https://ffmpeg.org/ffmpeg-filters.html#loudnorm) from FFmpeg, which was [originally written by Kyle Swanson](https://k.ylo.ph/2016/04/04/loudnorm.html). It will bring the audio to a specified target level. This ensures that multiple files normalized with this filter will have the same perceived loudness.

**What codec is chosen?**

The default audio encoding method is uncompressed PCM (`pcm_s16le`) to avoid introducing compression artifacts. This will result in a much higher bitrate than you might want, for example if your input files are MP3s.

Some containers (like MP4) also cannot handle PCM audio. If you want to use such containers and/or keep the file size down, use `-c:a` and specify an audio codec (e.g., `-c:a aac` for ffmpeg's built-in AAC encoder).

## Basic Usage

Supply one or more input files, and optionally, output file names:

```bash
ffmpeg-normalize input [input ...][-h][-o OUTPUT [OUTPUT ...]] [options]
```

Example:

```bash
ffmpeg-normalize 1.wav 2.wav -o 1-normalized.m4a 2-normalized.m4a -c:a aac -b:a 192k
```

For more information on the options (`[options]`) available, run `ffmpeg-normalize -h`, or read on.

## Examples

[Read the examples on the wiki.](https://github.com/slhck/ffmpeg-normalize/wiki/examples)

## Detailed Options

### File Input/Output

- `input`: Input media file(s)

- `-o OUTPUT [OUTPUT ...], --output OUTPUT [OUTPUT ...]`: Output file names.

    Will be applied per input file.

    If no output file name is specified for an input file, the output files
    will be written to the default output folder with the name `<input>.<ext>`, where `<ext>` is the output extension (see `-ext` option).

    Example: `ffmpeg-normalize 1.wav 2.wav -o 1n.wav 2n.wav`

- `-of OUTPUT_FOLDER, --output-folder OUTPUT_FOLDER`: Output folder (default: `normalized`)

    This folder will be used for input files that have no explicit output name specified.

### General

- `-f, --force`: Force overwrite existing files

- `-d, --debug`: Print debugging output

- `-v, --verbose`: Print verbose output

- `-q, --quiet`: Only print errors

- `-n, --dry-run`: Do not run normalization, only print what would be done

- `-pr`, `--progress`: Show progress bar for files and streams

- `--version`: Print version and exit

### Normalization

- `-nt {ebu,rms,peak}, --normalization-type {ebu,rms,peak}`: Normalization type (default: `ebu`).

    EBU normalization performs two passes and normalizes according to EBU R128.

    RMS-based normalization brings the input file to the specified RMS level.

    Peak normalization brings the signal to the specified peak level.

- `-t TARGET_LEVEL, --target-level TARGET_LEVEL`: Normalization target level in dB/LUFS (default: -23).

    For EBU normalization, it corresponds to Integrated Loudness Target in LUFS. The range is -70.0 - -5.0.

    Otherwise, the range is -99 to 0.

- `-p, --print-stats`: Print loudness statistics for both passes formatted as JSON to stdout.

### EBU R128 Normalization

- `-lrt LOUDNESS_RANGE_TARGET, --loudness-range-target LOUDNESS_RANGE_TARGET`: EBU Loudness Range Target in LUFS (default: 7.0).

    Range is 1.0 - 50.0.

- `--keep-loudness-range-target`: Keep the input loudness range target to allow for linear normalization.

- `--keep-lra-above-loudness-range-target`: Keep input loudness range above loudness range target.

    Can be used as an alternative to `--keep-loudness-range-target` to allow for linear normalization.

- `-tp TRUE_PEAK, --true-peak TRUE_PEAK`: EBU Maximum True Peak in dBTP (default: -2.0).

    Range is -9.0 - +0.0.

- `--offset OFFSET`: EBU Offset Gain (default: 0.0).

    The gain is applied before the true-peak limiter in the first pass only. The offset for the second pass will be automatically determined based on the first pass statistics.

    Range is -99.0 - +99.0.

- `--lower-only`: Whether the audio should not increase in loudness.

    If the measured loudness from the first pass is lower than the target loudness then normalization pass will be skipped for the measured audio source.

- `--auto-lower-loudness-target`: Automatically lower EBU Integrated Loudness Target.

    Automatically lower EBU Integrated Loudness Target to prevent falling back to dynamic filtering.

    Makes sure target loudness is lower than measured loudness minus peak loudness (input_i - input_tp) by a small amount.

- `--dual-mono`: Treat mono input files as "dual-mono".

    If a mono file is intended for playback on a stereo system, its EBU R128 measurement will be perceptually incorrect. If set, this option will compensate for this effect. Multi-channel input files are not affected by this option.

- `--dynamic`: Force dynamic normalization mode.

    Instead of applying linear EBU R128 normalization, choose a dynamic normalization. This is not usually recommended.

    Dynamic mode will automatically change the sample rate to 192 kHz. Use `-ar`/`--sample-rate` to specify a different output sample rate.

### Audio Encoding

- `-c:a AUDIO_CODEC, --audio-codec AUDIO_CODEC`: Audio codec to use for output files.

    See `ffmpeg -encoders` for a list.

    Will use PCM audio with input stream bit depth by default.

- `-b:a AUDIO_BITRATE, --audio-bitrate AUDIO_BITRATE`: Audio bitrate in bits/s, or with K suffix.

    If not specified, will use codec default.

- `-ar SAMPLE_RATE, --sample-rate SAMPLE_RATE`: Audio sample rate to use for output files in Hz.

    Will use input sample rate by default, except for EBU normalization, which will change the input sample rate to 192 kHz.

- `-ac`, `--audio-channels`: Set the number of audio channels. If not specified, the input channel layout will be used. This is equivalent to `-ac` in ffmpeg.

- `-koa, --keep-original-audio`: Copy original, non-normalized audio streams to output file

- `-prf PRE_FILTER, --pre-filter PRE_FILTER`: Add an audio filter chain before applying normalization.

    Multiple filters can be specified by comma-separating them.

- `-pof POST_FILTER, --post-filter POST_FILTER`: Add an audio filter chain after applying normalization.

    Multiple filters can be specified by comma-separating them.

    For EBU, the filter will be applied during the second pass.

### Other Encoding Options

- `-vn, --video-disable`: Do not write video streams to output

- `-c:v VIDEO_CODEC, --video-codec VIDEO_CODEC`: Video codec to use for output files (default: 'copy').

    See `ffmpeg -encoders` for a list.

    Will attempt to copy video codec by default.

- `-sn, --subtitle-disable`: Do not write subtitle streams to output

- `-mn, --metadata-disable`: Do not write metadata to output

- `-cn, --chapters-disable`: Do not write chapters to output


### Input/Output Format

- `-ei EXTRA_INPUT_OPTIONS, --extra-input-options EXTRA_INPUT_OPTIONS`: Extra input options list.

    A list of extra ffmpeg command line arguments valid for the input, applied before ffmpeg's `-i`.

    You can either use a JSON-formatted list (i.e., a list of comma-separated, quoted elements within square brackets), or a simple string of space-separated arguments.

    If JSON is used, you need to wrap the whole argument in quotes to prevent shell expansion and to preserve literal quotes inside the string. If a simple string is used, you need to specify the argument with `-e=`.

    Examples: `-ei '[ "-f", "mpegts", "-r", "24" ]'` or `-ei="-f mpegts -r 24"`

- `-e EXTRA_OUTPUT_OPTIONS, --extra-output-options EXTRA_OUTPUT_OPTIONS`: Extra output options list.

    A list of extra ffmpeg command line arguments valid for the output.

    You can either use a JSON-formatted list (i.e., a list of comma-separated, quoted elements within square brackets), or a simple string of space-separated arguments.

    If JSON is used, you need to wrap the whole argument in quotes to prevent shell expansion and to preserve literal quotes inside the string. If a simple string is used, you need to specify the argument with `-e=`.

    Examples: `-e '[ "-vbr", "3", "-preset:v", "ultrafast" ]'` or `-e="-vbr 3 -preset:v ultrafast"`

- `-ofmt OUTPUT_FORMAT, --output-format OUTPUT_FORMAT`: Media format to use for output file(s).

    See `ffmpeg -formats` for a list.

    If not specified, the format will be inferred by ffmpeg from the output file name. If the output file name is not explicitly specified, the extension will govern the format (see '--extension' option).

- `-ext EXTENSION, --extension EXTENSION`: Output file extension to use for output files that were not explicitly specified. (Default: `mkv`)

### Environment Variables

The program additionally respects environment variables:

- `TMP` / `TEMP` / `TMPDIR`

    Sets the path to the temporary directory in which files are
    stored before being moved to the final output directory.
    Note: You need to use full paths.

- `FFMPEG_PATH`

    Sets the full path to an `ffmpeg` executable other than
    the system default or you can provide a file name available on $PATH

## API

This program has a simple API that can be used to integrate it into other Python programs.

For more information see the [API documentation](https://htmlpreview.github.io/?https://github.com/slhck/ffmpeg-normalize/blob/master/docs/ffmpeg_normalize.html).

## FAQ

### My output file is too large?

This is because the default output codec is PCM, which is uncompressed. If you want to reduce the file size, you can specify an audio codec with `-c:a` (e.g., `-c:a aac` for ffmpeg's built-in AAC encoder), and optionally a bitrate with `-b:a`.

For example:

```bash
ffmpeg-normalize input.wav -o output.m4a -c:a aac -b:a 192k
```

### What options should I choose for the EBU R128 filter? What is linear and dynamic mode?

EBU R128 is a method for normalizing audio loudness across different tracks or programs. It works by analyzing the audio content and adjusting it to meet specific loudness targets. The main components are:

* Integrated Loudness (I): The overall loudness of the entire audio.
* Loudness Range (LRA): The variation in loudness over time.
* True Peak (TP): The maximum level of the audio signal.

The normalization process involves measuring these values (input) and then applying gain adjustments to meet target levels (output), typically -23 LUFS for integrated loudness. You can also specify a target loudness range (LRA) and true peak level (TP).

**Linear mode** applies a constant gain adjustment across the entire audio file. This is generally preferred because:

* It preserves the original dynamic range of the audio.
* It maintains the relative loudness between different parts of the audio.
* It avoids potential artifacts or pumping effects that can occur with dynamic processing.

**Dynamic mode**, on the other hand, can change the volume dynamically throughout the file. While this can achieve more consistent loudness, it may alter the original artistic intent and potentially introduce audible artifacts (possibly due to some bugs in the ffmpeg filter).

For most cases, linear mode is recommended. Dynamic mode should only be used when linear mode is not suitable or when a specific effect is desired. In some cases, `loudnorm` will still fall back to dynamic mode, and a warning will be printed to the console. Here's when this can happen:

* When the input loudness range (LRA) is larger than the target loudness range: If the input file has a loudness range that exceeds the specified loudness range target, the loudnorm filter will automatically switch to dynamic mode. This is because linear normalization alone cannot reduce the loudness range without dynamic processing (limiting). The `--keep-loudness-range-target` option can be used to keep the input loudness range target above the specified target.

* When the required gain adjustment to meet the integrated loudness target would result in the true peak exceeding the specified true peak limit. This is because linear processing alone cannot reduce peaks without affecting the entire signal. For example, if a file needs to be amplified by 6 dB to reach the target integrated loudness, but doing so would push the true peak above the specified limit, the filter might switch to dynamic mode to handle this situation. If your content allows for it, you can increase the true peak target to give more headroom for linear processing. If you're consistently running into true peak issues, you might also consider lowering your target integrated loudness level.

At this time, the `loudnorm` filter in ffmpeg does not provide a way to force linear mode when the input loudness range exceeds the target or when the true peak would be exceeded. There are some options to mitigate this:

- The `--keep-lra-above-loudness-range-target` option can be used to keep the input loudness range above the specified target, but it will not force linear mode in all cases.
- Similarly, the `--keep-loudness-range-target` option can be used to keep the input loudness range target.
- The `--lower-only` option can be used to skip the normalization pass completely if the measured loudness is lower than the target loudness.

### The program doesn't work because the "loudnorm" filter can't be found

Make sure you run a recent ffmpeg version and that `loudnorm` is part of the output when you run `ffmpeg -filters`. Many distributions package outdated ffmpeg versions, or (even worse), Libav's `ffmpeg` disguising as a real `ffmpeg` from the FFmpeg project.

Some ffmpeg builds also do not have the `loudnorm` filter enabled.

You can always download a static build from [their website](http://ffmpeg.org/download.html) and use that.

If you have to use an outdated ffmpeg version, you can only use `rms` or `peak` as normalization types, but I can't promise that the program will work correctly.

### Should I use this to normalize my music collection?

Generally, no.

When you run `ffmpeg-normalize` and re-encode files with MP3 or AAC, you will inevitably introduce [generation loss](https://en.wikipedia.org/wiki/Generation_loss). Therefore, I do not recommend running this on your precious music collection, unless you have a backup of the originals or accept potential quality reduction. If you just want to normalize the subjective volume of the files without changing the actual content, consider using [MP3Gain](http://mp3gain.sourceforge.net/) and [aacgain](http://aacgain.altosdesign.com/).

### Why are my output files MKV?

I chose MKV as a default output container since it handles almost every possible combination of audio, video, and subtitle codecs. If you know which audio/video codec you want, and which container is supported, use the output options to specify the encoder and output file name manually.

### I get a "Could not write header for output file" error

See the [next section](#the-conversion-does-not-work-and-i-get-a-cryptic-ffmpeg-error).

### The conversion does not work and I get a cryptic ffmpeg error!

Maybe ffmpeg says something like:

> Could not write header for output file #0 (incorrect codec parameters ?): Invalid argument

Or the program says:

> … Please choose a suitable audio codec with the `-c:a` option.

One possible reason is that the input file contains some streams that cannot be mapped to the output file, or that you are using a codec that does not work for the output file. Examples:

- You are trying to normalize a movie file, writing to a `.wav` or `.mp3` file. WAV/MP3 files only support audio, not video. Disable video and subtitles with `-vn` and `-sn`, or choose a container that supports video (e.g. `.mkv`).

- You are trying to normalize a file, writing to an `.mp4` container. This program defaults to PCM audio, but MP4 does not support PCM audio. Make sure that your audio codec is set to something MP4 containers support (e.g. `-c:a aac`).

The default output container is `.mkv` as it will support most input stream types. If you want a different output container, [make sure that it supports](https://en.wikipedia.org/wiki/Comparison_of_container_file_formats) your input file's video, audio, and subtitle streams (if any).

Also, if there is some other broken metadata, you can try to disable copying over of metadata with `-mn`.

Finally, make sure you use a recent version of ffmpeg. The [static builds](https://ffmpeg.org/download.html) are usually the best option.

### What are the different normalization algorithms?

- **EBU R128** is an EBU standard that is commonly used in the broadcasting world. The normalization is performed using a psychoacoustic model that targets a subjective loudness level measured in LUFS (Loudness Unit Full Scale). R128 is subjectively more accurate than any peak or RMS-based normalization. More info on R128 can be found in the [official document](https://tech.ebu.ch/docs/r/r128.pdf) and [the `loudnorm` filter description](http://k.ylo.ph/2016/04/04/loudnorm.html) by its original author.

- **Peak Normalization** analyzes the peak signal level in dBFS and increases the volume of the input signal such that the maximum in the output is 0 dB (or any other chosen threshold). Since spikes in the signal can cause high volume peaks, peak normalization might still result in files that are subjectively quieter than other, non-peak-normalized files.

- **RMS-based Normalization** analyzes the [RMS power](https://en.wikipedia.org/wiki/Root_mean_square#Average_power) of the signal and changes the volume such that a new RMS target is reached. Otherwise it works similar to peak normalization.

### Couldn't I just run `loudnorm` with ffmpeg?

You absolutely can. However, you can get better accuracy and linear normalization with two passes of the filter. Since ffmpeg does not allow you to automatically run these two passes, you have to do it yourself and parse the output values from the first run.

If ffmpeg-normalize is too over-engineered for you, you could also use an approach such as featured [in this Ruby script](https://gist.github.com/kylophone/84ba07f6205895e65c9634a956bf6d54) that performs the two `loudnorm` passes.

If you want dynamic normalization (the loudnorm default), simply use ffmpeg with one pass, e.g.:

```bash
ffmpeg -i input.mp3 -af loudnorm -c:a aac -b:a 192k output.m4a
```

### What about speech?

You should check out the `speechnorm` filter that is part of ffmpeg. It is a designed to be used in one pass, so you don't need this script at all.

See [the documentation](https://ffmpeg.org/ffmpeg-all.html#speechnorm) for more information.

### After updating, this program does not work as expected anymore!

You are probably using a 0.x version of this program. There are significant changes to the command line arguments and inner workings of this program, so please  adapt your scripts to the new one. Those changes were necessary to address a few issues that kept piling up; leaving the program as-is would have made it hard to extend it. You can continue using the old version (find it under *Releases* on GitHub or request the specific version from PyPi), but it will not be supported anymore.

### Can I buy you a beer / coffee / random drink?

If you found this program useful and feel like giving back, feel free to send a donation [via PayPal](https://paypal.me/WernerRobitza).

## Related Tools and Articles

- [Create an AppleScript application to drop or open a folder of files in ffmpeg-normalize](https://prehensileblog.wordpress.com/2022/04/15/create-an-applescript-application-to-drop-or-open-a-folder-of-files-in-ffmpeg-normalize/)

*(Have a link? Please propose an edit to this section via a pull request!)*

## Contributors

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tbody>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://overtag.dk/"><img src="https://avatars.githubusercontent.com/u/374612?v=4?s=100" width="100px;" alt="Benjamin Balder Bach"/><br /><sub><b>Benjamin Balder Bach</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=benjaoming" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://chaos.social/@eleni"><img src="https://avatars.githubusercontent.com/u/511547?v=4?s=100" width="100px;" alt="Eleni Lixourioti"/><br /><sub><b>Eleni Lixourioti</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=Geekfish" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/thenewguy"><img src="https://avatars.githubusercontent.com/u/77731?v=4?s=100" width="100px;" alt="thenewguy"/><br /><sub><b>thenewguy</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=thenewguy" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/aviolo"><img src="https://avatars.githubusercontent.com/u/560229?v=4?s=100" width="100px;" alt="Anthony Violo"/><br /><sub><b>Anthony Violo</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=aviolo" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://jacobs.af/"><img src="https://avatars.githubusercontent.com/u/952830?v=4?s=100" width="100px;" alt="Eric Jacobs"/><br /><sub><b>Eric Jacobs</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=jetpks" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/kostalski"><img src="https://avatars.githubusercontent.com/u/34033008?v=4?s=100" width="100px;" alt="kostalski"/><br /><sub><b>kostalski</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=kostalski" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="http://justinppearson.com/"><img src="https://avatars.githubusercontent.com/u/8844823?v=4?s=100" width="100px;" alt="Justin Pearson"/><br /><sub><b>Justin Pearson</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=justinpearson" title="Code">💻</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Nottt"><img src="https://avatars.githubusercontent.com/u/13532436?v=4?s=100" width="100px;" alt="ad90xa0-aa"/><br /><sub><b>ad90xa0-aa</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=Nottt" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/Mathijsz"><img src="https://avatars.githubusercontent.com/u/1891187?v=4?s=100" width="100px;" alt="Mathijs"/><br /><sub><b>Mathijs</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=Mathijsz" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/mpuels"><img src="https://avatars.githubusercontent.com/u/2924816?v=4?s=100" width="100px;" alt="Marc Püls"/><br /><sub><b>Marc Püls</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=mpuels" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="http://www.mvbattista.com/"><img src="https://avatars.githubusercontent.com/u/158287?v=4?s=100" width="100px;" alt="Michael V. Battista"/><br /><sub><b>Michael V. Battista</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=mvbattista" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="http://auto-editor.com"><img src="https://avatars.githubusercontent.com/u/57511737?v=4?s=100" width="100px;" alt="WyattBlue"/><br /><sub><b>WyattBlue</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=WyattBlue" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/g3n35i5"><img src="https://avatars.githubusercontent.com/u/17593457?v=4?s=100" width="100px;" alt="Jan-Frederik Schmidt"/><br /><sub><b>Jan-Frederik Schmidt</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=g3n35i5" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/mjhalwa"><img src="https://avatars.githubusercontent.com/u/8994014?v=4?s=100" width="100px;" alt="mjhalwa"/><br /><sub><b>mjhalwa</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=mjhalwa" title="Code">💻</a></td>
    </tr>
    <tr>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/07416"><img src="https://avatars.githubusercontent.com/u/14923168?v=4?s=100" width="100px;" alt="07416"/><br /><sub><b>07416</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=07416" title="Documentation">📖</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/sian1468"><img src="https://avatars.githubusercontent.com/u/58017832?v=4?s=100" width="100px;" alt="sian1468"/><br /><sub><b>sian1468</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=sian1468" title="Tests">⚠️</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/psavva"><img src="https://avatars.githubusercontent.com/u/1454758?v=4?s=100" width="100px;" alt="Panayiotis Savva"/><br /><sub><b>Panayiotis Savva</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=psavva" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/HighMans"><img src="https://avatars.githubusercontent.com/u/42877729?v=4?s=100" width="100px;" alt="HighMans"/><br /><sub><b>HighMans</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=HighMans" title="Code">💻</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://github.com/kanjieater"><img src="https://avatars.githubusercontent.com/u/32607317?v=4?s=100" width="100px;" alt="kanjieater"/><br /><sub><b>kanjieater</b></sub></a><br /><a href="#ideas-kanjieater" title="Ideas, Planning, & Feedback">🤔</a></td>
      <td align="center" valign="top" width="14.28%"><a href="https://ahmetsait.com/"><img src="https://avatars.githubusercontent.com/u/8372246?v=4?s=100" width="100px;" alt="Ahmet Sait"/><br /><sub><b>Ahmet Sait</b></sub></a><br /><a href="https://github.com/slhck/ffmpeg-normalize/commits?author=ahmetsait" title="Code">💻</a></td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td align="center" size="13px" colspan="7">
        <img src="https://raw.githubusercontent.com/all-contributors/all-contributors-cli/1b8533af435da9854653492b1327a23a4dbd0a10/assets/logo-small.svg">
          <a href="https://all-contributors.js.org/docs/en/bot/usage">Add your contributions</a>
        </img>
      </td>
    </tr>
  </tfoot>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

## License

The MIT License (MIT)

Copyright (c) 2015-2022 Werner Robitza

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
