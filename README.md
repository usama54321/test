In this assignment you will implement a simple rasterizer, including features like supersampling, hierarchical transforms, and texture mapping with antialiasing. At the end, you'll have a functional vector graphics renderer that can take in modified SVG (Scalable Vector Graphics) files, which are widely used on the internet.

## Logistics

### Deadline
* Assignment 1 is due Monday, September 18th at 11:55pm. Assignments which are turned in after 11:55pm will face a penalty of 10% per day policy,up till the 6th late day.


### Getting started

You can either download the zipped assignment straight to your computer or clone it from GitHub using the command

    $ git clone https://github.com/CGatLUMS/-PA1_Rasterization-.git


### What you will turn in
You will submit your entire project directory in a *zip* file on lms.lums.edu.pk . This should include a *website* directory containing a web-ready assignment write-up in a file *index.html*.

As you go through the assignment, refer to the write-up guidelines and deliverables section below. It is recommended that you accumulate deliverables into sections in your webpage write-up as you work through the project.

*Note: Do not squander all your hard work on this assignment by converting your png files into jpg or any other format!* Leave the screenshots as they are saved by the `'S'` key in the GUI, otherwise you will introduce artifacts that will ruin your rasterization efforts.

<img src="/cs184_sp16_content/article_images/3_7.jpg" width="800px" align="middle"/>

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

* Part 1: Rasterizing single-color triangles ()
* Part 2: Antialiasing triangles ()
* Part 3: Transforms ()

There is a fair amount of code in the CGL library, which we will be using for future assignments. The relevant header files for this assignment are *vector2D.h*, *matrix3x3.h*, *color.h*, and *renderer.h*.

Here is a very brief sketch of what happens when you launch `draw`: An `SVGParser` (in *svgparser.\**) reads in the input *svg* file(s), launches a OpenGL `Viewer` containing a `DrawRend` renderer, which enters an infinite loop and waits for input from the mouse and keyboard. DrawRend (*drawrend.\**) contains various callback functions hooked up to these events, but its main job happens inside the `DrawRend::redraw()` function. The high-level drawing work is done by the various `SVGElement` child classes (*svg.\**), which then pass their low-level point, line, and triangle rasterization data back to the three `DrawRend` rasterization functions.

Here are the files you will be modifying throughout the project:

1. *drawrend.cpp*, *drawrend.h*
2. *texture.cpp*
3. *transforms.cpp*
4. *svg.cpp*

In addition to modifying these, you will need to reference some of the other source and header files as you work through the project.

### Part 1: Rasterizing single-color triangles (___)
[**Relevant lecture: 2**](https://cs184.org/lecture/sampling/slide_021)

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



### Part 2: Antialiasing triangles (___)
[**Relevant lecture: 3**](https://cs184.org/lecture/antialiasing/slide_095)

Use supersampling to antialias your triangles. The `sample_rate` parameter in `DrawRend` (adjusted using the `-` and `=` keys) tells you how many samples to use per pixel.

The image below shows how sampling four times per pixel produces a better result than just sampling once, since some of the supersampled pixels are partially covered and will yield a smoother edge.

<img src="/uploads/article_images/3_1.jpg" width="500px" align="middle"/>

To do supersampling, each pixel is now divided into `sqrt(sample_rate) * sqrt(sample_rate)` sub-pixels. In other words, you still need to keep track of `height * width` pixels, but now each pixel has `sqrt(sample_rate) * sqrt(sample_rate)` sampled colors. You will need to do point-in-triangle tests at the center of each of these *sub-pixel* squares.

We provide a `SampleBuffer` class to store the sub-pixels. Each samplebuffer instance stores one pixel. Your task is to fill every sub-pixel with its correctly sampled color for every samplebuffer, and average all sub-pixels' colors within a samplebuffer to get a pixel's color. Since you've finished Part 1, you can use `SampleBuffer::fill_color()` function to write color to a sub-pixel.

Your triangle edges should be noticeably smoother when using > 1 sample per pixel! You can examine the differences closely using the pixel inspector. Also note that, it may take several seconds to switch to a higher sampling rate.

For convenience, here is a list of functions you will need to modify:

1. `DrawRend::rasterize_triangle`
2. `SampleBuffer::get_pixel_color`

**Extra Credit:** Implement an alternative sampling pattern, such as jittered or low-discrepancy sampling. Create comparison images showing the differences between grid supersampling and your new pattern. Try making a scene that contains aliasing artifacts when rendered using grid supersampling but not when using your pattern. You can also try to implement more efficient storage types for supersampled framebuffers, instead of using one samplebuffer per pixel.


### Part 3: Transforms ()
**Relevant lecture: 4**

Implement the three transforms in the *transforms.cpp* file according to the [SVG spec](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/transform). The matrices are 3x3 because they operate in homogeneous coordinates -- you can see how they will be used on instances of `Vector2D` by looking at the way the `*` operator is overloaded in the same file.

Once you've implemented these transforms, *svg/transforms/robot.svg* should render correctly, as follows:

<img src="/uploads/article_images/3_.jpg" width="400px" align="middle"/>

For convenience, here is a list of functions you will need to modify:

1. `translate`
2. `scale`
3. `rotate`

**Extra Credit:** Add an extra feature to the GUI. For example, you could make two unused keys to rotate the viewport. Save an example image to demonstrate your feature, and write about how you modified the SVG to NDC and NDC to screen-space matrix stack to implement it.

Reference:
Berkeley course - https://gitlab.com/cs184
