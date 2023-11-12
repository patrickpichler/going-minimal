Hello there!

Welcome everyone! Let's get started!

Now first things first, my name is Patrick Pichler and I am currently employed at Dynatrace as a
Product Security Engineer.

Originally I started my career as a Java Software Developer, but everything changed when I stumbled
upon Linux and the cloud. This definitely transformed me into a full-on Linux nerd. I *used* arch
btw. Do not question why I am currently on a Macbook though.

With this out of the way, let me ask you some quick questions. Who of you here has heard of
Distroless before? Who of has heard about Chainguard images?

Ok nice, for all of you who do not know neither of them, buckle your seatbelt as this will be one
heck of a ride.

Today we are going to containerize a small application, with various technologies. Let's pray to the
demo gods, that everything will workout fine 🙏.

Introducing our demo app for today: vulnerable-awk-playground.

This application provides a basic REST API designed to test AWK scripts against provided text.
However, there's a crucial issue: the original developer overlooked the fact that AWK features
various way of executing binaries.

It requires an AWK binary to be present on the system to run.

Admittedly, this example might seem somewhat contrived, but it serves well for our demonstration
purposes.

Let's dive right into containerizing the application then.

Now lets get to it. We first use a simple Dockerfile approach. The application is going to be built
as part of the Dockerfile build, which means, we need the Golang toolchain installed. To keep things
simple we are going to use `golang:1.21.3-bookworm` as our base image.

Take a look at the resulting Dockerfile. It is pretty straight forward. Time to build the image.
Alright, didn't take too long. Now, care to geuss the size of the resulting image? Any suggestions?
A whopping `1.01GB`. The base image adds a hefty `822MB`. But fear not, we can do a lot better.

We can step up our game with a nifty Docker feature called multi stage builds. Instead of cramming
the complete go build toolchain in the resulting runtime image (not that we need it anyway), there
is a second image just for building. Then we just copy over the binary. This will decrease the image
size by quite a bit.

Let's give it a shot and tweak our Dockerfile. First up, toss an `as` label at the end of the first
`FROM` directive. We call the first image `build`. Next we introduce a second `FROM`. `ubuntu:23.10`
should do the job just fine. Nice, it hopefully will result in a smaller image size. All that is
left to do, is to add the command for copying over the `vulnerable-awk-playground` binary. Voila, we
are done. We can build the image now. Nice! The resulting image has `101MB`. Still not great, but
way better than `1.01GB`.

Another quick win would be to replace `ubuntu` with `alpine`. `alpine`, for all of you who do not
know it, is a minimal Linux Distribution. It is often used as a base image for containers. One thing
to look out for though, is that it is using the `musl` implementation of libc instead of `glibc`.
This could yield some funky problems, so be sure to properly test your apps. With `ubuntu` replaced
lets build the image. Now look at that. The resulting image, only has `15.3MB`. You heard that right
`15MB`, vs the `101MB` with `ubuntu`.

Believe it or not, we can even do better in regards to size. While `alpine` is already pretty minimal
in the amount of things it ships by default, it still contains the `apk` package manager binary.
Our application only depends on `awk`, so this is just dead weight.

There is a special kind of base images, called distroless. The two most known distroless image
providers are Googles `distroless` project and Chainguards `chainguard-images`. Both of them are
pretty similar. While `distroless` is based on a stripped to bare version of Debian, Chainguard
images are based on `wolfi`. `What on earth is wolfi?` I already hear you ask. `wolfi` is a
distribution maintained by Chainguard. The magic part of it is, that it leverages the `apk` package
manager, but instead of using upstream `alpine` packages, it has its own. Also worth noting is, that
compared to `alpine`, `wolfi` ships with `GLIBC` instead of `musl`.

Chainguard offers a pretty large set of container images. They range from tools such as ArgoCD, over
nginx, as well as specialized images packaging language toolchains such as Golang or NodeJS.

Since `vulnerable-awk-playground` has a hard dependency on `awk`, we are going to use the
`cgr.dev/chainguard/busybox:latest` base image. By now you should know the drill. Replace the base
image, build it and check out the image size. `14.5MB` nice! By this point the image size win are
insignificant over `alpine`. The `chainguard` based image is a whopping `69` times smaller than the
image we started out with! Pretty Nice!

