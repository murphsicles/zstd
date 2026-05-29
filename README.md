# zstd

[![Zeta](https://img.shields.io/badge/Zeta-Language-%23FF6B6B?style=flat-square)](https://zeta-lang.org)
[![zorbs.io](https://img.shields.io/badge/zorbs.io-@compress/zstd-%2300C4FF?style=flat-square)](https://zorbs.io/@compress/zstd)
[![GitHub](https://img.shields.io/badge/GitHub-murphsicles/zstd-%23181717?style=flat-square)](https://github.com/murphsicles/zstd)

Fast streaming compression for Zeta. Zstd provides real-time compression and decompression using the Zstandard algorithm — offering higher compression ratios than gzip with faster decompression speeds.

## Quick Start

Add to your `zorb.toml`:

```toml
[dependencies]
@compress/zstd = "1.0"
```

Compress and decompress data:

```zeta
use @compress/zstd::{encode_all, decode_all};
use std::io::Cursor;

fn main() {
    let input: [u8; 1000] = [42; 1000]; // 1000 bytes of data

    // Compress with default compression level
    let compressed = encode_all(Cursor(input), DEFAULT_COMPRESSION_LEVEL).unwrap();
    print("Compressed: {} bytes → {} bytes\n", input.len, compressed.len);

    // Decompress back
    let decompressed = decode_all(Cursor(compressed)).unwrap();
    print("Decompressed: {} bytes\n", decompressed.len);
}
```

## Streaming API

Use `Encoder` and `Decoder` for streaming compression of large data:

```zeta
use @compress/zstd::stream::{Encoder, Decoder};
use std::io::{Read, Write, Cursor};

fn main() {
    // Streaming encode
    let mut encoder = Encoder::new(Vec<u8>(), 3).unwrap(); // Level 3
    encoder.write_all(my_data).unwrap();
    let compressed = encoder.finish().unwrap();

    // Streaming decode
    let mut decoder = Decoder::new(Cursor(compressed)).unwrap();
    let mut decompressed = Vec::new();
    decoder.read_to_end(&mut decompressed).unwrap();
}
```

## Bulk API

For one-shot compression where you already have all the data in memory:

```zeta
use @compress/zstd::bulk::{compress, decompress};

fn main() {
    let compressed = compress(my_data, DEFAULT_COMPRESSION_LEVEL).unwrap();
    let recovered = decompress(compressed, my_data.len()).unwrap();
}
```

## Compression Levels

| Level | Range | Typical Use |
|-------|-------|-------------|
| 1 | Fastest | Real-time, low CPU budget |
| 3 | Default | Good balance (general purpose) |
| 6–9 | Higher | Archival, smaller output |
| 10–22 | Ultra | Maximum compression (slow) |

```zeta
// Check available range
let (min, max) = compression_level_range();
print("Levels {} to {} are supported\n", min, max);
```

## Dictionary Compression

For small-data workloads (records, configs), pre-trained dictionaries improve compression ratios significantly:

```zeta
use @compress/zstd::dict::{EncoderDictionary, DecoderDictionary};

fn main() {
    let dict_data = train_dictionary(samples);
    let enc_dict = EncoderDictionary::new(dict_data, 3);
    let dec_dict = DecoderDictionary::new(dict_data);

    let compressed = encode_all_with_dict(data, enc_dict).unwrap();
    let recovered = decode_all_with_dict(compressed, dec_dict).unwrap();
}
```

## API Overview

| Function / Type | Description |
|----------------|-------------|
| `encode_all(input, level)` | One-shot compress everything |
| `decode_all(input)` | One-shot decompress everything |
| `Encoder` | Streaming compressor (implements `Write`) |
| `Decoder` | Streaming decompressor (implements `Read`) |
| `DEFAULT_COMPRESSION_LEVEL` | Default level (3) |
| `compression_level_range()` | Returns `(min, max)` supported levels |
| `bulk::compress(data, level)` | Bulk in-memory compression |
| `bulk::decompress(data, capacity)` | Bulk in-memory decompression |
| `dict` | Dictionary training and encoding |

## Tips

- For network streams, use `Encoder`/`Decoder` (streaming) — not `encode_all`
- If you're compressing a file on disk, use `Encoder` wrapping a `BufWriter`
- Bulk API is faster than streaming when you already have the full buffer
- Compression level 3 is the default for good reason — levels above 6 give diminishing returns on most data

## License

MIT
