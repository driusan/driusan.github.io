---
layout: default
title: The Git Fetch Protocol
---
I've been trying to write a [pure Go](https://github.com/driusan/go-git/)
git client so that I can commit and upload a [Plan 9](plan9.html) bug fix
for my [distributed bug tracker](https://github.com/driusan/).

With the [git technical documentation](https://github.com/git/git/tree/master/Documentation/technical) I managed to interpret the index file
format well enough to add support for "git add", "git reset", and 
"git checkout".

Now I'm at the part where where I need to communicate with remote 
repositories for things like git clone, git fetch, or git push. 
Here is what I've figured out so far (today's date is Feb 6, 2016, if
you're from the future.):

Cloning a repository is a simple concept. There's a remote git directory.
You want to copy it.  Since I'm trying to do this in pure Go, it seemed 
like the HTTP protocol would be the easiest way to achieve this.

Conceptually, since HTTP is a stateless protocol that serves files, the
clone protocol could be implemented by just sending a request for each
file that you want and crawling the remote .git directory. In fact, this 
is how the first versions of git worked. It was called the "Dumb HTTP 
Protocol." Today, no git major servers (such as github) support this 
anymore.  Instead, they insist that you use the more efficient "Smart 
(Git) HTTP protocol", so that's what I've been trying to write a client 
to do.

The smart protocol is smart in that, instead of sending every single 
file, the git client makes a request to the server to ask it what branches
the server has and what commit they're pointing to, and then another 
request telling the server which branches it wants, what commits it has, 
and then the server responds with a file that has only the difference
that the client needs.

Again, conceptually this maps pretty well to an HTTP. You would just need 
to send a GET request to figure out what the server has, and then a
POST request of data in a format like:
<code><pre>want (sha1 hash)
want (another sha1 hash)
have (a different sha1 hash)
have (yet nother sha1 hash)</pre></code>

and get, say, a tarball as a response.

That's sort of what the "Smart" git protocol does, except instead of
making a RESTful design over HTTP, it tries to pretend HTTP is a 
full-duplex connection that it directly maps its SSH protocol to.

Every line starts with 4 bytes. The 4 bytes are an ASCII string which
represent a hex value that needs to be parsed to tell you how long the
line is. This is necessary because the git protocol "lines" are a weird
binary plaintext hybrid, where some lines have "\0" in the middle of the
line and git assigns special meaning to the parts before and after the 
"\0".  (At least, it's necessary to have a fixed size line length instead 
of just treating "\n" as a line delimiter. Why not just use a uint32 
instead of an ASCII string to interpret as hex? I don't know.)

So instead of "want (sha1 hash)\n" each line needs to be "0032want (sha1 hash\n") (If you do the math and say "but 'want (sha1 hash)' should have a 
length of 0x2e in hex!", you're right, but the line length calculation 
*includes* the 4 character length prefix.).

The first line is an exception, because instead of "0032want (sha1 hash)\n"
the format is: "....want (sha1 hash)\0(list of capabilities supported)\n" 
where .... is a value that you'll need to calculate because the capabilities
is a variable length.

(This is for both the first line of the client request, and the first line
of the server response.)

Why not include an X-Git-Capabilities HTTP header and keep a consistent
format for every line? Presumably, because they wanted to just reuse the
SSH code, and since SSH is a stateful, bi-directional, full-duplex 
connection, this kind of thing makes (some) sense there.

After you've computed all your line lengths for your request and send
your post request in this format, you'll get back.. nothing but an
empty "200 OK" response.

Obviously, this was a bad request, but the server responded with a
"200 OK" response instead of "400 Bad Request" because the git 
"smart HTTP" documentation says that the only valid responses 
are either "200 OK" or "304 Not Modified"

So why was the request bad? That's because the protocol requires a 
"flush line" of "0000" (that's four bytes of value 0x30, not \0) to 
tell the server that it should compute a delta to send, rather than the 
server assuming that once the POST request is received the client 
wants a response, since as far as git concerned HTTP should be treated
the same as the SSH protocol.

So you add 0000 to your request, and you'll still get nothing, because
instead of assuming that after the client has told the server to calculate
a delta it wants that delta, the server wants to explicitly be told that
client is done sending data, so you need to include "00000009done\n"
(that's 0000 to tell it that there's a 0 length line and it should 
proceed, followed by "0009done\n" telling it that there's a line containing
"done\n" indicating the client is done. Recall that the line length includes the 4 characters telling it how long the line is, so "done\n" becomes 
a 9 characters line.)

There's a capability called "no-done" that you can send as an optimization
to get rid of the "0009done\n" line, but that doesn't work unless the
multi\_ack\_detailed capability is also present. So, essentially, you can
add "multi\_ack multi\_ack\_detailed no-done" at the start of the request to 
avoid having to send "0009done\n" at the end, but I don't want to do that
because presumably supporting multi\_ack complicates the implementation.

Anyways, after you've figured this all out and sent a valid request, the
the server will respond with a request of Content-Type 
"application/x-git-upload-result" (actually, the empty responses had 
that too) rather than a Content-Type telling you the format of the
response.

So what is the response? First, there's one line with the same
4 byte length prefix and containing either "0007NAK" or
"0030ACK (sha1 of the first common object)" (technically, this is
the response to the 0000 line, and then after sending this response
it waited for the "0009done\n" line to send the actual data commit
data.)

The actual data, after you've gotten this far, is a git pack file that
has it's own idiosyncrasies, but I'll save the details of that for
another day. (Also, I haven't entirely figured it out yet.)