Building containers with Dockerfiles is pretty easy. It sadly also has a lot of foot guns.

We now switch to a somewhat contrived example of a Dockerfile. We write some secret into some file
and delete it later. Just imaging instead of the secret being `super-secure`, it is some private ssh
key and we are using it to checkout our super secret magic sauce code from our Github repositories.
If we build the container image and run it, the `/etc/secret.txt` file no longer exists. If you
have worked a lot with containers you probably already know where this is heading, but if you are
somewhat new to the whole container world, this might be confusing. What if I tell you, the end user
of the container image still has access to the secret?

Before I explain you how, we need a bit of theory about the parts that make up a container image.
Nowadays pretty much all container related tooling produce container image according to the OCI
spec. OCI stands for Open Container Initiative. In a nutshell a container image is simply a tar ball
containing a `manifest.json` that describes things such as the entrypoint command of the image, as
well as the order of layers. All layers are then also stored as tar balls in the image. `Layers?`
you might ask yourself. Yeah that is right, a container images consist of layers. Each instruction
in a Dockerfile creates a new layer. In our example, there will be `4` layers created. In the end a
layer is once again a tar ball containing files and folders. When you then create a container from
that image, you can imagine the container runtime extracting all files from the tar balls one by
one in the order specified by the `manifest.json`. This is not exactly what is happening, but it is
good enough to for now.

All of this works pretty fine for adding new files in layers. Deleting a file is a bit more tricky.
Let's have a look at the OCI image spec. It says that file deletion are handled by special whiteout
files. They are empty files, with the same name as the file to delete, only to be prefixed by
`.wh.`. In our docker example this means, that the layer created by running the `rm` command will
contain a single empty file called in the `etc` folder called `.wh.secret.txt`.

We can see all of this in action, by simply exporting our container image by running `docker image
save insecure-layer-sample:latest -o image.tar`. Next we untar the file. The `manifest.json`. Tells
us, that we need to look at the xxxxx file, as it is the last layer in our image. Opening the
`xxxxx/layer.tar` file reveals the empty whiteout file under the `etc` folder.

Let's check out the second layer and who would have thought, it contains our secret.

While there certainly are ways of solving this issue in Dockerfiles, namely using secret mounts,
there also exist better solutions for building container images, in my opinion. If you for example
build a Java Service, I can highly recommend Jib. Jib can magically transform your Java service
into an container image, by simply adding it as a gradle/maven plugin to your build script. It even
goes as far as optimizing your image layers, to keep image build times low. Another very nice plus
point for Jib is, that it does not require a Docker daemon for building. This means it is trivial to
integrate into your CI pipelines.

If you are building a Golang service like we do, there is a similar tool called `ko`. You simply run
`ko build` and there you have an optimized container image for your application.

Both of those tools are pretty specialized tools for either Golang or Java. Both allow you to
specify the base images of the resulting containers. There also exists another handy tool. It is
called `apko`, is open source and was created by our friends over at Chainguard. All of the
Chainguard images are actually build via `apko`.

`apko` allows you specify in a declarative way what packages should be part of the resulting image.
As the name already implies, it leverages the same package format as `apk`. This means you can
easily build images with packages from both wolfi and alpine upstream repos.

The best part of `apko` then is, that it makes it trivial to build images only containing the
strictly necessary parts you need. Each installed package in an container image will increase the
attack surface of the container. Of course, a critical vulnerability in for example VIM can only
be exploited, if an attacker manages to executes commands in our container. For this we need to
have a RCE or command injection vulnerability. Each package then could act as the next link in
the attackers attack chain to take over our service.

Another very nice side effect of having only the absolutely necessary packages installed the reduce
CVE notification fatigue caused by image scanners. As mentioned before, do you really care that VIM
is vulnerable in your container image?

For example, let's compare the number of CVEs found in the various images we built today for the
`vulnerable-awk-playground`. We will use `trivy` for that job. All we need to run is `trivy image`
and pass it the image we want to scan. Here are the results (stand 12.11.2023):

