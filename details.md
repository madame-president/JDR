In July 2025, I got the following request:

1. I am running a full knots node, and I am parsing all the .dat files to build an internal API server with multiple endpoints customized to tradfi.

2. When parsing the .dat files, it appears that the magic bytes are NOT the standard (f9beb4d9)

3. We asked chatGPT, and it is more confused than us because it is claiming we are NOT running the proper node or not getting the blocks from the mainnnet

4. We checked and this is NOT the case. We are running the correct mainnet and software. What the hell is happening?

Solution: It turns out, the actual Bitcoin Software had a patch a few years ago were a XOR mask is applied, causing the magic bytes to be different.

I provided the correct script that would parse the .dat files ACCOUNTING for the XOR mask.

https://mempool.space/tx/0102471faa07de7be0b16686a601bb31b260628bd9369306c358f8dd3b712788


