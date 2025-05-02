# English Accent Classification Tool

## Purpose
This tool analyzes spoken English audio to predict the speaker's regional accent (e.g., American, British, Indian). It processes audio from URLs (like YouTube, Vimeo) or uploaded files and uses a pre-trained SpeechBrain model for classification. It's intended for internal use, potentially for evaluating spoken English during hiring processes.

## Collab Link:   https://colab.research.google.com/drive/1e3gf3OIhh1ioY3CRa3cvfKyGCRonx12k?authuser=0#scrollTo=9gRmCj56aTSO

## Prerequisites
* **Environment:** Designed to run in Google Colab.
* **Model Download:** Requires approximately 1.3 GB download for the classification model (`Jzuluaga/accent-id-commonaccent_xlsr-en-english` from Hugging Face Hub) upon first run (Cell 3). The model is cached in `/content/models_cache`.
* **Permissions:** Ensure you have the necessary permissions if using private URLs.
* **Dependencies:** Requires Python libraries like `speechbrain`, `torchaudio`, `yt-dlp`, `ffmpeg-python`, `transformers`, `omegaconf`, `soundfile`. It also requires the `ffmpeg` system package. These are installed in Cell 1.

## How to Run
Follow these steps sequentially within a Colab session:

1.  **Cell 1 (Setup):** Run this cell once per session. It installs/updates necessary Python libraries and the `ffmpeg` system package.
2.  **Cell 2 (Code Definitions):** Run this cell once per session after Cell 1. It defines configuration constants, the `AccentClassifier` class, helper functions for downloading/processing audio, and optional VAD functions.
3.  **Cell 3 (Initialize Classifier):** Run this cell once per session after Cells 1 & 2. It instantiates the `AccentClassifier` and loads the pre-trained model from the cache or downloads it (~1.3 GB) if run for the first time. **Wait for the "✅ Model is loaded and ready..." message.**
4.  **Cell 4 (Core Processing Functions):** Run this cell once per session after Cell 3. It defines functions for handling the processing workflow and displaying results with status messages.
5.  **Choose Input Method:**
    * **Cell 5 (Classify from URL):** Enter a video/audio URL (e.g., YouTube, Vimeo). Select optional preprocessing (Noise Reduction, Silero VAD). Run the cell to start the process.
    * **Cell 6 (Classify from File):** Run the cell. Use the widget to upload an audio file (e.g., `.wav`, `.mp3`, `.m4a`). Select optional preprocessing. Classification starts automatically after the upload completes.
    * **Cell 7 (Batch Classify from URLs):** Run the cell. Upload a `.txt` file containing one URL per line. Select optional preprocessing. The tool processes each URL sequentially and saves results to a CSV file, offering it for download.

## Input
* **URL (Cell 5/7):** A direct URL to a video or audio file (e.g., YouTube, Vimeo) containing English speech.
* **File Upload (Cell 6):** Common audio formats (e.g., `.wav`, `.mp3`, `.m4a`) containing English speech.
* **Batch File (Cell 7):** A `.txt` file with one URL per line.

## Preprocessing Options (Optional)
These can be enabled in Cells 5, 6, and 7 before running classification:

* **Basic Noise Reduction (`use_noise_reduction` / `upload_use_noise_reduction`):**
    * **What it does:** Uses `torchaudio.functional.vad` to attempt removing silence or very low-level noise based on a decibel threshold (`TORCHAUDIO_VAD_TRIGGER_LEVEL`). It essentially keeps segments above the threshold.
    * **When to use:** Useful for audio with distinct periods of silence (especially leading/trailing) or low background noise that you want to remove. It might be less effective if speech and noise levels are similar or overlap significantly. It might be skipped if your `torchaudio` version is too old or if it results in empty audio.

* **Silero VAD (`use_vad` / `upload_use_vad`):**
    * **What it does:** Uses the dedicated Silero VAD model (`snakers4/silero-vad`) to identify and extract only the segments predicted to contain actual speech, discarding silence and non-speech noise based on a confidence threshold (`SILERO_VAD_THRESHOLD`).
    * **When to use:** Generally more robust for isolating speech in noisy environments or audio containing significant non-speech sounds (music, background chatter, effects). Use this if you want to focus the classification *only* on the spoken parts, potentially improving accuracy if the non-speech parts are long or loud. It might discard speech if the threshold is too high or the speech quality is very poor. It will be skipped if the VAD model detects no speech at all.

**Recommendation:** Start without VAD. If results seem poor due to silence or background noise, try "Basic Noise Reduction" first. If noise is still problematic, try "Silero VAD". Using both might be redundant or overly aggressive, potentially removing useful speech segments.

## Output
The tool outputs the following in the Colab cell:
* **Predicted Accent:** The most likely English accent detected (e.g., "American English").
* **Confidence:** The model's confidence score (0-100%) for the prediction.
* **Explanation:** A brief text describing the result, confidence level, and potentially mentioning the next most likely accent or typical features.
* **Top Predictions:** A ranked list of the top 5 most likely accents and their confidence scores.
* For batch processing (Cell 7), results are also saved to a timestamped CSV file.

## Troubleshooting
* **Model Download Errors (Cell 3):** Check internet connection. Ensure Hugging Face Hub is accessible. Re-run Cell 3. Ensure `MODEL_CACHE_DIR` is writable.
* **`ffmpeg` / `yt-dlp` Errors:** Usually related to installation (Cell 1) or invalid URLs/formats. Restart runtime and re-run Cell 1 if installation errors persist. Check URL validity.
* **Long Runtimes:** Model download (first time), processing long audio/video, or batch processing takes time.
* **Low Confidence / Incorrect Prediction:** Can be due to poor audio quality, unclear speech, heavy background noise, short speech duration, or accents underrepresented in the model's training data. Try applying VAD options.
* **FFmpeg/ffprobe Errors during Processing:** Might indicate corrupted audio files or unsupported codecs not handled by `yt-dlp` or `ffmpeg`.
* **VAD Errors:** Silero VAD requires specific Pytorch versions and can sometimes fail; basic VAD depends on `torchaudio` version. If one fails, try running without it or using the other.

## Model Information
* **Source:** `Jzuluaga/accent-id-commonaccent_xlsr-en-english` on Hugging Face Hub.
* **Cache Directory:** `/content/models_cache`.
* **Interface:** Uses a custom SpeechBrain interface (`custom_interface.py`, `CustomEncoderWav2vec2Classifier`).