| Image                                                 | Vulnerability Count |
| ----------------------------------------------------- | ------------------- |
| vulnerable-awk-playground:all-in-one.json             | 326                 |
| vulnerable-awk-playground:multi-stage-ubuntu.json     | 13                  |
| vulnerable-awk-playground:multi-stage-alpine.json     | 4                   |
| vulnerable-awk-playground:multi-stage-chainguard.json | 0                   |

We have a clear winner here. There is not a single vulnerability found in the chainguard version of
our containerized application. I guess this is all that is required to say here.

Back to `apko`. As bad luck would have it, `vulnerable-awk-playground` is susceptible to RCE and
command injection. Let's transform it to only contain the absolute minimum of images installed.

There is already the first problem. `apko` can only install `APK`packages. How can we turn
`vulnerable-awk-playground` into an `APK`?

Meet the second amazing open source tool created by Chainguard: `melange`. With `melange` building
`APK` images is trivial. All you need to do is to once again specify how to build your app in an
declarative way and you are good to go.

Enough talk, lets get this APK party started.

I already have `apko` and `melange` installed on my system. We start out by creating the APK
package. For this we create a `melange.yaml`, specify some metadata, such as the name, setup which
packages for our build environment and define a build pipeline. The pipeline itself is rather
straight forward as Golang is pretty easy to build. Nice! Let's create a package signing key by
running `melange keygen` and create the APK package by running the `melange build` command. I am
using the `docker` runner + some custom settings, since I am on MacOS. If I would run a proper OS
like Linux, I could even make use of bubblewrap, which would allow us to build the package without
requiring higher privileges.

With the build done, we now have the APK added to the `packages` folder.

Next up, lets build our image. For this we create an `apko.yaml` file. First we need to specify
the keyring that our build trusts when installing packages. We also need to declare a set of
repositories `apko` will get APK packages from, as well as the packages we want installed in our
image. We use the standard wolfi repo, as well as our local `packages` folder, which contains the
result of the `melange` build. Next up, we setup the accounts used. It is best practice to not
run your containers as root, hence we create a `nobody` user. Last, but not least, we specify the
entrypoint. Here we simply run `vulnerable-awk-playground`. This should be it! Buiding the image is
as simple as running `apko build`. This leaves us with a tar ball of the image. We can import it
via the `docker load` command. Let's check for the image size by running `docker image ls`. It has
`20.8MB`, not too bad. What about vulnerabilities? Trivy repots **0** vulnerabilities. This is
amazing. Also exploitation just has become harder. The container doesn't even have a shell
installed.

Having a minimal container image is not a silver bullet though. Did you know that you can create a
reverse shell via AWK as well? To further harden your service, you should apply a strategy called
defense in depth. In a nutshell you can think of it like a medival castle. Instead of having a
single wall around it, they had multiple layers of defenses. Be it a moat with a draw bridge,
or multiple rings of walls. Back in devops land, this pretty much translates to having security
not just at your application level, but also in the container itself, as well as the underlying
infrastructure. Defense in Depth could pretty much be a talk on its own, but here are some tips on
how to further harden your services.

The most impactful change you should make is, to not run your applications as the root user. If an
attacker maanges to escape the container, by running as root in the container, they will have root
outside of the container. This is not universally true, as rootless container runtimes exists, but
all in all, creating a dedicated user for your app in the image is the preferred way.

Additionally, try to run your containers only with the absolute minimum of capabilities required by
your services. For example, running our `vulnerable-awk-playground` as a container with the
`--privileged` flag would be a recipe for disaster. For an attacker it would then be trivial to
escape from the container, by e.g. loading a kernel module.

Next, when running the container, make sure it has a read only filesystem. This makes it once again
harder for attackers to for example download a crypto miner. There are still ways around this, but
they will create more noise and are hence more probable to be detected.

Another very important tip, only use host mounts, if there is no way around them. Host volume
mounts, or host path mounts in k8s punch a hole through the isolation we get from containers right
to the underlying host. There have already been numerous vulnerabilities with host paths, where it
allowed processes within containers to read arbitrary files on the host.

Alright, we are now nearing the end of my talk. If you would like to learn more about securely
building containers or container security in general, I would recommend you hit over to my blog at
patrickpichler.dev. It currently looks a bit empty, but trust me, I have some very interesting
articles lined up.

I hope you enjoyed the talk and if you have any questions, remarks or whatever, feel free to
approach me and talk to me!
