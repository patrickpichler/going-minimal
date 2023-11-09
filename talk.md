Hello there!

Welcome everyone! Lets get started!

Now first things first, my name is Patrick Pichler and I am currently employed at Dynatrace as a
Product Security Engineer.

Originally I started my career as a Java Software Developer, but everything changed when I stumbled
upon Linux and the cloud. This definitely transformed me into a full-on Linux nerd. I *used* arch
btw. Do not question why I am currently on a Macbook though.

With this out of the way, let me ask you some quick questions. Who of you here has heard of
Distroless before? Who of has heard about Chainguard images?

Ok nice, for all of you who do not know neither of them, buckle your seatbelt as this will be one
heck of a ride.

Today we are going to containerize some small applications, with various technologies. Lets pray to
the demo gods, that everything will workout fine 🙏.

Introducing our demonstration application for today: vulnerable-awk-playground.

This application provides a basic REST API designed to test AWK scripts against provided text.
However, there's a crucial issue: the original developer overlooked the fact that AWK has a system
call, enabling it to execute any local binary.

Admittedly, this example might seem somewhat contrived, but it serves well for our demonstration
purposes.

Lets dive right into containerizing the application then.

We start out with a simple Dockerfile approach. The application will be build as part of the image
building process. Since we have a Golang App, lets use the `golang` base image.

Now lets get to it. We first use a simple Dockerfile approach. The application is going to be built
as part of the Dockerfile build, which means, we need the Golang toolchain installed. To keep things
simple we are going to use `golang:1.21.3-bookworm` as our base image.

Lets have a look at the resulting Dockerfile. Nice, pretty straight forward. Then lets build the
image. Alright, didn't take too long. Lets guess, how big will the resulting image be? Any
suggestions? Whopping `1.01GB`. The base image attributes `822MB` to the resulting size. We for
sure can do better.

Alright, Docker has the concept of multi stage builds. This means, that instead of having the full
build toolchain in the resulting runtime image (we do not need it at all anyway), we have a second
image and just copy over the binary. This should drastically decrease the image size.

Lets give it a shot and modify our Dockerfile. First we are going to add an `as` label. Lets call
the first image `build`. Next we introduce a second `FROM`. We are going to use `ubuntu:23.10` as
the base for our runtime image. Nice, it hopefully will result in a smaller image size. All that is
left to do, is to copy over the resulting binary to the second image and voila, we are done. Lets
give it a shot and build the image. Nice! The resuling image has `101MB`. Still not great, but way
better than `1.01GB`.

You might ask yourself, why are big container images even a problem? Networking is fast, so why
should we care that our production image is 3GB in size?

One of the biggest reason to aim for minimal images, is the reduced attack surface. If malicious
attackers gain access to your running containers, any additionally not required binary could give
them an further link in their attack chain.

**TODO:** Come up with good example to showcase potential exploit

Additionally, the signal to noise ratio of CVE scanners will be improved by a lot. If you only
include libraries and binaries actually needed at runtime, the chance that a vulnerability in one
of those requires your attention is by far larger, than lets say a vulnerability in VIM. Even though
vim is the best text editor there is, it should not be part of all your images.

Another benefits of a minimal image, is the reduced pull time. Even though networks are incredible
fast nowadays, your container will pulled faster, if the image size is smaller.
