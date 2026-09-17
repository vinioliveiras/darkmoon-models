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

### `models-generative.darkmoon` — release [`generative-v2`](../../releases/tag/generative-v2)

Stable Diffusion XL inpainting, exported to ONNX. Five graphs that run in
sequence for one generative fill, which is why they are one download.
Each is an `.onnx` graph beside its own `.onnx_data`, because three of
them hold more weight than the ONNX format can carry in a single file.

| file | precision | size |
|---|---|---|
| `sdxl_inpaint_unet_fp16.onnx` | float16 | 4900.4 MB |
| `sdxl_inpaint_text_encoder_2_fp16.onnx` | float16 | 1325.6 MB |
| `sdxl_inpaint_text_encoder_fp16.onnx` | float16 | 221.4 MB |
| `sdxl_inpaint_vae_decoder.onnx` | float32 | 188.9 MB |
| `sdxl_inpaint_vae_encoder.onnx` | float32 | 130.4 MB |

The VAE stays at full precision on purpose: SDXL's overflows float16 and
decodes to a black frame. The three that were converted keep float32
inputs and outputs, so the precision of a graph is a property of the file
and not of the code that runs it.

Pack: 7,095,713,792 bytes, SHA-256
`3f467b15fa5cdb64945f4b837c7c83010bc6a1bed8657ccd05a55608673ab915`.

A release asset stops at 2 GiB, so the pack is published as four parts —
`models-generative.darkmoon.part1` to `part4` — which darkmoon fetches and
concatenates. The digest above is of the assembled file. To put them
together by hand:

```
cat models-generative.darkmoon.part? > models-generative.darkmoon
```

**Licence: [CreativeML Open RAIL++-M](LICENSE-SDXL.txt)**, reproduced in
full in this repository as that licence requires. It is not an
unrestricted licence: Attachment A lists uses the weights may not be put
to, and those restrictions bind everyone who uses them, including through
darkmoon.

Weights by Stability AI and the diffusers team
([stable-diffusion-xl-1.0-inpainting-0.1](https://huggingface.co/diffusers/stable-diffusion-xl-1.0-inpainting-0.1)).
There is no official ONNX export of this model, so darkmoon makes its own
(`flutter_app/tool/export_sdxl_inpaint.py` in the application's
repository) and checks it against the reference implementation before
publishing it: the text encoders and the UNet agree to six decimal
places, the VAE round trip to four. The conversion changes the format the
weights are stored in and nothing else — the authorship and the licence
above are the upstream model's, unchanged.

### `models-generative.darkmoon` — release [`generative-v1`](../../releases/tag/generative-v1)

Stable Diffusion v1.5 inpainting, exported to ONNX float16. Superseded by
`generative-v2` and no longer read by darkmoon; kept so that older
versions of the application go on working.

| file | size |
|---|---|
| `sd15_inpaint_unet_fp16.onnx` | 1640.5 MB |
| `sd15_inpaint_text_encoder_fp16.onnx` | 235.0 MB |
| `sd15_inpaint_vae_decoder_fp16.onnx` | 94.5 MB |
| `sd15_inpaint_vae_encoder_fp16.onnx` | 65.3 MB |

Pack: 2,134,310,912 bytes, SHA-256
`06c2f94bef4d03f8ea09955b5452ad0b5fb0c23dca79008089198cc1276587eb`.

**Licence: [CreativeML Open RAIL-M](LICENSE-CreativeML-Open-RAIL-M.md)**,
with the same Attachment A restrictions as above. Weights by Robin
Rombach, Patrick Esser and contributors
([stable-diffusion-inpainting](https://huggingface.co/stable-diffusion-v1-5/stable-diffusion-inpainting));
ONNX float16 export by
[RanaLLC](https://huggingface.co/RanaLLC/stable-diffusion-v1-5-inpainting-onnx-fp16).

## Using a pack

darkmoon offers to download the generative pack the first time Generative
fill is used, and keeps it in the folder it was installed into, beside
the models that shipped with it. Nothing has to be done by hand.

To install one manually — on a machine with no connection, say — put the
assembled `.darkmoon` file in that same folder:

| | |
|---|---|
| Windows | `models\` beside `darkmoon.exe` |
| macOS | `darkmoon.app/Contents/Resources/models/` |
| Linux | `models/` beside the `darkmoon` binary |

The dialog in the application prints the exact path for the machine it is
running on. A pack is found by being in that folder; nothing else is
needed, and any pack found there is used.

Up to darkmoon v1.18.0 downloads went to the user's application data
directory instead (`%LOCALAPPDATA%\darkmoon\models`,
`~/Library/Application Support/darkmoon/models`,
`~/.local/share/darkmoon/models`). That folder is still searched, so a
pack already fetched there does not have to be fetched again.

## Checking a download

```
sha256sum models-generative.darkmoon
```

against the digest above. darkmoon does this itself and refuses to
install a pack that does not match — a partial or damaged download is
discarded rather than kept.
