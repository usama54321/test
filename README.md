In this assignment you will implement a simple rasterizer, including features like supersampling, hierarchical transforms, and texture mapping with antialiasing. At the end, you'll have a functional vector graphics renderer that can take in modified SVG (Scalable Vector Graphics) files, which are widely used on the internet.

## Logistics

### Deadline
* Assignment 1 is due Monday, September 18th at 11:55pm. Assignments which are turned in after 11:55pm will face a penalty of 10% per day policy,up till the 6th late day.


### Getting started

<b>You can</b> either <b>download the zipped assignment straight to your computer</b> or clone it from GitHub using the command

    $ git clone https://github.com/CG452/-PA1_Rasterization-.git

We will be using CMake to build the assignments. If you don't have CMake (version >= 2.8) on your personal computer, you can install it using apt-get on Linux or Macports/Homebrew on OS X. Alternatively, you can download it directly from the CMake website.
To build the code, start in the folder that GitHub made or that was created when you unzipped the download. Run

    mkdir build; cd build

to create a build directory and enter it, then

    cmake ..
to have CMake generate the appropriate Makefiles for your system, then

    make 
to make the executable, which will be deposited in the build directory.

If you are having problems on OS X, you should first try upgrading to the latest version of Xcode and installing command line tools by running the command

    xcode-select --install

### What you will turn in
You will submit your entire project directory in a *zip* file on lms.lums.edu.pk

*Note: Do not squander all your hard work on this assignment by converting your png files into jpg or any other format!* Leave the screenshots as they are saved by the `'S'` key in the GUI, otherwise you will introduce artifacts that will ruin your rasterization efforts.

### Some advise
Assignments are 40% of the grade. They will give a massive boost to your knowledge and skills. Take them seriously!

Start Early: Get the setup out of the way as soon as possible. So you can have more time on the actual task.

And Don't Plagarize: ... You already know this one by now.

## Using the GUI

You can run the executable with the command

    ./draw ../svg/basic/test1.svg

A flower should show up on your screen. After finishing Part 3, you will be able to change the viewpoint by dragging your mouse to pan around or scrolling to zoom in and out. Here are all the keyboard shortcuts available (some depend on you implementing various parts of the assignment):

|Key | Action|
|:-----:|------|
|`' '`  | return to original viewpoint|
|`'-'`  | decrease sample rate|
|`'='` | increase sample rate|
|`'Z'` | toggle the pixel inspector|
|`'P'` | switch between texture filtering methods on pixels|
|`'L'` | switch between texture filtering methods on mipmap levels|
|`'S'` | save a *png* screenshot in the current directory|
| `'1'-'9'`  | switch between svg files in the loaded directory|

The argument passed to `draw` can either be a single file or a directory containing multiple *svg* files, as in

    ./draw ../svg/basic/

If you load a directory with up to 9 files, you can switch between them using the number keys 1-9 on your keyboard.

## Assignment structure

The Assignment has 3 parts, worth a total of 100 possible points. Some require only a few lines of code, while others are more substantial.

**Breakdown**

* Part 1: Rasterizing single-color triangles
* Part 2: Antialiasing triangles
* Part 3: Transforms 
* Part 4: Barycentric coordinates 
* Part 5: "Pixel sampling" for texture mapping

Relevant lectures:
https://lms.lums.edu.pk/portal/site/e838d91a-c7c2-40b6-b63f-86b802fa5c07/page/8264668a-1767-4682-96c8-cbdb54658c62

There is a fair amount of code in the CGL library, which we will be using for future assignments. The relevant header files for this assignment are *vector2D.h*, *matrix3x3.h*, *color.h*, and *renderer.h*.

Here is a very brief sketch of what happens when you launch `draw`: An `SVGParser` (in *svgparser.\**) reads in the input *svg* file(s), launches a OpenGL `Viewer` containing a `DrawRend` renderer, which enters an infinite loop and waits for input from the mouse and keyboard. DrawRend (*drawrend.\**) contains various callback functions hooked up to these events, but its main job happens inside the `DrawRend::redraw()` function. The high-level drawing work is done by the various `SVGElement` child classes (*svg.\**), which then pass their low-level point, line, and triangle rasterization data back to the three `DrawRend` rasterization functions.

Here are the files you will be modifying throughout the project:

1. *drawrend.cpp*, *drawrend.h*
2. *texture.cpp*
3. *transforms.cpp*
4. *svg.cpp*

In addition to modifying these, you will need to reference some of the other source and header files as you work through the project.

