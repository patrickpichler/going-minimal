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
give it a shot and build the image. Nice! The resulting image has `101MB`. Still not great, but way
better than `1.01GB`.

Another quick win would be to replace `ubuntu` with `alpine`. `alpine`, for all of you who do not
know it, is a minimal Linux Distribution. It is often used as base image for containers. One thing
to look out for though, is that it is based on `musl` instead of `glibc`. This could yield some
funky problems, so be sure to properly test your apps. Ok with `ubuntu` replaced lets build the
image. Now look at that. The resulting image, only has `15.3MB`. You heard that right `15MB`, vs the
`101MB` with `ubuntu`.

Believe it or not, we can even do better. While `alpine` is nice as a base image, `musl` might cause
issues. Additionally, `alpine` still comes with an `apk` binary. We do not need it for our service,
so lets get rid of it. There exist a special kind of base images, called distroless. They aim to
ship only with the strictly necessary files, like SSL certificates and libssl. The two most known
distroless image providers are Googles `distroless` project and Chainguard `chainguard-images`. Both
of them are pretty similar. While `distroless` is based on a stripped to bare Debian, Chainguard
images are wolfi based. `What on earth is wolfi?` I already hear you ask. In a nutshell, `wolfi` is
using the `apk` package manager, but instead of using upstream `alpine` packages, it comes with its
own set. The images provided by Chainguard are also build in a special way with a tool called
`apko`. It allows you to specify the content of your image in a declarative way. We later have a
closer look at it in action. Ok, so Chainguard offers a set of base images for quite a lot of
services out there. To name a few: argocd, nginx, nodejs, postgres, and so on. They also offer a
base image for busybox. Since busybox is also implementing AWK, we can replace the `alpine` image
we use with `cgr.dev/chainguard/busybox:latest`. You know the drill by now. Lets build it and have
a look at the resulting size. Alright, the image has `14.5MB` nice!

Cool! We brought down the image size from a whopping `1.01GB` to `14.5MB`. I would say image size is
somewhat important, but the gains we got from `alpine` are not even worth mentioning in my opinion.
As already hinted in the switch to Chainguard, there are still unrelated binaries and packages
installed in the base images. For example in `alpine` there is `apk`, as well as `busybox`, in the
busybox Chainguard image there still is `busybox`, even though we only need `awk`. Don't get me
started on the `ubuntu` image. There are all sorts of things installed, such as `perl`, `pgrep`, you
get the gist. Every installed package in our container images, will increase the attack surface of
our container at runtime. Of course a critical vulnerability in for example VIM alone will not make
your container vulnerable, as it isn't directly exposed to the outside. The issue lies more with
offering an potential attacker, who successfully exploited our service running to for example
execute code, another link in his attack chain. Pretty much every software has bugs. Even powerful
components such as `sudo` or any distributions package manager. Some of those vulnerabilities allow
an attacker to gain higher privileges. This is where defense in depth and zero trust comes into
play. By removing as much of the attack surface as possible, we make it harder for potential
attackers to further traverse through our infrastructure. But how can we build such a minimal
image, only containing the dependencies we need four our application. There are multiple ways. One
of the simplest would be from simply starting from a `scratch` image. The `scratch` image is pretty
much an empty image. This might cause some issues. For example our application might require system
libraries to be in place (like libc), or to have a `/etc/passwd` file in place. For such use cases,
the Chainguard static image might be for you. But what if we need some additional software installed?
Like in our example of the vulnerable-awk-playground. Here we have a hard dependency on AWK. Since
distroless containers do not ship a package manager, we will have a hard time adding additional
packages. Of course we could just copy them in place in the image by hand, but this is most often
harder done then said.

Meet the solution to this problem `apko`. As said before, `apko` is used quite heavily in Chainguard
images. The nice folks over at Chainguard even made it open source and usable for by pretty much
everyone. As word of warning, there is some effort required in switching your application to
build with `apko`. As said before and as the name implies, `apko` uses `APK` package manger index
files under the hood to install packages. To be able to install our `vulnerable-awk-playground`
application then in an `apko` build image, we need to somehow build an `APK` package out of it. Our
friends over at Chainguard also got us covered here. They offer a second open source tool, called
`melange`. With it, it is pretty easy to create `APK` packages from source files. As `apko` it
follows an declarative approach, with a bit of imperative pipelining sprinkled in, when needed.

Enough talk, lets get this APK party started.

I already have `apko` and `melange` locally installed. We start out by creating th



----

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
