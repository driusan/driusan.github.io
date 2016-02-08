---
layout: default
title: The Git Pack Format
---
If you recall at the end of my [last post](git-fetch.html) the result of making a request to $GITURL/git-upload-pack
to download a file (yes, you read that correctly) is a git pack file.

So what's the format of a pack file? It turns out it's a [well documented](https://github.com/git/git/blob/master/Documentation/technical/pack-format.txt),
but poorly specified format.

If you read the documentation, it starts like so:

<pre>
== pack-*.pack files have the following format:

   - A header appears at the beginning and consists of the following:

     4-byte signature:
         The signature is: {'P', 'A', 'C', 'K'}

     4-byte version number (network byte order):
	 Git currently accepts version number 2 or 3 but
         generates version 2 only.

     4-byte number of objects contained in the pack (network byte order)</pre>
     
 Okay, good, there's a signature to identify the file type (and ensure we have the right
 endianness), a version number of the file format (N.B. when they say "number" they mean
 unsigned), and a declaration of the number of objects in the file.
 
 <pre>
   - The header is followed by number of object entries, each of
     which looks like this:</pre>
     
 And then we get to the data, this seems easy by comparison so far.
 
 <pre>
     (undeltified representation)
     n-byte type and length (3-bit type, (n-1)\*7+4-bit length)
     compressed data
     
     (deltified representation)
     n-byte type and length (3-bit type, (n-1)\*7+4-bit length)
     20-byte base object name if OBJ_REF_DELTA or a negative relative
	 offset from the delta object's position in the pack if this
	 is an OBJ_OFS_DELTA object compressed delta data</pre>
	 
What?

I'll save you the googling to figure out what this means (luckily there's plenty of blog posts
and the like explaining it.)

Each entry starts with a variable length header. The most significant bit ( x & 0xF0) of each
byte tells you if this is the last byte in this header. Every time the bit is set, you need to mask
out the bit, and then read the next byte. The next byte represents the next 7 bits of the length
encoded in the header. So until you get to a byte where b & 0xF0 is 0, you need to keep doing 
the following:

1. Read a byte
2. Blit out the high bit.
3. Bitwise Or the remaining 7 bits onto address (number of bytes read so far * 7)

This is the general algorithm git pack files seem to use to represent sizes. I'm not sure why they
don't just use a uint64, because if you're storing more than an exabyte in git you're probably doing
something wrong, but I guess if you *need* to represent an unbounded number somehow this
is as good of a way as any.

So anyways:

The first byte is an exception, because bit-1 is the "continue" bit, 2-4 are a uint8 representing
the object type (defined later), and then the remaining 4 bits are the first 4 bits. So for the sake
of this entry header, add 4 bits to all your shifts. (And just read the first 4 bits by masking out 0x0F
on the first byte.)

This is followed by the (zlib) compressed data, which we should be able to just read out because
we have the length, right?

Nope!

The length in the header is the length of the *uncompressed* data, while the data stream is compressed.
So how do you know where the next entry starts? Unless you have the pack index file (which git-upload-pack
didn't send), you don't.

It turns out, zlib compression ignores arbitrary bytes at the end of a data stream. This means that, for the
first entry, you can just take the address immediately after the size header, and pass it to your zlib decompressor
to get the data.

This works well for the first entry, except most high level zlib libraries (ie. compress/zlib in Go) read more
bytes than they need and leave the io.Reader pointer past the end of the data. 

My first attempt to getting the address of the second entry was to take the the address of the first entry,
after decompressing it recompress it with the same algorithm/compression level, and then use that
to figure out the length of the compressed block. This didn't work. (The length was off by 3 bytes, and
the data was slightly different.) I don't know why this is [(my StackOverflow question doesn't have any
answers)](http://stackoverflow.com/questions/35258001/git-packfile-entry-offset-calculations), but
I guess zlib compression can't be used as a hash.

Attempt number two:

Each zlib chunk ends with a 4 byte digest/summary for validating that the data wasn't corrupted. I took the compress/zlib
library, from Go, modified it to expose the digest, and then backtrack through the file after reading looking for this signature.
This did, in fact, work and get me the proper offsets.

So now we've got:
1. An object type
2. A length of the object
3. An uncompressed data stream.

Git objects generally have the form "[type as a string] [length]\0[blob]". So since we have all that data, we can just put it
in a file, take a sha1 hash to get the identifier, zlib compress it, and put it in .git/objects/, right?

The valid object types are:

- OBJ_COMMIT  = 1
- OBJ_TREE = 2
- OBJ_BLOB = 3
- OBJ_TAG  = 4
- (5 is reserved for future use)
- OBJ_OFS_DELTA = 6
- OBJ_REF_DELTA = 7
	
This approach works for commit, tree, blob, and tag. But packs are almost certain to have, at least OBJ_REF_DELTA. (OBJ_OFS_DELTA
should only be present if you negotiated for it when calling git-upload-pack.)

If you go back to the spec, you'll notice that there were headers for "deltified", and "non-deltified". The commit/tree/blob/tag
are the non-deltified ones, the REF_DELTA and OFS_DELTA are deltified. This means that instead of storing the object, the data
stores the changes from another reference. How do you interpret those changes?

It turns out, this isn't documented anywhere obvious. Googling will turn up things like StackOverflow threads where people ask
and get responses like "it used to be this algorithm, then it changed to this one, and now it's a modified version of that."

The best reference I've been able to find is [this one](http://stefan.saasen.me/articles/git-clone-in-haskell-from-the-bottom-up/#format_of_the_delta_representation)
from someone trying to do git clone in Haskell, but I haven't finished implementing it yet to see if it works. All I can say for sure is that
it involves a lot of similar bit-wise tricks to get it to work.
