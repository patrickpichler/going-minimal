# Problem with big container images
* Waste a lot of disk space
    * Longer pull times
    * More storage costs
* Bigger attack surface
    * Less binaries == less things that can be exploited by hackers
* Lots of noise caused by CVEs in unused components
    * Signal overload might lead to serious "real" CVEs being missed

# Meet the scratch image
* It is possible to use scratch containers and just run a static build binary
    * ==> Perfect to demo
* Works somewhat well for static binaries build with languages such as Golang, Rust, C
* Often static binaries still need supporting files/directories, such as root certificate data for TLS connections, or /tmp/ folder

# Meet Distroless philosophy
* Only contain bare minimum
    * E.g. /tmp/ folder, TLS root certs
* Various different projects that offer such images
    * Google Distroless
    * Chainguard Images
* Due to lack of most binaries, number of unrelated CVEs should be significantly lower
    * Some variants do not even offer a shell
* Not without problems
    * Lack of shell makes it somewhat hard to debug applications running in a distroless container
    * Solution for this: ephemeral containers
