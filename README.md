Download Link: https://assignmentchef.com/product/solved-csci-2400-performance-lab
<br>
<h1>1          Introduction</h1>

This assignment deals with optimizing memory intensive code. Image processing offers many examples of functions that can benefit from optimization. In this lab, we will consider two image processing operations: rotate, which rotates an image counter-clockwise by 90<sup>◦</sup>, and smooth, which “smooths” or “blurs” an image.

For this lab, we will consider an image to be represented as a two-dimensional matrix <em>M</em>, where <em>M<sub>i,j </sub></em>denotes the value of (<em>i,j</em>)th pixel of <em>M</em>. Pixel values are triples of red, green, and blue (RGB) values. We will only consider square images. Let <em>N </em>denote the number of rows (or columns) of an image. Rows and columns are numbered, in C-style, from 0 to <em>N </em>− 1.

Given this representation, the rotate operation can be implemented quite simply as the combination of the following two matrix operations:

<ul>

 <li><em>Transpose</em>: For each (<em>i,j</em>) pair, <em>M<sub>i,j </sub></em>and <em>M<sub>j,i </sub></em>are interchanged.</li>

 <li><em>Exchangerows</em>: Row <em>i </em>is exchanged with row <em>N </em>− 1 − <em>i</em>.</li>

</ul>

This combination is illustrated in Figure 1.

The smooth operation is implemented by replacing every pixel value with the average of all the pixels around it (in a maximum of 3 × 3 window centered at that pixel). Consider Figure 2. The values of pixels M2[1][1] and M2[N-1][N-1] are given below:

M2

M2

<strong>j                                                         i</strong>

Figure 1: Rotation of an image by 90<sup>◦ </sup>counterclockwise

Figure 2: Smoothing an image

<h1>2          Hand Out Instructions</h1>

Start by copying perflab-handout.tarto the directory in which you plan to do your work. Then give the command: tar xvf perflab-handout.tar. This will cause a number of files to be unpacked into the directory. The only file you will be modifying and handing in is kernels.c. The driver.c program is a driver program that allows you to evaluate the performance of your solutions. Use the command make driver to generate the driver code and run it with the command ./driver.

Looking at the file kernels.c you’ll notice a C structure student into which you should insert the requested identifying information. Do this right away so you don’t forget.

<h1>3          Implementation Overview</h1>

<h2>Data Structures</h2>

The core data structure deals with image representation. A pixel is a struct as shown below:

typedef struct { unsigned short red; /* R value */ unsigned short green; /* G value */ unsigned short blue; /* B value */

} pixel;

As can be seen, RGB values have 16-bit representations (“16-bit color”). An image I is stored as a one-dimensional array of pixels, where the (<em>i,j</em>)th pixel is I[RIDX(i,j,n)]. Here n is the dimension of the image matrix, and RIDX is a macro defined as follows: #define RIDX(i,j,n) ((i)*(n)+(j))

See the file defs.h for this code.

You should think of I[RIDX(i,j,n)] as equivalent to I[i][j] for most purposes – the reason RIDX is used at all is because it allows run-time changes of the array size, which is needed for the testing/grading code.

<h2>Rotate</h2>

The following C function computes the result of rotating the source image src by 90<sup>◦ </sup>and stores the result in destination image dst. dim is the dimension of the image.

void naive_rotate(int dim, pixel *src, pixel *dst)

{

int i, j; for (i = 0; i &lt; dim; i++){ for (j = 0; j &lt; dim; j++){ dst[RIDX(dim-1-j, i, dim)].red = src[RIDX(i, j, dim)].red; dst[RIDX(dim-1-j, i, dim)].green = src[RIDX(i, j, dim)].green; dst[RIDX(dim-1-j, i, dim)].blue = src[RIDX(i, j, dim)].blue;

}

}

}

The above code scans the rows of the source image matrix, copying to the columns of the destination image matrix. Your task is to rewrite this code to make it run as fast as possible using techniques like code motion, loop unrolling and blocking.

