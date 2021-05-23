# EvE Model Extraction
Included are a set of files to extract 3D models from EVE Online.

If you have any questions, eve-mail me at Kei Aozora.

### CCP Copyright Notice

EVE Online, the EVE logo, EVE and all associated logos and designs are the intellectual property of CCP hf. All artwork, screenshots, characters, vehicles, storylines, world facts or other recognizable features of the intellectual property relating to these trademarks are likewise the intellectual property of CCP hf. EVE Online and the EVE logo are the registered trademarks of CCP hf. All rights are reserved worldwide. All other trademarks are the property of their respective owners. CCP is in no way responsible for the content on or functioning of this website, nor can it be liable for any damage arising from the use of this website.

## Setup
To set folder, point to \EVE\sharedcache\tq

Ship models are located in \dx9\model\ship\

.gr2 files are the model data.

.dds files are the texture data.

## Model
The included TriExporter supports exporting .gr2 to .obj, .3ds, and a few other formats (File>Export model...). Use that.

For some reason, the UV map for the models are upside down (at least in Blender). To fix this, use the UV editor, select all vertices and scale -1 in the Y axis. 

## Texture
The usual texture maps are packed into different channels inside several .dds files, namely those ending with \_ar, \_no, and \_pdmg.

### Breakdown

#### \_ar file:

RGB : Albedo

A   : Glossiness

#### \_no file:

G   : Normal map X

A   : Normal map Y

B   : Ambient Occlusion

#### \_pmdg file:

R   : Paint

G   : Material

B   : Dirt

A   : Glow

(Note: R=Red, G=Green, B=Blue, A=Alpha)

You can use an image processing software (such as GIMP) to separate all of these out manually. Alternatively, you can directly process (most) .dds files inside Blender with relative ease thanks to the node-based shading system it uses.

### How to unpack (Blender):

_An implementation of the method listed below can be found inside the included sample file._

Import the .dds files with the Image Texture node and separate the channels with a Separate RGB node.

#### Color
1) Plug the material map (\_pdmg G) into a Color Ramp node (constant interpolation, stops at (0, 0.25, 0.5, 0.75), color can be arbitrary). The color ramp determines the base color of the model.
2) Add 1) and the paint map (\_pdmg R) with the MixRGB node (Add, factor 1).
3) Subtract the AO map (\_no B) (color1) and the albedo map (\_ar RGB) (color2) with the MixRGB node (Subtract, factor 0.2).
4) Multiply 2) and 3) with the MixRGB node (Multiply, factor 1).
5) Multiply 4) and the dirt map (\_pdmg B) with the MixRGB node (Multiply, factor arbitrary).
6) Plug into Base Color input

#### Metallic
1) Plug the material map (\_pdmg G) into a Color Ramp node (constant interpolation, stops at 0 and where the metallic cutoff is (match with color output), color black and white, respectively).
2) Plug into Metallic input

#### Roughness
~~1) Invert the glossiness map (\_ar A) with the Invert node (factor 1).~~

Currently doesn't work, and I don't know why. To work around this, use GIMP to separate out the alpha map from the \_ar file, invert it, export as .png.

2) Clamp the output of 1) with the Math node (Multiply, check Clamp, value arbitrary (default 1)). This is to tone the how rough the texture will be.
3) Plug into Roughness input


#### Emission
1) Plug the glow map (\_pmdg A) into a Color Ramp node (ease interpolation, stops at (0, arbitrary), color black and arbitrary, respectively).
2) Plug into Emission input

#### Normal
The normal map cannot be processed directly inside Blender. The following will use GIMP, but any image editing software should be fine.

(In GIMP)

1) Decompose the \_no file (Color>Components>Decompose) (Color model RGBA, Decompose to layers checked).
2) Compose (Color>Components>Compose) with Red channel as green layer, Green channel as alpha layer, and Blue channel as Mask value (255).
3) Export as .png.

(In Blender)

4) Plug the normal map into a Normal Map node (Tangent Space, strength arbitrary)
5) Plug into Normal input