### Part 1: Rasterizing single-color triangles
Relevant Lecture: [https://lms.lums.edu.pk/access/content/group/e838d91a-c7c2-40b6-b63f-86b802fa5c07/Lectures/CS452_CG_L2_Rasterization.pdf]

Triangle rasterization is a core function in the graphics pipeline to convert input triangles into framebuffer pixel values. In Part 1, you will implement triangle rasterization using the methods discussed in lecture 2 to fill in the `DrawRend::rasterize_triangle(...)` function in *drawrend.cpp*.

Notes:
  * It is recommended that you implement `SampleBuffer::fill_color()` function first, so that you can see rasterized points and lines for *basic/test[1/2/3].svg*.
  * Remember to do point-in-triangle tests with the point exactly at the center of the pixel, not the corner. Your coordinates should be equal to some integer point plus (.5,.5). (As discussed in class)
  * For now, ignore the `Triangle *tri` input argument to the function. We will come back to this later.
  * You are encouraged but not required to implement the edge rules for samples lying exactly on an edge.
  * Make sure the performance of your algorithm is no worse than one that checks each sample within the bounding box of the triangle.

When finished, you should be able to render many more test files, including those with rectangles and polygons, since we have provided the code to break these up into triangles for you. In particular, *basic/test3.svg*, *basic/test4.svg*, and *basic/test5.svg* should all render correctly.

For convenience, here is a list of functions you will need to modify:

1. `DrawRend::rasterize_triangle`
2. `SampleBuffer::fill_color`

**Extra Credit:** Make your triangle rasterizer super fast (e.g., by factoring redundant arithmetic operations out of loops, minimizing memory access, and not checking every sample in the bounding box). Write about the optimizations you used. Use `clock()` to get timing comparisons between your naive and speedy implementations.


### Part 2: Antialiasing triangles

Use supersampling to antialias your triangles. The `sample_rate` parameter in `DrawRend` (adjusted using the `-` and `=` keys) tells you how many samples to use per pixel.

The image below shows how sampling four times per pixel produces a better result than just sampling once, since some of the supersampled pixels are partially covered and will yield a smoother edge.

To do supersampling, each pixel is now divided into `sqrt(sample_rate) * sqrt(sample_rate)` sub-pixels. In other words, you still need to keep track of `height * width` pixels, but now each pixel has `sqrt(sample_rate) * sqrt(sample_rate)` sampled colors. You will need to do point-in-triangle tests at the center of each of these *sub-pixel* squares.

We provide a `SampleBuffer` class to store the sub-pixels. Each samplebuffer instance stores one pixel. Your task is to fill every sub-pixel with its correctly sampled color for every samplebuffer, and average all sub-pixels' colors within a samplebuffer to get a pixel's color. Since you've finished Part 1, you can use `SampleBuffer::fill_color()` function to write color to a sub-pixel.

Your triangle edges should be noticeably smoother when using > 1 sample per pixel! You can examine the differences closely using the pixel inspector. Also note that, it may take several seconds to switch to a higher sampling rate.

For convenience, here is a list of functions you will need to modify:

1. `DrawRend::rasterize_triangle`
2. `SampleBuffer::get_pixel_color`

**Extra Credit:** Implement an alternative sampling pattern, such as jittered or low-discrepancy sampling. Create comparison images showing the differences between grid supersampling and your new pattern. Try making a scene that contains aliasing artifacts when rendered using grid supersampling but not when using your pattern. You can also try to implement more efficient storage types for supersampled framebuffers, instead of using one samplebuffer per pixel.


### Part 3: Transforms

Implement the three transforms in the *transforms.cpp* file according to the [SVG spec](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/transform). The matrices are 3x3 because they operate in homogeneous coordinates -- you can see how they will be used on instances of `Vector2D` by looking at the way the `*` operator is overloaded in the same file.

Once you've implemented these transforms, *svg/transforms/robot.svg* should render correctly, as follows:

For convenience, here is a list of functions you will need to modify:

1. `translate`
2. `scale`
3. `rotate`

**Extra Credit:** Add an extra feature to the GUI. For example, you could make two unused keys to rotate the viewport. Save an example image to demonstrate your feature, and write about how you modified the SVG to NDC and NDC to screen-space matrix stack to implement it.


### Part 4: Barycentric coordinates

Familiarize yourself with the `ColorTri` struct in *svg.h*. Modify your implementation of `DrawRend::rasterize_triangle(...)` so that if a non-NULL `Triangle *tri` pointer is passed in, it computes barycentric coordinates of each sample hit and passes them to `tri->color(...)` to request the appropriate color. Note that the barycentric coordinates are stored with type `Vector3D`, so you should use `p_bary[i]` to store the barycentric coordinate corresponding to triangle vertex $P_i$ for $i=0,1,2$.

Implement the `ColorTri::color(...)` function in *svg.cpp* so that it interpolates the color at the point `p_bary`. This function is very simple: it does not need to make use of `p_dx_bary` or `p_dy_bary`, which are for texture mapping. Note that this `color()` function plays the role of a very primitive shader.

For convenience, here is a list of functions you will need to modify:

1. `DrawRend::rasterize_triangle`
2. `ColorTri::color`

### Part 5: "Pixel sampling" for texture mapping

Familiarize yourself with the `TexTri` struct in *svg.h*. This is the primitive that implements texture mapping. For each vertex, you are given corresponding *uv* coordinates that index into the `Texture` pointed to by `*tex`.

To implement texture mapping, `DrawRend::rasterize_triangle` should fill in the `psm` and `lsm` members of a `SampleParams` struct and pass it to `tri->color(...)`. Then `TexTri::color(...)` should fill in the correct *uv* coordinates in the `SampleParams` struct, and pass it on to `tex->sample(...)`. Then `Texture::sample(...)` should examine the `SampleParams` to determine the correct sampling scheme.

The GUI toggles `DrawRend`'s `PixelSampleMethod` variable `psm` using the `'P'` key. When `psm == P_NEAREST`, you should use nearest-pixel sampling, and when `psm == P_LINEAR`, you should use bilinear sampling.

For now, you can pass in dummy `Vector3D(0,0,0)` values for the `p_dx_bary` and `p_dy_bary` arguments to `tri->color`

For this part, just pass `0` for the `level` parameter of the `sample_nearest` and `sample_bilinear` functions.

For convenience, here is a list of functions you will need to modify:

1. `DrawRend::rasterize_triangle`
2. `TexTri::color`
3. `Texture::sample`
4. `Texture::sample_nearest`
5. `Texture::sample_bilinear`


### Part 6: "Scanline Algorithm"
tenative 

Reference:
Berkeley course - https://gitlab.com/cs184