See the file kernels.c for this code.

<h2>Smooth</h2>

The smoothing function takes as input a source image src and returns the smoothed result in the destination image dst. Here is part of an implementation:

void naive_smooth(int dim, pixel *src, pixel *dst)

{

int i, j, ii, jj; pixel_sum ps;

for (j = 0; j &lt; dim; j++){ for (i = 0; i &lt; dim; i++){

initialize_pixel_sum(&amp;ps); for(ii = max(i-1, 0); ii &lt;= min(i+1, dim-1); ii++){ for(jj = max(j-1, 0); jj &lt;= min(j+1, dim-1); jj++){ accumulate_sum(&amp;ps, src[RIDX(ii,jj,dim)]); } } dst[RIDX(i,j,dim)].red = ps.red/ps.num; dst[RIDX(i,j,dim)].green = ps.green/ps.num; dst[RIDX(i,j,dim)].blue = ps.blue/ps.num; }

}

}

The functions max, min, initializepixelsum, and accumulatesum are all functions you can and probably will want to modify.

This code (and the helper functions) are all in the file kernels.c.

<h2>Performance measures</h2>

Our main performance measure is <em>CPE </em>or <em>Cycles per Element</em>. If a function takes <em>C </em>cycles to run for an image of size <em>N </em>× <em>N</em>, the CPE value is <em>C/N</em><sup>2</sup>. Table 1 summarizes the performance of the naive implementations shown above and compares it against an optimized implementation. Performance is shown for 5 different values of <em>N</em>. All measurements were made on a perf server.

The ratios (speedups) of the optimized implementation over the naive one will constitute a <em>score </em>of your implementation. To summarize the overall effect over different values of <em>N</em>, we will compute the <em>geometric mean </em>of the results

<table width="536">

 <tbody>

  <tr>

   <td width="200">Test case</td>

   <td colspan="3" width="82">1</td>

   <td colspan="3" width="49">2</td>

   <td colspan="3" width="49">3</td>

   <td colspan="3" width="49">4</td>

   <td width="15">5</td>

   <td width="93"> </td>

  </tr>

  <tr>

   <td width="200">Method                                  N</td>

   <td colspan="2" width="68">64</td>

   <td colspan="3" width="49">128</td>

   <td colspan="3" width="45">256</td>

   <td colspan="3" width="45">512</td>

   <td colspan="2" width="37">1024</td>

   <td width="93">Geom. Mean</td>

  </tr>

  <tr>

   <td width="200">Naive rotate (CPE)</td>

   <td colspan="2" width="68">5.8</td>

   <td colspan="3" width="49">6.6</td>

   <td colspan="3" width="45">9.8</td>

   <td colspan="3" width="45">10.5</td>

   <td colspan="2" width="37">29.5</td>

   <td width="93"> </td>

  </tr>

  <tr>

   <td width="200">Optimized rotate (CPE)</td>

   <td colspan="2" width="68">4.2</td>

   <td colspan="3" width="49">4.1</td>

   <td colspan="3" width="45">4.1</td>

   <td colspan="3" width="45">5.1</td>

   <td colspan="2" width="37">19.6</td>

   <td width="93"> </td>

  </tr>

  <tr>

   <td width="200">Speedup (naive/opt)</td>

   <td colspan="2" width="68">1.4</td>

   <td colspan="3" width="49">1.6</td>

   <td colspan="3" width="45">2.4</td>

   <td colspan="3" width="45">2.1</td>

   <td colspan="2" width="37">1.5</td>

   <td width="93">1.8</td>

  </tr>

  <tr>

   <td width="200">Method                                  N</td>

   <td width="57">64</td>

   <td colspan="3" width="49">128</td>

   <td colspan="3" width="49">256</td>

   <td colspan="3" width="49">512</td>

   <td colspan="3" width="41">1024</td>

   <td width="93">Geom. Mean</td>

  </tr>

  <tr>

   <td width="200">Naive smooth (CPE)</td>

   <td width="57">224.8</td>

   <td colspan="3" width="49">237.9</td>

   <td colspan="3" width="49">240.7</td>

   <td colspan="3" width="49">250.8</td>

   <td colspan="3" width="41">364.1</td>

   <td width="93"> </td>

  </tr>

  <tr>

   <td width="200">Optimized smooth (CPE)</td>

   <td width="57">37.8</td>

   <td colspan="3" width="49">38.2</td>

   <td colspan="3" width="49">38.2</td>

   <td colspan="3" width="49">38.9</td>

   <td colspan="3" width="41">41.4</td>

   <td width="93"> </td>

  </tr>

  <tr>

   <td width="200">Speedup (naive/opt)</td>

   <td width="57">5.9</td>

   <td colspan="3" width="49">6.2</td>

   <td colspan="3" width="49">6.3</td>

   <td colspan="3" width="49">6.5</td>

   <td colspan="3" width="41">8.8</td>

   <td width="93">6.7</td>

  </tr>

  <tr>

   <td width="200"></td>

   <td width="57"></td>

   <td width="11"></td>

   <td width="15"></td>

   <td width="23"></td>

   <td width="11"></td>

   <td width="15"></td>

   <td width="23"></td>

   <td width="7"></td>

   <td width="18"></td>

   <td width="23"></td>

   <td width="4"></td>

   <td width="22"></td>

   <td width="15"></td>

   <td width="93"></td>

  </tr>

 </tbody>

