---
layout: post
title:  "Presenting the Presentation Update: AMD FidelityFX Super Resolution, FXAA, CAS, Variable Refresh Rate, and more!"
date:   2022-01-29
categories: updates
excerpt_separator: "<!--more-->"
---

Hello again! It’s [Triang3l](https://twitter.com/Triang3l), the graphics programmer of Xenia, and this is the second post in the Xenia blog about the graphics subsystems in the emulator.

This time, we’ll cover the **new huge update** to the other, host side of graphics in Xenia — the **image presentation** architecture — what takes the game’s image output and displays it in the Xenia window.

Specifically, with this new update, ✨**Xenia will look more beautiful than it ever looked before✨ — at a low, or even near-zero, performance cost!** This is achieved thanks to the magical wonders of the image post-processing effects developed in the industry, as well as changes to the emulator that help the newly added filters shine in an Xbox 360 emulation environment.

Additionally, the update brings OS presentation and window interaction reworks, in particular, that help **reduce the image output latency and provide more stable frame pacing**.

In this post, we’ll cover:

* Post-processing effects added in the new update:
    * **The gorgeous AMD FidelityFX Super Resolution 1.0**, also known as **FSR**, for **very high-quality edge-preserving and sharpness-restoring upscaling** even from sub-720p to the resolutions of modern monitors!
    * **AMD FidelityFX Contrast Adaptive Sharpening**, or **CAS**, **enhancing the fidelity of the picture** especially in the areas where it’s needed the most.
    * **NVIDIA Fast Approximate Anti-Aliasing 3.11** — **FXAA** — **eliminating jaggies in the image** regardless of what anti-aliasing method the game uses internally, or if it doesn’t at all, and especially **useful in combination with FSR or CAS** so they have good edges to enhance.
    * **Dithering** with a blue noise pattern to **increase the smoothness of gradients**.
* **Non-square resolution scaling** added recently — most interestingly, the **1x2 and 2x1 modes**, which **significantly improve the amount of detail with a much smaller performance impact than 2x2** — now **providing near-native quality when combined with FSR and FXAA**!
* **Variable Refresh Rate support**, as well as **presentation threading improvements**, for reducing the latency before the game’s output appears on screen.
* **High DPI monitor support** and other window system interaction enhancements.

**You can download the updated release of Xenia from the [download page on the Xenia website](/download/)**, or using the **link in the #📝links-and-related-servers📝 channel on the [Xenia Discord server](https://discord.gg/Q9mxZf9)**.

![Before — no FXAA, no FSR or CAS, no dithering](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/comparison_before.jpg)

_[Halo 3 on Xenia before](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/comparison_before.jpg)_

![After — with FXAA, no FSR or CAS, no dithering](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/comparison_after.jpg)

_[Halo 3 with FXAA (extreme quality preset) and FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/comparison_after.jpg)_

<!--more-->
Note that due to the download quota on AppVeyor that Xenia has been experiencing issues with lately, **the old download location is not available anymore**. Now, as of the end of January 2022, **Xenia builds are hosted as [GitHub Releases in a special repository](https://github.com/xenia-project/release-builds-windows/releases)** — thanks to our core contributor [Rick Gibbed](https://twitter.com/gibbed), the transition to the new location was quick and largely seamless. However, **the best way to update Xenia is to use the [download page on the Xenia website](/download/) which contains the most up-to-date download link**.

We also want to **thank the users of the [official DirectX Discord server](https://discord.com/invite/directx)** for providing highly detailed answers to questions even about the most obscure parts of the presentation architecture on Windows — most importantly **[Jesse Natalie](https://github.com/jenatali)**, also known as **SoldierOfLight**, a Principal Software Engineer on the DirectX team at Microsoft, whose primary focus includes the DXGI presentation infrastructure on Windows, and **[Kaldaien](https://github.com/Kaldaien)**, the developer of the [Special K](https://wiki.special-k.info/) game tweaking and modifying framework. The server is a super helpful and friendly place for everyone developing Windows games and applications utilizing the DirectX frameworks, as well as for rendering engineers and programmers working with GPUs in general.

Don’t forget to **join the [Xenia Discord server](https://discord.gg/Q9mxZf9)** where you can share your emulation experience, get support from the Xenia community, and discuss the Xbox 360 and its internals.

If you want to **support the development of Xenia financially**, you can **[become a patron of Xenia on Patreon](https://www.patreon.com/xenia_project)**, and now you can also **sponsor individual core contributors via the 💜 Sponsor button in the [Xenia repository on GitHub](https://github.com/xenia-project/xenia)**.

# Highest-quality pixels, out of what we have

The most shiny part of the new update is the addition of image post-processing effects.

Most Xbox 360 games have an internal rendering resolution of 1280x720, or often even smaller. This is way lower — more than 2 times smaller — than the pixel count of 1080p TV screens that the Xbox 360 was commonly used with. To achieve a high visual quality even with a much lower rendering resolution, the Xbox 360 uses hardware video output scaling¹ that performs advanced filtering involving a large number of samples of the source image for each output pixel.

The hardware scaler, however, is a largely unresearched part of the console, and so far, Xenia has been using the most primitive approach — bilinear filtering — to stretch the image if the size of the window differs from the game’s output resolution. This produces a very low-quality image, with heavily blurred edges and details, often with “star” patterns along the edges.

But since the time of the development of the Xenos GPU of the Xbox 360, a lot of interesting image processing developments have been made in the industry — and now **several effects are available in Xenia for high-quality scaling**!

The **menu where the effects can be toggled and configured** can be opened by pressing **Display → Post-processing settings** in the menu bar, or **F6** on the keyboard. Even though these effects were developed by AMD and NVIDIA, they **work on all graphics cards** being purely open-source mathematical algorithms.

![The post-processing settings menu](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/settings.png)

_[The post-processing settings menu](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/settings.png)_

¹ _Contrary to popular belief, scaling is not a part of the functionality of the ANA or the HANA chip — rather, it appears to be performed by the display controller component of the ATI Avivo system that was implemented in the highly experimental R400-based Xenos GPU — however, correct us if we’re wrong!_

## NVIDIA Fast Approximate Anti-Aliasing 3.11 — FXAA

While true geometry anti-aliasing — multisample anti-aliasing (MSAA) — has no per-pixel bandwidth cost on the Xbox 360 thanks to its GPU’s architecture containing a dedicated chip that performs per-sample blending and depth/stencil culling with frame buffers placed in its internal super-fast eDRAM (this was covered in detail in the “[Leaving No Pixel Behind: New Render Target Cache](/updates/2021/04/27/leaving-no-pixel-behind-new-render-target-cache-3x3-resolution-scaling.html)” post in the Xenia blog), it definitely didn’t come without a development cost. Due to the limited amount of the eDRAM, game developers had to use tile-based rendering even for 2x MSAA at 1280x720, which often required large-scale reorganization of the frame rendering pipeline of the game engine. In addition, as with MSAA, one pixel doesn’t correspond to a single position in the world anymore, integration of various screen-space effects is complicated with MSAA.

Because of that, several games — including titles as legendary as Halo 3 — have no geometry anti-aliasing in their rendering pipelines, and “jaggies” are highly prominent in them if the image is displayed as is. And even when games used MSAA, it was very often limited to just 2 samples, as 4x MSAA requires a larger number of tiles, so geometry processing costs are higher because transformations for objects appearing in multiple tiles need to be performed multiple times — or something like [HDR tone mapping being done after resolving MSAA](http://theagentd.blogspot.com/2013/01/hdr-inverse-tone-mapping-msaa-resolve.html) (as the Xbox 360 doesn’t have the functionality for reading directly from the eDRAM in pixel shaders for per-sample post-processing before calculating the averaged color for the pixel within the aforementioned fast render backend chip) simply ruined the quality of MSAA in some games.

Additionally, MSAA in its pure form only combats geometry aliasing along the edges of polygons, but it doesn’t help eliminate aliasing caused by what’s done inside pixel shaders — some common examples of aliasing of this nature in games are surfaces with per-pixel texture cutout (like grates, foliage), shadows at sharp viewing angles, and thin lighting details.

Now, Xenia includes **FXAA 3.11** — the **[Fast Approximate Anti-Aliasing](https://en.wikipedia.org/wiki/Fast_approximate_anti-aliasing)** filter created by Timothy Lottes at NVIDIA. Being an image filter, while not being able to reconstruct thin objects that have disappeared from the final image, it analyzes edges in the image and smoothens them — **removing spatial aliasing regardless of whether it was caused by geometry or by pixel shading**, with **coherent result even in motion due to sub-pixel aliasing removal**, which makes it **great for combining with upscaling and sharpening filters** — AMD FidelityFX Super Resolution and Contrast Adaptive Sharpening — also added to Xenia in this update.

**Two FXAA options** are available in Xenia:

* **Normal quality** — may be preferable for additional aliasing cleanup when the game already has high-quality anti-aliasing like 4x MSAA. The image is sharper, but very small details may occasionally still look noisy. Corresponds to the default quality preset 12 in the FXAA shader.
* **Extreme quality** — recommended for games without anti-aliasing or with 2x MSAA. The picture is smoother, more uniform and more coherent in motion, but may look too blurry. Uses the maximum FXAA shader quality preset of 39, and a higher sub-pixel aliasing removal quality and more sensitive edge thresholds.

Note that FXAA is a post-processing filter applied to the final image, which includes text, thin lines, and other elements that are not part of the 3D scene, and the legibility of them may be obstructed by FXAA — so, if the game already has anti-aliasing that makes FXAA in Xenia unnecessary, such as, the game has built-in FXAA, or an even smoother solution like temporal anti-aliasing (TAA), or 4x MSAA with no prominent shading aliasing, it’s recommended not to turn on FXAA in Xenia. While _FXAA commonly helps AMD FidelityFX Super Resolution (FSR) detect correct edges_, if it’s combined anti-aliasing in the game that is already smooth enough, it may _excessively blur the edges, worsening the result of FSR_ in some cases — when enabling FXAA, check if it actually makes the game look more comfortable, and choose the right quality option for the game.

The following FXAA comparison screenshots were taken at different game frames — animated objects may look different, but static edges are the same. You’ll see Halo 3 many times in this post — not only because it’s a legendary masterpiece, but also since the image quality in it heavily depends on post-processing, as the game has no built-in anti-aliasing, and its rendering resolution is below 720p, upscaled by the hardware scaler.

_(FXAA comparison screenshots taken at different game frames, in-game animation may slightly differ)_

![No FXAA — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_off.jpg)

_[No FXAA — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_off.jpg)_

![FXAA normal quality — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_normal.jpg)

_[FXAA normal quality — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_normal.jpg)_

![FXAA extreme quality — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_extreme.jpg)

_[FXAA extreme quality — Halo 3, no in-game AA, upscaled from 1152x640 to 1280x720 with FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_halo3_extreme.jpg)_

![No FXAA — the building has a jagged shadow — Sonic the Hedgehog, 2x MSAA, 1280x720, no scaling/sharpening](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_sonic_off.jpg)

_[No FXAA — notice the jagged shadow of the building — Sonic the Hedgehog, 2x MSAA, 1280x720, no scaling/sharpening](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_sonic_off.jpg)_

![FXAA extreme quality — the shadow aliasing is gone now — Sonic the Hedgehog, 2x MSAA, 1280x720, no scaling/sharpening](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_sonic_on.jpg)

_[FXAA extreme quality — the shadow aliasing is gone now — Sonic the Hedgehog, 2x MSAA, 1280x720, no scaling/sharpening](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fxaa_sonic_on.jpg)_

_Note: FXAA is currently available only in the Direct3D 12-based GPU emulation backend in Xenia. As of January 2022, the Vulkan-based backend is unmaintained and uses the legacy architecture not including the emulation improvements present in the Direct3D 12-based GPU emulation implementation. On Vulkan, FXAA will be available in Xenia when the old Vulkan backend is replaced with the new emulation logic based on the code used in the Direct3D 12 backend. All other effects added in this update are applied on the window presentation side, which was fully rewritten, so they’re available on both backends._

## AMD FidelityFX Contrast Adaptive Sharpening — CAS

Another nice effect added in the update is AMD’s solution for restoring the sharpness of details — **[FidelityFX Contrast Adaptive Sharpening](https://gpuopen.com/fidelityfx-cas/)**.

Fine details of the image can be blurred and lost due to various reasons, such as texture filtering (or just insufficient texture resolution), post-processing anti-aliasing being too aggressive (including the FXAA filter in Xenia itself), or the combining of multiple frames in temporal anti-aliasing. Also, the amount of detail and contrast may vary between different portions of the image.

AMD FidelityFX CAS helps **restore the details of the image that have become blurry**, in a way that takes the contrast in the vicinity of the pixel into account and **makes the sharpness consistent throughout the image**, while also **not introducing ringing artifacts like glow around edges** that are often caused by lower-quality sharpening filters.

It’s important that **CAS needs to be used in combination with anti-aliasing** to avoid amplifying the aliased edges — **enabling FXAA in Xenia is heavily recommended to use CAS** if the game doesn’t have high-quality anti-aliasing internally.

In the settings window, the **additional sharpness setting** allows for controlling the amount of sharpness CAS produces, from 0 to 1 (higher is sharper). In many cases, 0 is enough — setting this to values too high may result in a pretty noisy image. However, for excessively blurry games, such as those using temporal anti-aliasing, a higher value may be desirable.

CAS is able to sharpen the image **at the original resolution**, when **downscaling**, and even when **upsampling it by factors of up to 2x2** (in case the window is more than 2x2 times larger than the game’s output resolution, Xenia automatically limits CAS scaling to 2x2, and stretches the result to the target size with bilinear filtering). However, **for upscaling, there’s another, far more interesting solution that we’ll cover shortly!**

![No CAS — the image is slightly blurred due to the upscaling done by the game outside the control of the emulator — Call of Duty 4: Modern Warfare, 2x MSAA, upscaled from 1024x600 to 1280x720 by the game, extreme quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_cod4_off.jpg)

_[No CAS — the image is slightly blurred due to the upscaling done by the game outside the control of the emulator — Call of Duty 4: Modern Warfare, 2x MSAA, upscaled from 1024x600 to 1280x720 by the game, extreme quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_cod4_off.jpg)_

![CAS, additional sharpness 0.5 — sharper and more consistent surface details — Call of Duty 4: Modern Warfare, 2x MSAA, upscaled from 1024x600 to 1280x720 by the game, extreme quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_cod4_on.jpg)

_[CAS, additional sharpness 0.5 — sharper and more consistent surface details — Call of Duty 4: Modern Warfare, 2x MSAA, upscaled from 1024x600 to 1280x720 by the game, extreme quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_cod4_on.jpg)_

![No CAS — Banjo-Kazooie, 4x MSAA, 1280x720, normal quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_bk_off.jpg)

_[No CAS — Banjo-Kazooie, 4x MSAA, 1280x720, normal quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_bk_off.jpg)_

![CAS, additional sharpness 1.0 — more cartoon-like visuals with low-resolution textures — Banjo-Kazooie, 4x MSAA, 1280x720, normal quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_bk_on.jpg)

_[CAS, additional sharpness 1.0 — more cartoon-like visuals with low-resolution textures — Banjo-Kazooie, 4x MSAA, 1280x720, normal quality FXAA](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/cas_bk_on.jpg)_

## AMD FidelityFX Super Resolution 1.0 — FSR

And here is the most powerful upsampling magic that AMD has unleashed last year — **[FidelityFX Super Resolution 1.0](https://gpuopen.com/fidelityfx-superresolution/)**, also known as **FSR**, and it is now available in Xenia too, effectively **completely solving the image upscaling problem** in the emulator at virtually no performance cost!

FSR is a **super-high-quality upsampling filter designed specifically for games**. It consists of two effects — EASU and RCAS — that work together to ensure that the fidelity of both the overall shapes and the little details are not only preserved, but even intensified.

The first pass of FSR is **EASU** — **Edge Adaptive Spatial Upsampling**. This is the part that performs the actual upscaling. Its main visual effect is **preserving hard edges of scene features with their original shapes**, so **edges of objects look similar to what they would look like in a true high-resolution rendered image**.

Consider playing a PC game in windowed mode — changing the resolution in the settings also changes the size of the window. When you set the resolution to 1280x720, you get a 1280x720 window where the game is rendered at its internal rendering resolution — objects in the scene have clearly defined edges, but they’re small. Then you change it to 1600x900 — and you get a 1600x900 window, and edges are still as sharp — but everything is bigger. Ultimately, you switch to the native 2560x1440 fullscreen mode of your monitor. And again, you get an image with clear edges of what’s drawn in it. Edges become **longer**, but **not wider**.

More precisely, **edges don’t have width at all** — they’re **just where the object ends**. It’s a hard boundary — a **point can be either within the polygon** on the screen (belongs to the object, for example, to a wall of a building), or it can be **outside the polygon** (corresponds to whatever is around the corner of that building) _(there is also the third case, when the point is exactly on the edge, but the rasterization rules treat this case as inside or as outside depending on the orientation of the edge, so it can be ignored)_. If you **increase the rendering resolution**, you get **more points inside and more points outside** the polygon.

However, this is **totally not what basic bilinear filtering does**. If you take an image where the point on the left is inside the polygon, and the point on the right is outside of it, and stretch it with bilinear filtering to make it twice as wide (or, for simplicity, just linear along one axis — bilinear is pretty much doing this first along one axis, and then along another), you’ll get two points in between — and one will be “75% inside the polygon”, and another “25% inside the polygon”. Which doesn’t make any sense at all because it’s a point in space, with just one exact position. Mathematically, an edge is a step function — but linear interpolation turns it into a linear slope. This is a problem similar to the one that is also present, for instance, in [alpha cutout textures](https://steamcdn-a.akamaihd.net/apps/valve/2007/SIGGRAPH2007_AlphaTestedMagnification.pdf) (the presentation linked has an example of raw and thresholded bilinearly-upscaled edges, and edges upscaled properly, but the exact technique used there is not applicable to the case of hardware rasterization of shaded objects to a raster image — attached to provide an example of where bilinear filtering is appropriate and where it’s not).

This “pixel is a point” model essentially represents rasterization without anti-aliasing, where whether a pixel is inside a polygon or not is determined by the center of the cell of the pixel grid on the screen. With multisample anti-aliasing (MSAA), or with temporal anti-aliasing (TAA) which spreads the general concept behind MSAA across multiple frames by adding a different sub-pixel offset, or “jitter”, to the scene geometry in each frame, multiple points scattered throughout the cell corresponding to the pixel on the screen grid are used in the inside/outside check, approximating the “pixel is a square” model. In this model, the polygon’s color is mixed with the background color depending on which fraction of the pixel’s square is within the polygon. So, if the polygon covers 75% of the square, the pixel will have 75% of its color + 25% of the background color.

But if we **make the width and the height of the framebuffer two times larger**, inside the old original square, **there will be 4 new squares, each having the size of 0.5x0.5 original pixel squares**. Some of these 4 squares may be entirely inside the polygon, some may be completely outside, some may be crossed by the edge and need their own factors of mixing, **depending on the orientation and the location of the edge**. But **bilinear filtering**, again, **doesn’t care about any of that**. Where we need a finer coverage **of smaller areas (0.5x0.5)**, it instead just mixes already mixed colors from **across a bigger area (2x2)**, again, using linear interpolation along one axis followed by linear interpolation along another, which has nothing to do with the shape of the edge.

To summarize, **you cannot bilinearly interpolate _coverage_**. Coverage is a fraction of an area, an integral of the shape bounded by its edges (a triangle in case of GPU rasterization), over an area (a square representing a pixel in the framebuffer pixel grid).

**EASU**, on the other hand, using the colors from 12 samples of the original image, quoting [the FSR page on the AMD GPUOpen website](https://gpuopen.com/fidelityfx-superresolution/), “**detects gradient reversals — essentially looking at how neighboring gradients differ — from a set of input pixels. The intensity of the gradient reversals defines the weights to apply to the reconstructed pixels at display resolution**." The algorithm also takes into account not only the intensity of the gradient reversals, but the direction of the gradient as well. This results in **razor-thin edges** that **keep their original orientation**, and that **maintain stability in motion**.

After EASU, the second part addresses another problem. **RCAS** — **Robust Contrast Adaptive Sharpening** — performs sharpening to amplify low-resolution details and to make them finer. Similar to CAS, it analyzes the local contrast to make sharpness consistent in different parts of the image, but as it doesn’t take the image as its original rendering resolution, the algorithm has some differences from CAS, it’s more exact.

The EASU part of FSR, just like CAS, is designed for scaling by factors of up to 2x2. Normally, the algorithm is used for a fixed set of upsampling factors — 1.3x, 1.5x, 1.7x or 2x per dimension, In Xenia, however, both the game’s rendering resolution and the size of the window in pixels can be completely arbitrary — for example, the user may want to upscale a 1280x720 game to a 3840x2160 display, which has 3x3 times more pixels. But remember that EASU preserves edges — after EASU, you get an image with more or less the same clear edges as in the original image, just the objects they belong to are bigger. And EASU doesn’t sharpen the picture — sharpening is performed by a separate RCAS pass. So, **in case scaling by more than 2x2** is needed, Xenia simply **applies EASU multiple times**, at each iteration upsampling by up to 2x2 the resolution of the previous result. In this example, the original 1280x720 image is first upscaled to 2560x1440, with twice as long edges, and then to the 3840x2160 target — and after that, RCAS is done to recover the blurred details.

**Note that having high-quality anti-aliasing is extremely important with FSR** — unless the game itself offers even smoother anti-aliasing, **enabling FXAA is absolutely necessary for FSR to produce good results!** Without anti-aliasing, or with one that is not highly effective, FSR will treat the “jaggies” as actual rectangular “stairs”, and it will make them stand out even more.

The **sharpness reduction slider is available in the settings**. Unlike in CAS, though, **lower values mean more sharpness** — it’s specified in stops, with 1 resulting in 50% of the maximum sharpening, and with 2 — the effective maximum — producing 25% of the sharpening. The default is 0.2, but the desired value may vary depending on the scaling ratio and how blurry the game’s rendering itself is (but 0 produces much more subtle results than 1 in CAS).

FSR, though, is designed specifically for image upsampling. For **displaying at the original resolution, or downscaling to a smaller window**, Xenia **automatically switches to CAS** — in this case, the FSR sharpness reduction setting is also ignored, and the CAS additional sharpness knob is used instead as it normally is with CAS.

FSR is especially **helpful if actual internal resolution scaling can’t be used due to accuracy/stability or performance issues**, but it may be useful when rendering oversampling is enabled as well — for example, not all games are rendered at 1280x720, and Halo 3 with 2x2 resolution scaling will be drawn at 2304x1280 — and then FSR can rescale it to the full 2560x1440 of a QHD monitor. But **the most interesting scenario of FSR usage with respect to resolution scaling** is **combining it with smaller rendering resolution factors like 1x2 or 2x1** — we’ll discuss this in the next section!

![No FSR — Halo 3, no in-game AA, extreme quality FXAA, 1152x640 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_halo3_off.jpg)

_[No FSR — Halo 3, no in-game AA, extreme quality FXAA, 1152x640 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_halo3_off.jpg)_

![FSR — Halo 3, no in-game AA, extreme quality FXAA, 1152x640 upscaled to 2304x1280 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_halo3_on.jpg)

_[FSR — Halo 3, no in-game AA, extreme quality FXAA, 1152x640 upscaled to 2304x1280 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_halo3_on.jpg)_

![No FSR — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_bknb_off.jpg)

_[No FSR — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_bknb_off.jpg)_

![FSR — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_bknb_on.jpg)

_[FSR — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_bknb_on.jpg)_

![No FSR — Viva Piñata, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_vp_off.jpg)

_[No FSR — Viva Piñata, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_vp_off.jpg)_

![FSR, sharpness reduction 1.0 — Viva Piñata, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_vp_on.jpg)

_[FSR, sharpness reduction 1.0 — Viva Piñata, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_vp_on.jpg)_

![No FSR — Mirror’s Edge, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_me_off.jpg)

_[No FSR — Mirror’s Edge, 2x MSAA, extreme quality FXAA, 1280x720 bilinear-stretched to 2560x1440](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_me_off.jpg)_

![FSR — Mirror’s Edge, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_me_on.jpg)

_[FSR — Mirror’s Edge, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/fsr_me_on.jpg)_

The negative texture LOD biasing (or, in case of different horizontal and vertical scales, texture coordinate gradient scaling) that is recommended by the developers of FSR for restoring the details lost due to texture filtering, however, has not been implemented in Xenia as of this update. It would require architecturally pretty complex feedback from host image presentation to texture fetching in the emulation logic, and may cause unwanted side effects if game shaders treat texture mipmaps in a special way. But the primary reason is that there is already a substantial amount of shading aliasing in games, especially in motion, and that sometimes causes a dissonance between clearly-defined temporally-coherent object shapes and noisy surfaces — and unlike more modern times, very few games in the Xbox 360 library contain a temporal anti-aliasing (TAA) solution to eliminate this kind of aliasing. However, texture coordinate gradient adjustment may be something to experiment with in the future.

Note that FSR is applied to the final image the game sends to the display. In some games, the resolution of the final output matches the 3D scene rendering resolution even if it’s not 1280x720, for example, in Halo 3, which draws the scene in 1152x640, and sends the 1152x640 image directly to the display controller. In this case, FSR provides the best quality. However, some games partially do upscaling by themselves, such as Call of Duty 4: Modern Warfare, rendering the scene at 1024x600 and then stretching it to 1280x720 before drawing the HUD. In those games, the result will be less sharp as the edges are blurred by the game, which is outside the control of the emulator.

## Non-square rendering resolution scaling

While this is not exactly a part of this update — this functionality was added to the emulator last month (though it still hasn’t been covered in this blog!) — the new post-processing additions are precisely what it needed to combine its low performance overhead with comfortable, high-fidelity visuals!

Previously, only two rendering resolution scale factors were available in Xenia — 2x2 (in games with 1280x720 rendering resolution, to 2560x1440) and 3x3 (from 1280x720 to 3840x2160, or 4K).

Because framebuffers on the Xbox 360, unlike on the PC with microarchitecture-specific data layouts and immediate-mode rendering, share the eDRAM space, and in many cases intentionally inherit the data from each other by reusing what has already been written to the eDRAM, Xenia often has to copy data between PC framebuffers to preserve the eDRAM contents across framebuffer configuration changes. The “[Leaving No Pixel Behind: New Render Target Cache](/updates/2021/04/27/leaving-no-pixel-behind-new-render-target-cache-3x3-resolution-scaling.html)” post in the Xenia blog contains a detailed description of the framebuffer emulation issues. In many games (note that none of this has to be done on the real console at all), the copying amounts to a substantial fraction of the time of the frame.

Thus, even 2x2 resolution scaling is quite costly. Not only 4 times as many pixels of the scene need to be rasterized, depth-tested, shaded and written — but 4 times more framebuffer data needs to be copied as well. In several games on Xenia, stable 30 FPS can’t be maintained with 2x2 resolution scaling even on mid-range modern systems.

Now, a portion of the rendering resolution scaling logic has been generalized, and Xenia now supports **separate horizontal and vertical scaling factors**!

The `draw_resolution_scale` configuration variable has been replaced with two — **`draw_resolution_scale_x`**, which controls the **horizontal** scaling factor, and **`draw_resolution_scale_y`**, for **vertical** scaling.

**Integer factors from 1 to 3** are supported (the limit is still 3 primarily to the infamous [half-pixel offset](https://aras-p.info/blog/2016/04/08/solving-dx9-half-pixel-offset/) in Direct3D 9 on the Xbox 360, the effect of which is partially worked around in Xenia by duplicating the second column/row of pixels into the first when copying framebuffers to textures — but various one-pixel gaps are still present in some games with resolution scaling; a 4x factor would cause the gaps to be 2 pixels wide, which would require a different hack inside Xenia).

This change adds the most interesting resolution scaling scenario — **scaling by a factor of 1x2 or 2x1**, from 1280x720 to 1280x1440 or 2560x720. Proper 1920x1080 can’t be implemented reliably as that would require a fractional scale — 1.5x1.5 — which would cause ambiguity when odd coordinates are involved (especially with the half-pixel offset, again).

1x2 or 2x1 resolution scaling increases the pixel count by just 2, so it’s much lighter than full 2x2 scaling. However, because along one axis the image is still below the native resolution on a 1080p or a 1440p display, with simple bilinear filtering, still looks pretty blurry.

But now, **1x2 or 2x1 scaling can be combined with AMD FidelityFX Super Resolution**, as well as **FXAA**. The combination of slight rendering supersampling with high-quality image resizing provides a **balanced, systemic solution for increasing image fidelity** in games running on Xenia. Increasing the rendering resolution adds the texture and shading details, and then FXAA smoothens the edges, most importantly those that don’t have enough resolution — vertical edges with 1x2 scaling, horizontal with 2x1 — and finally, FSR upscales the image keeping clear shapes of the objects in the scene in the EASU pass and amplifying the details using RCAS. The final image has object edges looking almost like with native resolution rendering, sufficient detail, and a reasonable amount of shading aliasing in motion.

Rendering resolution scaling, though, is still a hack that games are completely unaware of, so there may be various issues — effects involving custom texture filtering logic (bloom, blur, shadow mapping) may display stripe/grid patterns or be not as strong as intended, gaps as a result of the half-pixel offset becoming whole-pixel, brightness in HDR may be computed for sides/corners of the image rather than the entire screen, and sometimes intermediate framebuffer textures may be flickering or missing at all.

![Halo 3 with 1x1 resolution scale, no in-game AA, extreme quality FXAA, 1152x640 upscaled to 2304x1280 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_1x1.jpg)

_[Halo 3 with 1x1 resolution scale, no in-game AA, extreme quality FXAA, 1152x640 upscaled to 2304x1280 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_1x1.jpg)_

![Halo 3 with 1x2 resolution scale, no in-game AA, extreme quality FXAA, 1152x1280 upscaled to 2304x1440 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_1x2.jpg)

_[Halo 3 with 1x2 resolution scale, no in-game AA, extreme quality FXAA, 1152x1280 upscaled to 2304x1440 and then to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_1x2.jpg)_

![Halo 3 with 2x2 resolution scale, no in-game AA, extreme quality FXAA, 2304x1280 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_2x2.jpg)

_[Halo 3 with 2x2 resolution scale, no in-game AA, extreme quality FXAA, 2304x1280 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_halo3_2x2.jpg)_

![Banjo-Kazooie: Nuts & Bolts with 1x2 resolution scale, 2x MSAA, extreme quality FXAA, 1280x1440 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_bknb_1x2.jpg)

_[Banjo-Kazooie: Nuts & Bolts with 1x2 resolution scale, 2x MSAA, extreme quality FXAA, 1280x1440 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_bknb_1x2.jpg)_

![Viva Piñata with 1x2 resolution scale, 2x MSAA, extreme quality FXAA, 1280x1440 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_vp_1x2.jpg)

_[Viva Piñata with 1x2 resolution scale, 2x MSAA, extreme quality FXAA, 1280x1440 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_vp_1x2.jpg)_

![Mirror’s Edge with 2x1 resolution scale, 2x MSAA, extreme quality FXAA, 2560x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_me_2x1.jpg)

_[Mirror’s Edge with 2x1 resolution scale, 2x MSAA, extreme quality FXAA, 2560x720 upscaled to 2560x1440 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/resolution_me_2x1.jpg)_

## 10 bits per channel precision and dithering

Internally, all the post-processing effects are performed with an **intermediate precision of 10 bits per channel** (or even more for the luminance in FXAA) so they can be chained with no precision loss potentially introducing banding, and because several games send a 10bpc image to the display.

Xenia is now also able to **dither the result to 8bpc** using a **[blue noise](https://surma.dev/things/ditherpunk/) pattern that is smooth and uniform-looking** when viewed from a distance. Though the effect is rarely noticeable, and nothing is changed by it when the game outputs a 8bpc image with no post-processing or scaling done by Xenia (and that’s why it’s enabled by default unlike the rest of the effects which apply major changes), it can improve the smoothness of gradients especially in games outputting a 10bpc image, such as Halo 3, especially when FSR is enabled, which can make banding in the original image stronger as it may consider it a feature in the image.

Compare the following screenshots of Halo 3 with brightness and contrast increased:

![No dithering — highly visible banding in the sky, the menu background also has a striped pattern](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/dither_off.png)

_[No dithering — highly visible banding in the sky, the menu background also has a striped pattern](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/dither_off.png)_

![Dithering enabled — the sky and the menu background are much smoother](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/dither_on.png)

_[Dithering enabled — the sky and the menu background are much smoother](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/dither_on.png)_

## Post-processing configuration variables

Aside from using the **post-processing settings menu** that can be opened by pressing **F6** or **Display → Post-processing settings** in the menu bar, the effects can be configured via the [Xenia configuration file](https://github.com/xenia-project/xenia/wiki/Options):

* **`postprocess_antialiasing`** — anti-aliasing filter
    * `fxaa` — use FXAA with the normal quality preset
    * `fxaa_extreme` — use FXAA with the extreme quality preset
    * Anything else — don’t apply an anti-aliasing filter
* **`postprocess_scaling_and_sharpening`** — scaling and sharpening filter
    * `cas` — use AMD FidelityFX CAS for sharpening and for upscaling to up to 2x2
    * `fsr` — use AMD FidelityFX FSR for upscaling and sharpening, or CAS when not upscaling
    * Anything else — don’t apply a scaling and sharpening filter
* **`postprocess_ffx_cas_additional_sharpness`** — additional sharpness in CAS, higher is sharper, 0 to 1, the default value is 0
* **`postprocess_ffx_fsr_sharpness_reduction`** — sharpness reduction in FSR, lower is sharper, 0 to 2, the default value is 0.2
* **`postprocess_dither`** — whether to apply dithering noise to smoothen gradients

While **game-specific configuration files are currently largely unsafe** as changing many of the settings while the emulator is already running is not supported — the game configuration file feature is currently in a prototype state — **specifying the post-processing options listed above in per-game configuration files is supported** explicitly (but **not the resolution scale** variables, which take effect during the initialization of the emulator).

# Variable Refresh Rate and lowering latency

This update includes a completely reworked image presentation architecture, and one of the changes is the support for **Variable Refresh Rate (VRR) on AMD FreeSync, NVIDIA G-SYNC and VESA Adaptive-Sync monitors in fullscreen**.

Games on the Xbox 360 in the NTSC regions usually have the frame rate locked at 30 or 60 FPS. However, modern displays may have a refresh rate that is not a multiple of 60 — such as 144 Hz. And even when the refresh rate of the monitor and the target frame rate of the game align, the actual frame rate may drop below the target one.

Now, when Xenia is in the borderless fullscreen mode (Display → Fullscreen in the menu, or F11), and nothing (like other windows on the display) forces window composition preventing Xenia from presenting independently — the “independent flip” presentation mode is active (you can learn more about the presentation modes on Windows in [Jesse Natalie’s talk](https://www.youtube.com/watch?v=E3wTajGZOsA) about them), a **VRR-compatible monitor will try to display the image as quickly as possible to reduce the latency and to stabilize the visible frame pacing even if its frame rate is below the refresh rate of the monitor**.

Note that on non-VRR displays, or on VRR-compatible monitors as well or in some extreme cases, tearing may be observed in fullscreen because supporting VRR implies allowing the monitor to display the image at any moment. Xenia prefers lowering latency over preventing tearing, but if tearing is visible to you, and you want to prevent it, you can use the following [configuration options](https://github.com/xenia-project/xenia/wiki/Options) to disable it (by changing their values from `true` to `false`):

* Direct3D 12:
    * `d3d12_allow_variable_refresh_rate_and_tearing`
* Vulkan (both need to be disabled to ensure that tearing may not occur, however, disabling it may result in subpar frame pacing depending on the platform’s window system — not all platforms support the mailbox presentation model, and those who don’t will fall back to the FIFO model with forced vertical sync waits potentially blocking the UI thread):
    * `vulkan_allow_present_mode_immediate`
    * `vulkan_allow_present_mode_fifo_relaxed`

Presentation to the host display is also now done with the `DXGI_PRESENT_RESTART` option to recover from high-latency situations in games quicker once the performance returns to normal by discarding outdated frames.

Also, an experimental feature for presentation directly from the GPU emulation thread, instead of the OS paint event handler executed on the UI thread, was added, to bypass the threading and the event dispatching in the host operating system and send the image to the display infrastructure as soon as it’s ready. Though painting internally doesn’t have a frame rate limit on Windows, and a `Present` call appears to be able to block when rendering on a GPU different than the one the monitor is connected to, also potentially blocking the GPU emulation thread — this needs testing and research — on Windows, the paint message is low-priority, so to handle different input situations more safely in the future, it’s enabled by default. On GTK on GNU/Linux, however, the widget draw signal is sent at the refresh rate of the monitor (or lower), so direct presentation triggering is even more beneficial there. The configuration variable toggling this new behavior is `host_present_from_non_ui_thread`.

# High DPI support and window interaction rework

All levels of **display pixel density awareness** on Windows are now supported by Xenia:

* Windows Vista system DPI awareness
* Windows 8.1 per-monitor DPI awareness version 1
* Windows 10 Anniversary Update per-monitor DPI awareness version 1 with non-client area DPI scaling
* Windows 10 Creators Update per-monitor DPI awareness version 2

The one-pixel border issue at certain display densities or scaling factors has been fixed, also making low-latency independent presentation possible in fullscreen mode in the scenarios when this issue previously appeared.

Dear ImGui overlays of the internal UI, such as the system input dialogs in games, the post-processing settings and the debugger, are now also DPI-scaled. For the profiler overlay, DPI scaling is disabled by default, however, for cleaner text and geometry, but it can be enabled via the `profiler_dpi_scaling` configuration variable.

![Xenia with 150% display scaling — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 1920x1080 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/high_dpi.jpg)

_[Xenia with 150% display scaling — Banjo-Kazooie: Nuts & Bolts, 2x MSAA, extreme quality FXAA, 1280x720 upscaled to 1920x1080 by FSR](/images/posts/2022-01-29-presenting-the-presentation-update-amd-fidelityfx-fsr-fxaa-cas-vrr/high_dpi.jpg)_

**Drawing of Dear ImGui dialogs and the profiler is also now limited to the frame rate of the monitor** where most of the Xenia window is located, so it will not be occupying all the remaining GPU and CPU UI thread time trying to be performed at thousands of frames per second taking free computing resources from the game. On operating systems where the UI thread paint event doesn’t have implicit frame rate limiting (on Windows, there’s no implicit limit), this has no noticeable effect on the latency of game frames — the wait is interrupted once a new game frame has arrived.

The overall architecture of the interaction with the OS window and presentation has also been entirely reworked for ease of porting to other operating systems and their window systems in the future. Platform events such as surface loss and recreation are now handled safely, and window state change handling is internally more clearly defined and more unified between platforms.
