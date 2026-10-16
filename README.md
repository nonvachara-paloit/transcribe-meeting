# transcribe-meeting

Local, offline meeting transcription. Produces two files per recording:

- `<name> transcript.txt` – one line per segment with `[mm:ss–mm:ss]` timestamps
- `<name> transcript - readable.txt` – segments merged into paragraphs

## Engines

| Platform | Engine used by default | Hardware |
|---|---|---|
| Apple silicon Mac (M1+) | [mlx-whisper](https://github.com/ml-explore/mlx-examples/tree/main/whisper) | Metal GPU |
| Everything else (Intel Mac, Linux, Windows) | [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | CPU (`int8`) |

Override with `--engine mlx`, `--engine faster-whisper` or `--engine crisperwhisper`. `--engine mlx` fails on non-Apple-silicon machines.

`crisperwhisper` uses [CrisperWhisper 2.0](https://huggingface.co/nyralabs/CrisperWhisper2.0_large), a verbatim model that keeps fillers ("um", "uh"), repetitions and false starts. It is never chosen by `auto`. The standard weights are licensed for non-commercial research only; Pro models are not supported.

## Installation

Requires Python 3.9+ and [ffmpeg](https://ffmpeg.org/) (used by mlx-whisper to decode audio).

```bash
# macOS
brew install ffmpeg

python3 -m venv venv
source venv/bin/activate

# Apple silicon (default engine)
pip install mlx-whisper

# Other platforms, or to use --engine faster-whisper
pip install faster-whisper

# To use --engine crisperwhisper
pip install "crisperwhisper[transformers]" "transformers<5"
```

Install only the package for the engine you need. On Apple silicon you may install both to switch between them.

The script uses whichever Python is active. If you run it outside any virtualenv and a `venv/` folder sits next to the script, it automatically re-launches itself with `venv/bin/python`.

## Usage

```bash
./transcribe-meeting recording.m4a
./transcribe-meeting recording.mp3 --model small --output-dir ./transcripts
./transcribe-meeting recording.mp3 --language en
./transcribe-meeting recording.mp3 --engine faster-whisper
./transcribe-meeting recording.mp3 --engine crisperwhisper --model turbo
```

To run the script from any directory with its virtual environment, add this function to `~/.zshrc`. Replace `/path/to/transcribe-meeting` with the project directory:

```zsh
transcribe-meeting() {
	"/path/to/transcribe-meeting/venv/bin/python" \
		"/path/to/transcribe-meeting/transcribe-meeting" \
		"$@"
}
```

| Option | Description |
|---|---|
| `--model` | Model name such as `small`, `medium` (default), `large-v3`, `turbo`. With mlx, a full Hugging Face repo id (e.g. `mlx-community/whisper-large-v3-turbo`) is also accepted. With `crisperwhisper`: `large` (default), `turbo`, `medium` or `small`. |
| `--output-dir` | Where transcripts are written (default: current directory) |
| `--language` | Language code such as `en`; auto-detected if omitted |
| `--engine` | `auto` (default), `mlx`, `faster-whisper`, or `crisperwhisper` |

Progress is shown on stderr while transcribing.

## Models

Models download on first use to the standard Hugging Face cache (`~/.cache/huggingface`, or `$HF_HOME` if set) and are reused afterwards.

- mlx engine: `mlx-community/whisper-<model>-mlx` (`turbo` maps to `mlx-community/whisper-large-v3-turbo`)
- faster-whisper engine: `Systran/faster-whisper-<model>`
- crisperwhisper engine: `nyralabs/CrisperWhisper2.0_<model>`

## Differences between engines

- faster-whisper applies a voice-activity filter that skips silence; mlx-whisper does not, so long silences may occasionally produce spurious text.
- The mlx output header does not include a language confidence value.
- crisperwhisper segments are built from word timestamps (split on pauses, sentence ends or 20 words) and the header shows the requested language rather than a detected one.