</table>

Table 1: CPEs and Ratios for Optimized vs. Naive Implementations

for these 5 values. That is, if the measured speedups for <em>N </em>= {64<em>,</em>128<em>,</em>256<em>,</em>512<em>,</em>1024} are <em>R</em>64, <em>R</em>128, <em>R</em>256, <em>R</em>512, and <em>R</em>1024, then we compute the overall performance as

<em>R </em>= p<sup>5 </sup><em>R</em>64× <em>R</em>128× <em>R</em>256× <em>R</em>512× <em>R</em>1024

Assumptions

To make life easier, you can assume that <em>N </em>is a multiple of 32. Your code must run correctly for all such values of <em>N</em>.

<h1>4          Infrastructure</h1>

We have provided support code to help you test the correctness of your implementations and measure their performance. This section describes how to use this infrastructure. The exact details of each part of the assignment is described in the following section.

Note: The only source file you will be modifying is kernels.c.

<h2>Versioning</h2>

You will find yourself writing many versions of the rotate and smooth routines. To help you compare the performance of all the different versions you’ve written, we provide a way of “registering” functions.

For example, the file kernels.c that we have provided you contains the following function:

void register_rotate_functions() { add_rotate_function(&amp;rotate, rotate_descr);

}

This function contains one or more calls to addrotatefunction. In the above example,

addrotatefunction registers the function rotate along with a string rotatedescr which is an ASCII description of what the function does. See the file kernels.c to see how to create the string descriptions. This string can be at most 256 characters long.

A similar function for your smooth kernels is provided in the file kernels.c.

<h2>Driver</h2>

The source code you will write will be linked with object code that we supply into a driver binary. To create this binary, you will need to execute the command

make driver

You will need to re-make driver each time you change the code in kernels.c. To test your implementations, you can then run the command: unix&gt; ./driver

The driver can be run in four different modes:

<ul>

 <li><em>Default mode</em>, in which all versions of your implementation are run.</li>

 <li><em>Autograder mode</em>, in which only the rotate() and smooth() functions are run. This is the mode we will run in when we use the driver to grade your handin.</li>

 <li><em>File mode</em>, in which only versions that are mentioned in an input file are run.</li>

 <li><em>Dump mode</em>, in which a one-line description of each version is dumped to a text file. You can then edit this text file to keep only those versions that you’d like to test using the <em>file mode</em>. You can specify whether to quit after dumping the file or if your implementations are to be run.</li>

