# knitr-for-asthma-crisprcas9-gene
knitr for asthma  crisprcas9 gene
LDPC (Low Density Parity Check) Library

LDPC codes are a type of linear forward error correcting code. They are high performance and in some cases approach the Shannon Limit. A few protocols/standards that use LDPC:

    IEEE 802.3an (10GBASE-T)
    802.16E (Mobile WIMAX)
    DVB-C2 (Digital Video Broadcasting - Cable)
    G.hn (ITU-T Standard for networking over power lines)

This repository contains an LDPC encoder and decoder implemented in Python and C++. The objective is to create a fast, high performance, and easy to use LDPC encoder/decoder.

The LDPC codes used in this project are from the IEEE 802.16E standard (Mobile WIMAX). These matrices are used because they are a special form of LDPC codes known as Quasi-Cyclic (QC) LDPC codes. QC-LDPC codes are faster to encode and decode. All of the LDPC matrices and coderates from the 802.16e-2012 standard are available for use.
Requirements

    CMake
    Compiler with OpenMP support (standard with GCC v4.9 and later. For macOS, see instructions below.)
    VOLK
    MathGL- v2.3.3, used to plot BER curves
        to install on Ubuntu 16.04: sudo apt-get install mathgl libmgl-dev

Encoder

The encoder is based on the algorithm found in [1]. It takes advantage of the dual-diagonal structure found in the LDPC codes used by WiMax. With minor modifications, this encoder will also work the the LDPC codes found in 802.11n.

The encoder currently supports the 1/2, 2/3A, 2/3B, 3/4A, and 5/6 code rates. The base matrix for the 3/4B matrix is structured a little differently than the others and is not implemented. The encoder supports all of the block sizes available in the 802.16e standard.

Usage: ./test_encoder <rate> <z> <num_rounds> <errors_to_add> <unencoded_data_file> <encoded_data_file>
Argument Description:
rate: LDPC code rate - half-rate        = 0
                       two-thirds-A     = 1
                       two-thirds-B     = 2
                       three-quarters-A = 3
                       three-quarters-B = 4
                       five-sixths      = 5
z: Z Factor (please refer to section 8.4.9.2.5 of the 802.16-2012 standard for more information)
num_rounds: How many LDPC encoding rounds to run.
errors_to_add: How many errors to add to each LDPC codeword.
unencoded_data_file: File to write the unencoded data to.
encoded_data_file: File to write LDPC encoded data to.

Decoder

The decoder uses the Turbo-Decoding Message-Passing Algorithm [2]. The decoder currently supports the 1/2, 2/3A, 2/3B, 3/4A, 3/4B, and 5/6 code rates. The decoder supports all of the block sizes available in the 802.16e standard.

Usage: ./test_decoder <num_threads> <rate> <z> <num_codewords> <encoded_data_file> <decoded_data_file>
Argument Description:
num_threads: Number of threads to use.  num_codewords modulo num_threads == 0.
rate: LDPC code rate - half-rate        = 0
                       two-thirds-A     = 1
                       two-thirds-B     = 2
                       three-quarters-A = 3
                       three-quarters-B = 4
                       five-sixths      = 5
z: Z Factor (please refer to section 8.4.9.2.5 of the 802.16-2012 standard for more information)
num_codewords: How many LDPC codewords are in the file.
encoded_data_file: File to read the encoded data from.
decoded_data_file: File to write decoded data to.

The throughput of the decoder is increased using openmp threading. Parallelization is performed at the LDPC frame level making this ideal for bursts that contain multiple LDPC frames. This approach eliminates the concurrency issues found when parallelization is performed at the Check Node level. If this decoder is used on streaming LDPC frames, there will be latency added by the aggregation of frames.
BER_runner

The BER runner creates an encoder/decoder pair and returns the Bit Error Rate (BER) curve for the user specified code rate and size. The resulting plot is a saved as a PNG file in to the directory where the tool is running.

