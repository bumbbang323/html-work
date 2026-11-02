# Material textures for fibre-vat.html

These three greyscale images are already embedded in `fibre-vat.html` as data URIs; the copies here are for reference and re-processing. They were generated with an image model (1024×1024), then converted to greyscale, high-pass flattened, made seamless by half-offset cross-fading, contrast-reduced and downscaled. None of them carries data, and the page works the same with any of them removed (each layer falls back to a flat colour).

| File | Layer in the page | Blend | Prompt summary |
| --- | --- | --- | --- |
| `paper-grain-512.jpg` | page background, tiled at 512 px | `background-blend-mode: soft-light`, mean grey 50 % so only grain is added | top-down handmade cotton-rag paper surface, seamless, grey, very low contrast, no text |
| `vat-lightfield-384.jpg` | vat floor behind the WebGL canvas, stretched to cover | `background-blend-mode: screen`, pre-darkened to 0–20 % | smooth off-centre bright-field illumination falloff, no specimen, no text |
| `couch-felt-384.jpg` | the felt under the saved-sheet pile, tiled at 192 px | `background-blend-mode: soft-light` | papermaker's wool felt / fibre-network noise with faint twill, seamless, grey |