</ul>

If run without any arguments, driver will run all of your versions (<em>default mode</em>). Other modes and options can be specified by command-line arguments to driver, as listed below:

-g : Run only rotate() and smooth() functions (<em>autograder mode</em>).

-f &lt;funcfile&gt; : Execute only those versions specified in &lt;funcfile&gt; (<em>file mode</em>).

-d &lt;dumpfile&gt; : Dump the names of all versions to a dump file called &lt;dumpfile&gt;, <em>one line </em>to a version (<em>dump mode</em>).

-q : Quit after dumping version names to a dump file. To be used in tandem with -d. For example, to quit immediately after printing the dump file, type ./driver -qd dumpfile.

-h : Print the command line usage.

<h2>Student Information</h2>

Important: Before you start, you should fill in the struct in kernels.c with your information (name and email address).

<h1>5          Assignment Details</h1>

<h2>Optimizing Rotate (20 points)</h2>

In this part, you will optimize rotate to achieve as low a CPE as possible. You should compile driver and then run it with the appropriate arguments to test your implementations.

For example, running driver with the supplied naive version (for rotate) might generate the output shown below: unix&gt; ./driver

Teamname: bovik

Member 1: Harry Q. Bovik

Email 1: <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="f4969b829d9fb49a9b839c918691da919081">[email protected]</a>

Rotate: Version = naive_rotate: Naive baseline implementation:

<table width="478">

 <tbody>

  <tr>

   <td width="191">Dim             64</td>

   <td width="64">128</td>

   <td width="64">256</td>

   <td width="64">512</td>

   <td width="64">1024</td>

   <td width="32">Mean</td>

  </tr>

  <tr>

   <td width="191">Your CPEs       5.8</td>

   <td width="64">6.5</td>

   <td width="64">10.0</td>

   <td width="64">10.4</td>

   <td width="64">29.7</td>

   <td width="32"> </td>

  </tr>

  <tr>

   <td width="191">Baseline CPEs 5.8</td>

   <td width="64">6.6</td>

   <td width="64">9.8</td>

   <td width="64">10.5</td>

   <td width="64">29.5</td>

   <td width="32"> </td>

  </tr>

  <tr>

   <td width="191">Speedup         1.0</td>

   <td width="64">1.0</td>

   <td width="64">1.0</td>

   <td width="64">1.0</td>

   <td width="64">1.0</td>

   <td width="32">1.0</td>

  </tr>

 </tbody>

</table>

<h2>Optimizing Smooth (20 points)</h2>

In this part, you will optimize smooth to achieve as low a CPE as possible.

For example, running driver with the supplied naive version (for smooth) might generate the output shown below:

unix&gt; ./driver

Smooth: Version = naive_smooth: Naive baseline implementation:

Dim             32      64      128     256     512     Mean

Your CPEs   226.0 238.1 240.3 252.4 364.1 Baseline CPEs 224.8 237.9 240.7 250.8 364.1

Speedup         1.0     1.0     1.0     1.0     1.0     1.0

Some advice. Focus on optimizing the inner-most loop (the code that gets repeatedly executed in a loop) using the optimization tricks covered in class. The smooth is more compute-intensive and less memory-sensitive than the rotate function, so the optimizations are of somewhat different flavors. Consider looking at the assembly code generated for the rotate and smooth, and/or running a profiler.

<h2>Coding Rules</h2>

You may write any code in kernels.c you want, as long as it satisfies the following:

<ul>

 <li>It must not interfere with the time measurement mechanism. You will also be penalized if your code prints any extraneous information.</li>

</ul>

You can only modify code in kernels.c. You are allowed to define macros, additional global variables, and other procedures in these files.

<h1>6          Evaluation</h1>

Your solutions for rotate and smooth will each count for 50% of your grade, or up to 20 code-execution points. In your interview, you will be asked to explain what you changed about your code, and why. You might be asked how some small additional change would effect performance. score for each will be based on the following:

