# Rust: `no_std` digital low pass filter library
This is a `no_std` Rust library for simple digital low pass filters. You can use it for example to 
get the low frequencies from a song.

**I'm not an expert on digital signal processing. Code contributions are highly welcome! :)**

## How to use
```rust
use std::path::PathBuf;
use std::fs::File;
use audio_visualizer::waveform::staticc::png_file::visualize;
use audio_visualizer::{Channels, ChannelInterleavement};
use std::time::Instant;
use lowpass_filter::simple::apply_lpf_i16;
use minimp3::{Decoder as Mp3Decoder, Frame as Mp3Frame, Error as Mp3Error};

fn main() {
    let mut path = PathBuf::new();
    path.push("src/test/samples");
    path.push("sample_1.mp3");
    let mut decoder = Mp3Decoder::new(File::open(path).unwrap());

    let mut lrlr_mp3_samples = vec![];
    loop {
        match decoder.next_frame() {
            Ok(Mp3Frame { data: samples_of_frame, .. }) => {
                for sample in samples_of_frame {
                    lrlr_mp3_samples.push(sample);
                }
            }
            Err(Mp3Error::Eof) => break,
            Err(e) => panic!("{:?}", e),
        }
    }
    // split into left and right channel
    let (mut left, mut right) = Channels::Stereo(ChannelInterleavement::LRLR)
        .stereo_interleavement()
        .to_channel_data(&lrlr_mp3_samples);


    // left
    apply_lpf_i16(&mut left, 44100, 120);
    // right
    apply_lpf_i16(&mut right, 44100, 120);

    // visualize audio as waveform in a PNG file
    visualize(
        &left,
        Channels::Mono,
        "src/test/out",
        "sample_1_waveform_lowpassed_left.png"
    );
    visualize(
        &right,
        Channels::Mono,
        "src/test/out",
        "sample_1_waveform_lowpassed_right.png"
    );
}
```

## Visual Examples
### #1: Original Waveform
![Example 1: Original Waveform of a short sample](sample1_waveform.png "Example 1: Original Waveform of a short sample")
### #1: Lowpassed Waveform
![Example 1: Lowpassed Waveform of a short sample](sample1_waveform_lowpassed.png "Example 1: Lowpassed Original Waveform of a short sample")
### #2: Original Waveform
![Example 1: Original Waveform of a song](song_waveform.png "Example 1: Original Waveform of a song")
### #2: Lowpassed Waveform
![Example 1: Lowpassed Waveform of a song](song_waveform_lowpassed.png "Example 1: Lowpassed Original Waveform of a song")