Usage: ./BER_example <num_threads> <rate> <z> <EbNo_start> <EbNo_end> <EbNo_step> 
Argument Description:
num_threads: Number of threads to use.  num_codewords modulo num_threads == 0.
rate: LDPC code rate - half-rate        = 0
                       two-thirds-A     = 1
                       two-thirds-B     = 2
                       three-quarters-A = 3
                       three-quarters-B = 4
                       five-sixths      = 5
z: Z Factor (please refer to section 8.4.9.2.5 of the 802.16-2012 standard for more information)
EbNo_start: Starting value for BER plot x axis (dB)
EbNo_end: Ending value for BER plot x axis (dB)
EbNo_step: Step value for BER plot x axis (dB)

An example plot is show below:

alt text
Capacity Curve

The Capacity Curve example creates a BPSK encoder/decoder pair and calculates the Eb/No value where the Bit Error Rate (BER) is 1e-5 for all code rates. The user specifies the Z factor. The resulting plot is a saved as a PNG file in to the directory where the tool is running. The plot also shows the Shannon limit and BPSK-AWGN bound for reference purposes.

Usage: ./Capacity_example <num_threads> <z>
Argument Description:
num_threads: Number of threads to use.
z: Z Factor (please refer to section 8.4.9.2.5 of the 802.16-2012 standard for more information)

An example plot is show below:

alt text
Tools

To build the tools, perform the following steps:

    clone the repository: https://github.com/myersw12/wimax_ldpc_lib.git
    create the build directory: cd wimax_ldpc_lib && mkdir build && cd build
    run cmake: cmake ../
    run make: make -j4

To install the library after the build is complete, run the following:

    sudo make install
    sudo ldconfig

The executables listed below are found in the wimax_ldpc_lib/build/lib/ directory.
test_encoder

Example Usage: ./lib/test_encoder 0 96 500 100 data.bin encoded.bin

Stats from running on a i7-4771 @ 3.50GHz (running the example above, 4 threads):

    Average Rate (Mbits/Sec): 171.904706
    Fastest Iteration (Mbits/Sec): 252.659283
    Slowest Iteration (Mbits/Sec): 16.031953

test_decoder

Example Usage: ./lib/test_decoder 4 0 96 500 encoded.bin decoded.bin

Stats from running on a i7-4771 @ 3.50GHz (decoding the output from the above example, 4 threads):

    Average Rate (Mbits/Sec): 31.755333
    Fastest Iteration (Mbits/Sec): 35.466888
    Slowest Iteration (Mbits/Sec): 11.636364

macOS

The default clang compiler in macOS does not provide OpenMP support. Before building please follow the instructions below:

brew install llvm
export CC=/usr/local/opt/llvm/bin/clang
export CXX=/usr/local/opt/llvm/bin/clang++ 
export LDFLAGS="-L/usr/local/opt/llvm/lib"
export CPPFLAGS="-I/usr/local/opt/llvm/include"

Directory Structure

    /alist - alist files for all of the 802.16e code rates. Provided for reference and convenience.
    /cmake - cmake modules
    /include - header files
    /ipython_notebooks - Encoder/Decoder examples in python
    /lib - source code
    /python_ldpc - python implementation of encoder & decoder

References

1: Chia-Yu Lin, Chih-Chun Wei and Mong-Kai Ku, "Efficient encoding for dual-diagonal structured LDPC codes based on parity bit prediction and correction," APCCAS 2008 - 2008 IEEE Asia Pacific Conference on Circuits and Systems, Macao, 2008, pp. 1648-1651. doi: 10.1109/APCCAS.2008.4746353

2: B. Le Gal and C. Jego, "High-Throughput Multi-Core LDPC Decoders Based on x86 Processor," in IEEE Transactions on Parallel and Distributed Systems, vol. 27, no. 5, pp. 1373-1386, May 1 2016. doi: 10.1109/TPDS.2015.2435787