<ul>

 <li>Correctness: You will get zero code-execution points for buggy code that causes the driver to complain! This includes code that correctly operates on the test sizes, but incorrectly on image matrices of other sizes. As mentioned earlier, you may assume that the image dimension is a multiple of 32.</li>

 <li>CPE: You will get full credit for your implementations of rotate and smooth if they are correct and achieve mean CPEs at or above thresholds <em>S</em><em>r </em>= 1<em>.</em>8 and <em>S</em><em>s </em>= 6<em>.</em>7 You will get partial credit for a correct implementation that does better than the supplied naive one.</li>

</ul>

More specifically, with <em>S</em><em>r </em>your averaged speedup for rotate and <em>S</em><em>s </em>your averaged speedup for smooth, the following equations are used to calculate your code-execution points (assuming you made at least some degree of improvement).

<ul>

 <li>rotate score: 5+((<em>S</em><em>r </em>−1<em>.</em>0)∗18<em>.</em>75) points</li>

 <li>smooth score: 5+((<em>S</em><em>r </em>−1<em>.</em>0)∗2<em>.</em>64) points</li>

</ul>

Other Note: Extra credit will require doing even better than <em>S</em><em>r </em>= 1<em>.</em>8 and <em>S</em><em>s </em>= 6<em>.</em>7, and may require esoteric or extreme modifications. For now, getting the full 10 extra credit points will require doing 10% better (ie, <em>S</em><em>r </em><em>&gt;</em>= 2<em>.</em>0 and <em>S</em><em>s </em><em>&gt;</em>=7<em>.</em>4).

<h2>Self-Assessment</h2>

Performance can vary widely from computer to computer, even if all of the machines are using the same virtualmachine image. Your final grade will be determined using one of the ”perf” machines listed below; the same one used to produce the baseline and optimize values listed above. We have a number of servers set up, which all perform <em>almost </em>the same:

<ul>

 <li>perf-01.cs.colorado.edu</li>

 <li>perf-02.cs.colorado.edu</li>

 <li>perf-03.cs.colorado.edu</li>

 <li>perf-04.cs.colorado.edu</li>

</ul>

To get your files on to the server:

scp -r ./perflab-handout &lt;identikey-name&gt;@perf-XX.cs.colorado.edu:˜

You may be prompted about a security key: type ‘yes’. You will be asked for your password: enter it.

To ssh in to a server:

ssh &lt;identikey-name&gt;@perf-XX.cs.colorado.edu

You may be prompted about a security key: type ‘yes’. You will be asked for your password: enter it.

Once your files are on the server and you are ssh’d in, you can use make to compile your code and the driver to test it as normal. Be sure to always recompile your code for the server (use the command make clean before make if necessary).

Note: You should work all the time on your local machine without connecting to the ”perf” servers, when you make sure that you have a performance improvement on your local machine you may connect to one of the servers to verify and test your results.

Note: If multiple students are connected to the same server at the same time running tests, it will effect performance. Thus, please log out of the servers when done. You can use the who command once ssh’d in to the server to see if anyone else is also using it. If so, you can check one of the other four machines.

Generally, we recommend you do most of your work just testing on your machine – as long as you don’t modify smoothnaive, the speedup between that and your fastest version will be a decent approximation of your final score.

At least once before your final submission, we recommend you ssh in to one of ther perf machines , and run make clean, then make, then ./driver -g. The resulting reported score should be very close to the score computed at the start of your grading interview.

<h1>7          Hand In Instructions</h1>

When you have completed the lab, you will upload one file, kernels.c, to Moodle.

<ul>

 <li>Make sure you have included your identifying information in the team struct in c.</li>

 <li>Make sure that the rotate() and smooth() functions correspondto your fastest implemnentations,as these are the only functions that will be tested when we use the driver to grade your assignement.</li>

 <li>Remove any extraneous print statements.</li>

</ul>

Good luck!