# JC4827W543 AVI Player with Audio

Touchscreen AVI player.  
The app scans AVI files on the SD card, shows a simple file picker UI, and plays the selected video with audio on the 480x272 display.

<a href="https://www.buymeacoffee.com/thelastoutpostworkshop" target="_blank">
<img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png" alt="Buy Me A Coffee">
</a>

## Youtube Tutorial
[<img src="https://github.com/thelastoutpostworkshop/images/blob/main/avi_player.png" width="500">](https://youtu.be/mnOzfRFQJIM)

## What this application does

- Initializes display, touchscreen, SD card, and I2S audio output.
- Reads AVI files from `SD:/avi`.
- Shows a touch UI with left arrow (previous file), right arrow (next file), and play button.
- Plays the selected AVI, then returns to the selection screen.

## User-facing features

- Touch-based file browsing (wrap-around navigation).
- On-screen filename title.
- One-tap playback.
- Video + audio playback from SD card.
- Automatic return to menu after playback ends.

## What you can adjust in code

### Main app settings (`JC4827W543_avi_player.ino`)

- `AVI_FOLDER`: folder scanned on SD (default `"/avi"`).
- `MAX_FILES`: max number of AVI files loaded into the menu.
- `TOUCH_SDA`, `TOUCH_SCL`, `TOUCH_INT`, `TOUCH_RST`: touch controller pin mapping.
- `TOUCH_WIDTH`, `TOUCH_HEIGHT`: touch coordinate bounds.
- `touchController.setRotation(...)`: touch rotation/orientation.
- `SD.begin(..., 10000000, ...)`: SD SPI clock speed (10 MHz default).
- UI sizing values in `loop()` / `displaySelectedFile()`: arrow size/margins and play button size/position.
- Touch debounce delays (`delay(50)`, `delay(300)`).

### Video/audio playback tuning (`AviFunc.h`, `esp32_audio.h`)

- `SKIP_FRAME_TOLERANT_MS` (`AviFunc.h`): frame skip tolerance for smoother playback under load.
- Codec flags in `AviFunc.h`: `AVI_SUPPORT_CINEPAK` (enabled) and `AVI_SUPPORT_MJPEG` (disabled by default).
- `I2S_DEFAULT_GAIN_LEVEL` (`esp32_audio.h`): output volume/gain.
- `I2S_DEFAULT_SAMPLE_RATE` (`esp32_audio.h`): initial I2S sample rate.
- I2S DMA buffering in `esp32_audio.h`: `dma_buf_count` and `dma_buf_len`.

## Dependencies

Install these in Arduino IDE:

- Board: `ESP32S3 Dev Module` (tested with Espressif core `v3.2.0`)
- Library: `GFX Library for Arduino` (tested `v1.5.6`)
- Library: `Dev Device Pins` (tested `v0.0.2`)
- Library: `TAMC_GT911` (tested `v1.0.2`)
- ZIP library: `avilib` - https://github.com/lanyou1900/avilib.git
- ZIP library: `arduino-libhelix` - https://github.com/pschatzmann/arduino-libhelix.git

## SD card layout

Put AVI files in:

```text
/avi
  Apollo 11 Launch.avi
  Forest.avi
  ...
```

## Video conversion (FFmpeg)

This project is configured for **Cinepak video** and currently uses **MP3 audio decode** (`mp3_player_task_start()` in the sketch).

Recommended command:

```bash
ffmpeg -i input.mp4 -c:v cinepak -q:v 10 -vf "fps=24,scale=480:272" -c:a mp3 -ac 1 -ar 22050 output.avi
```

Notes:

- Keep width/height multiples of 4 for Cinepak.
- Lower `-q:v` gives better quality but larger file size.
- Lower FPS or bitrate can improve playback smoothness on constrained media/cards.

## Build and run

1. Open `JC4827W543_avi_player.ino` in Arduino IDE.
2. Select board `ESP32S3 Dev Module`.
3. Install libraries listed above.
4. Copy AVI files to SD card under `/avi`.
5. Upload and reset.
6. Use touch arrows to pick a file and tap play.
