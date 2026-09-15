# darkmoon model packs

Neural network weights for the [darkmoon](https://darkmoon.pt) photo
editor, published as release assets because they are too large to travel
inside the application download.

This repository holds **no code**. It exists so the packs have somewhere
public and permanent to live, with the licences and the attribution of
the models inside them beside the files themselves — which is what those
licences require, and what a link in an application's credits screen
cannot do on its own.

## What a pack is

A `.darkmoon` pack is one uncompressed, block-aligned container holding
several ONNX models, with a JSON table of contents at the front. The
application memory-maps it and hands each model to onnxruntime as a
pointer, so no part of a gigabyte passes through its heap and only the
pages a given model needs are ever read from the disk. The format is
documented in `flutter_app/lib/native/model_pack.dart` in the
application's own repository.

Uncompressed on purpose: these weights give back about 8 % to zlib or
lzma alike, and a compressed model would have to be inflated into memory
before it could be used at all.

## The packs

### `models-generative.darkmoon` — release [`generative-v1`](../../releases/tag/generative-v1)

Stable Diffusion v1.5 inpainting, exported to ONNX float16. Four graphs
that run in sequence for one generative fill, which is why they are one
download.

| file | size |
|---|---|
| `sd15_inpaint_unet_fp16.onnx` | 1640.5 MB |
| `sd15_inpaint_text_encoder_fp16.onnx` | 235.0 MB |
| `sd15_inpaint_vae_decoder_fp16.onnx` | 94.5 MB |
| `sd15_inpaint_vae_encoder_fp16.onnx` | 65.3 MB |

Pack: 2,134,310,912 bytes, SHA-256
`06c2f94bef4d03f8ea09955b5452ad0b5fb0c23dca79008089198cc1276587eb`.

**Licence: [CreativeML Open RAIL-M](LICENSE-CreativeML-Open-RAIL-M.md)**,
reproduced in full in this repository as that licence requires. It is not
an unrestricted licence: Attachment A lists uses the weights may not be
put to, and those restrictions bind everyone who uses them, including
through darkmoon.

Weights by Robin Rombach, Patrick Esser and contributors
([stable-diffusion-inpainting](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-inpainting));
ONNX float16 export by
[RanaLLC](https://huggingface.co/RanaLLC/stable-diffusion-v1-5-inpainting-onnx-fp16).
The conversion to ONNX and to float16 changes the numbers a weight is
stored in and nothing else — the authorship and the licence above are the
upstream model's, unchanged.

## Using a pack

darkmoon offers to download the generative pack the first time Generative
fill is used, and keeps it in the machine's own application data
directory. Nothing has to be done by hand.

To install one manually — on a machine with no connection, say — put the
`.darkmoon` file in that directory:

| | |
|---|---|
| Windows | `%LOCALAPPDATA%\darkmoon\models` |
| macOS | `~/Library/Application Support/darkmoon/models` |
| Linux | `$XDG_DATA_HOME/darkmoon/models`, or `~/.local/share/darkmoon/models` |

The dialog in the application prints the exact path for the machine it is
running on. A pack is found by being in that folder; nothing else is
needed, and any pack found there is used.

## Checking a download

```
sha256sum models-generative.darkmoon
```

against the digest above. darkmoon does this itself and refuses to
install a pack that does not match — a partial or damaged download is
discarded rather than kept.
