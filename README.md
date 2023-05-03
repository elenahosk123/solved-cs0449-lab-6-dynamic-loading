Download Link: https://assignmentchef.com/product/solved-cs0449-lab-6-dynamic-loading
<br>
In this lab, you’ll be writing a <strong>simple file compression/decompression utility.</strong> No, you’re not writing a compression algorithm!! Instead, you’ll dynamically load <a href="https://zlib.net/">zlib</a> to do the compression and decompression for you.

This lab is a little more hands-off. I’m giving you a goal and some tools, and I wanna see how well you can put them together into a functioning program.

<a href="https://zlib.net/manual.html#Utility">Refer to the official zlib documentation for the three functions you’ll be using.</a>

<h2>Getting started</h2>

First get a little test file from me:

$ cp /afs/pitt.edu/home/j/f/jfb42/public/html/img1.bmp .

Then, here’s how your program should work:

$ ./lab6 -c img1.bmp &gt; compressedimg1$ ./lab6 -d compressedimg1 &gt; img2.bmp

You can use ls -l to see the size of the files in bytes. The original and final image files should be 1179702 bytes. The compressed file should be 995320 bytes.

The first command compresses img1.bmp into the compressedimg1 file. The second decompresses that file into img2.bmp.

After those two commands, img1.bmp and img2.bmp should be identical – in contents and length.

<h2>Your program</h2>

Here is a description of how your program will work. <strong>Remember to start writing your code from the top down.</strong> Stub out some functions for doing these things and call them from main.

<ul>

 <li>if argc &lt; 3, complain and exit.</li>

 <li><a href="#how-to-do-dynamic-loading-on-unix">load the zlib library.</a></li>

 <li><a href="#loading-zlib-and-the-needed-functions">extract the 3 functions you need.</a></li>

 <li>open file argv[2] for <strong>binary reading</strong>.</li>

 <li>if argv[1] is “-c”

  <ul>

   <li><a href="#how-big-is-a-file">read the entire file</a> into an <strong>input buffer</strong> that you malloc</li>

   <li>malloc a <strong>output buffer</strong> using compressBound to figure out what size it should be</li>

   <li><a href="#using-the-zlib-compress-and-uncompress-">use compress() to compress the input buffer into the output buffer.</a></li>

   <li>fwrite three things to stdout:

    <ul>

     <li>the <strong>uncompressed size</strong> (as an unsigned long)</li>

     <li>the <strong>actual compressed size</strong> (as an unsigned long)</li>

     <li>the <strong>output buffer</strong></li>

    </ul></li>

  </ul></li>

 <li>else if argv[1] is “-d”

  <ul>

   <li>fread two things:

    <ul>

     <li>the <strong>uncompressed size</strong></li>

     <li>the <strong>compressed size</strong></li>

    </ul></li>

   <li>malloc an <strong>input buffer</strong> big enough to hold the compressed data</li>

   <li>fread the rest of the data into that buffer (using the compressed size)</li>

   <li>malloc an <strong>output buffer</strong> big enough to hold the uncompressed data (using the uncompressed size)</li>

   <li><a href="#using-the-zlib-compress-and-uncompress-">use uncompress() to decompress the input buffer into the output buffer</a></li>

   <li>fwrite the output buffer to stdout</li>

  </ul></li>

 <li>else, complain and exit.</li>

</ul>

Your program should be fairly robust. It should <strong>give an error message and then exit</strong> in the following situations:

<ul>

 <li>too few program arguments</li>

 <li>invalid argv[1] (neither “-c” nor “-d”)</li>

 <li>couldn’t open the input file</li>

 <li>couldn’t open libz.so</li>

 <li>couldn’t get one or more of the symbols from zlib</li>

 <li>compress or uncompress failed (returned a negative number)</li>

</ul>

<h2>How to do dynamic loading on UNIX</h2>

This is an example. Don’t copy and paste the code into your program. Come on.

#include &lt;dlfcn.h&gt; in your program.

When you compile, give gcc the -ldl (that’s lowercase LDL) flag, like gcc -o lab6 -ldl abc123_lab6.c.

To dynamically load a library:

void* lib = dlopen(library_file_name, RTLD_NOW); if(lib == NULL){    // couldn’t load the library!    // give an error and exit.}

Then, to extract symbols from it, use dlsym:

void (*brand_new_function)() = dlsym(lib, “brand_new_function”); if(brand_new_function == NULL){    // couldn’t load the symbol!    // give an error and exit.}

Be sure to check the return values of dlopen/dlsym as shown above. Otherwise you’ll start getting segfaults and not know why.

<h2>Loading zlib and the needed functions</h2>

On thoth, zlib is already installed. It’s named “libz.so”, so use that as the first argument to dlopen.

You can make these global variables in your program. This <strong>is</strong> actually a legitimate use for globals!

The three functions you need to extract are the following:

unsigned long (*compressBound)(unsigned long length);int (*compress)(void *dest, unsigned long* destLen,               const void* source, unsigned long sourceLen);int (*uncompress)(void *dest, unsigned long* destLen,               const void* source, unsigned long sourceLen);

For example, to load compressBound,

compressBound = dlsym(lib, “compressBound”); if(compressBound == NULL){    // uh oh…}

<h2>How big is a file?</h2>

If you’ve opened a file, and you want to see how many bytes it is, it’s simple:

<ul>

 <li>fseek to the end of the file</li>

 <li>use ftell to get the current position into an unsigned long variable

  <ul>

   <li><strong>this is the size!</strong></li>

  </ul></li>

 <li>fseek back to the beginning of the file</li>

</ul>

<h2>Using the zlib compress and uncompress functions</h2>

Both functions have the same sort of prototype. Let’s look at compress for now:

int (*compress)(void *dest, unsigned long* destLen, const void* source, unsigned long sourceLen);

<ul>

 <li>dest is the destination buffer, where the compressed data will go.</li>

 <li>destLen is the length of the destination buffer, but notice, <strong>it’s a pointer.</strong>

  <ul>

   <li>when you call compress, give it the address of the length of your buffer.</li>

   <li>that will tell compress how big the destination buffer is.</li>

   <li>then, compress will <strong>change the value of your buffer length variable.</strong>

    <ul>

     <li>why does it do this?</li>

     <li>cause it doesn’t know exactly how big the compressed data will be!</li>

    </ul></li>

   <li>so after compress returns, your variable now contains the “correct” compressed size.

    <ul>

     <li>you can now write it out.</li>

    </ul></li>

  </ul></li>

 <li>source is the uncompressed buffer.</li>

 <li>sourceLen is the size of the uncompressed buffer.</li>

</ul>

uncompress works virtually identically, except swap the words “compressed” and “uncompressed.” &#x1f61b;

<h2>Using fread/fwrite with single variables</h2>

You can think of a single variable as an array of length 1. So…

unsigned long myvar = …;fwrite(&amp;myvar, sizeof(myvar), 1, myfile);