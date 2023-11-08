Hello there!

Welcome everyone! Lets get started!

Now first things first, my name is Patrick Pichler and I am currently employed at Dynatrace as a
Product Security Engineer.

Originally I started my career as a Java Software Developer, but everything changed when I stumbled
upon Linux and the cloud. This definitely transformed me into a full-on Linux nerd. I *used* arch
btw. Do not question why I am currently on a macbook though.

With this out of the way, let me ask you some quick questions. Who of you here has heared of
distroless before? Who of has heard about chainguard images?

Ok nice, for all of you who do not know neither of them, buckle your seatbelts as this will be one
heck of a ride.

Today we are going to containerize some small applications, with various technologies. Lets pray to
the demo gods, that everything will workout fine 🙏.

Introducing our demonstration application for today: vulnerable-awker.

This application provides a basic REST API designed to test AWK scripts against provided text.
However, there's a crucial issue: the original developer overlooked the fact that AWK has a system
call, enabling it to execute any local binary.

Admittedly, this example might seem somewhat contrived, but it serves well for our demonstration
purposes.

Lets dive right into containerizing the application then.

We start out with a simple Dockerfile approach. The application will be build as part of the image
building process. Since we have a Golang App, lets use the `golang` base image.

Now lets get to it. We first use a simple Dockerfile approach and use ubuntu as our base image.
We are going to build our application as part of the Dockerfile. This means that we need the golang
toolchain installed.

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
