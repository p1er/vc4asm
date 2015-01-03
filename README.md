# VC4ASM - macro assembler for Broadcom VideoCore IV

      aka Raspberry Pi GPU

The goal of the vc4asm project is a **full featured
        macro assembler** and disassembler with **constraint checking**. A
      simulator for debugging purposes is planned.

      The work is based on [qpu-asm
        from Pete Warden](https://github.com/jetpacapp/qpu-asm) which itself is based on [Eman's
        work](https://github.com/elorimer/rpi-playground/tree/master/QPU/assembler) and some ideas also taken from [<span
          class="vcard-fullname" itemprop="name">Herman H Hermitage</span>](https://github.com/hermanhermitage/videocoreiv-qpu).
      But it goes further by far. First of all it supports macros and functions.

      Unfortunately this area is highly undocumented in the public domain. And
      so the work uses only the code from [hello_fft](https://github.com/raspberrypi/userland/tree/master/host_applications/linux/apps/hello_pi/hello_fft)
      which is available as source and binary as Rosetta Stone.

[&rarr; Assembler](#vc4asm), [&rarr;
        Disassembler](#vc4dis), [&rarr;Build instructions](#build), [&rarr;Samples](#sample)
      [&rarr; Contact](#contact)

## Download &amp; history

Download [Source code, Raspberry Pi binary,
        examples and this documentation](../vc4asm.tar.bz2) (100k)

    <dl>
      <dt>V0.1</dt>
      <dd>First public release.</dd>
      <dt>V0.1.1</dt>
      <dd>Added support for binary constants between the code.</dd>
    </dl>

## <a id="vc4asm" name="vc4asm"></a>Assembler <tt>vc4asm</tt>

The heart of the software. It assembles QPU code to binary or C
      constants.

    <pre>vc4asm [-o &lt;bin-output&gt;] [-c &lt;c-output&gt;] [-E &lt;preprocessed&gt;] &lt;qasm-file&gt; [&lt;qasm-file2&gt; ...]</pre>

### Options

    <dl>
      <dt><tt>-o &lt;bin-output&gt; </tt></dt>
      <dd>File name for binary output. If omitted no binary output is generated.

        Note that <tt>vc4asm</tt> always writes _little endian_
        binaries.</dd>
      <dt><tt>-c &lt;C-output&gt;</tt></dt>
      <dd>File name for C/C++ output. The result does not include surrounding
        braces. So write it to a separate file and include it from C as follows:

        <tt>static const uint32_t qpu_code[] = {

          #include&lt;C-output&gt;

          };</tt></dd>
      <dt><tt>-C</tt><tt> &lt;C-output&gt;</tt></dt>
      <dd>Same as <tt>-c</tt>, but suppress trailing '<tt>,</tt>'.</dd>
      <dt><tt>-V</tt></dt>
      <dd>Check for Videocore IV constraints, e.g. reading a register file
        address immediately after writing it.</dd>
      <dt><tt>-E &lt;preprocessed-output&gt;</tt></dt>
      <dd>This is experimental and intended for debugging purposes only.</dd>
    </dl>

### File arguments

You can pass _multiple files_ to <tt>vc4asm</tt> but this will not
      create separate object modules. Instead the files are simply concatenated
      in order of appearance. You may use this feature to include platform
      specific definitions without the need to include them explicitly from
      every file. E.g.:

      <tt>vc4asm -o code.bin BCM2835.qinc gpu_fft_1k.qasm</tt>

### Assembler reference

1.  [Expressions and operators](expressions.html)
2.  [Assembler directives](directives.html)
3.  [Instructions](instructions.html)

See the [Broadcom
        VideoCore IV Reference Guide](http://www.broadcom.com/docs/support/videocore/VideoCoreIV-AG100-R.pdf) for the semantics of the instructions
      and registers.

## <a id="vc4dis" name="vc4dis"></a>Disassembler <tt>vc4dis</tt>

    <pre>vc4dis [-o &lt;qasm-output&gt;] [-x[&lt;input-format&gt;]] [-M] [-F] [-v] [-b &lt;base-addr&gt;] &lt;input-file&gt; [&lt;input-file2&gt; ...]</pre>

### Options

    <dl>
      <dt><tt>-o &lt;qasm-output&gt; </tt></dt>
      <dd>Assembler output file, <tt>stdout</tt> by default.

        Note that <tt>vc4asm</tt> always writes _little endian_
        binaries.</dd>
      <dt><tt>-x&lt;input-format&gt;</tt></dt>
      <dd><tt>32</tt> - 32 bit hexadecimal input, .e. 2 qwords per instruction,
        default if <tt>&lt;input-format&gt;</tt> is missing.

        <tt>64</tt> - 64 bit hexadecimal input.

        <tt>0&nbsp;</tt> - binary input, _little endian_, default
        without <tt>-x</tt>.</dd>
      <dt><tt>-M</tt></dt>
      <dd>Do not generate <tt>mov</tt> instructions. <tt>mov</tt> is no native
        QPU instruction, it is emulated by trivial operators like <tt>or r1,
          r0, r0</tt>. Without this option <tt>vc4dis</tt> generated <tt>mov</tt>
        instead of the real instruction if such a situation is detected.</dd>
      <dt><tt>-F</tt></dt>
      <dd>Do not write floating point constants. Without this option <tt>vc4dis</tt>
        writes immediate values that are likely to be a floating point number as
        float. This may not always hit the nail on the head.</dd>
      <dt><tt>-v</tt></dt>
      <dd>Write QPU instruction set bit fields as comment right to each
        instruction. This is mainly for debugging purposes.</dd>
      <dt><tt>-V</tt></dt>
      <dd>Check for Videocore IV constraints, e.g. reading a register file
        address immediately after writing it.</dd>
      <dt><tt>-b &lt;base-addr&gt;</tt></dt>
      <dd>Base address. This is the physical memory address of the first
        instruction code passed to <tt>vc4dis</tt>. This is only significant
        for absolute branch instructions.</dd>
    </dl>

### File arguments

If you pass multiple input files they are disassembled all together into
      a single result as if they were concatenated.

      The format of the input is controlled by the <tt>-x</tt> option. All
      input files must use the same format.

## <a id="build" name="build"></a>Build instructions

The source code has hopefully no major platform dependencies, i.e. you
      don't need to build it on the Raspberry. But it requires a [C++11](https://en.wikipedia.org/wiki/C++11)
      compliant compiler to build. So Raspbian Wheezy is not sufficient. In fact
      I only tested with gcc 4.8/4.9.

*   Download and extract the source.
*   Go to folder <tt>src</tt>.
*   If not Linux have a look at the first few lines of <tt>Makefile</tt>.
*   Execute <tt>make</tt>.
*   Now <tt>vc4asm</tt> and <tt>vc4dis</tt> executables should build.

## <a id="sample" name="sample"></a>Sample programs

All sample programs require _root access_ to run. This is because
      of the need to call <tt>mmap</tt>.

Furthermore you need to create a local character device named <tt>char_dev</tt>
      to access the vcio driver of the Raspi kernel: <tt>sudo mknod char_dev c
        100 0</tt>

      The latter won't work if you are at an nfs share or something like that.
      In this case create the device in the root file system e.g. <tt>/dev/vcio</tt>
      and place a symbolic link to the device with <tt>ln -s /dev/vcio char_dev</tt>.

All these restrictions apply to the original <tt>hello_fft</tt> as well.

### simple

This is a very simple program that demonstrates the use of all available
      operators with small immediate values. It is not optimized in any way.

### hello_fft

This is the well known hello_fft sample available. The only difference is
      that it is faster because the shader code has been optimized. The gain is
      about 10% of code size and about 3% of the run time.

## <a id="contact" name="contact"></a>Contact

Comments, ideas, bugs, improvements to _raspi at maazl dot de_.
