# Going Minimal: A guide to improving container image security

## Story
* A few months ago I attended WeAreDevelopers
* Few talks about cloud/k8s/containers
* I was surprised that nobody used minimal container images
* Nice look outside my bubble
* Now I am standing here to tell you about our lord and saviour: minimal container images

## Outline
* Take a small sample application (or well known small open source project)
* Show that it currently uses Dockerfile + docker build for building the image
* Base image should be something like ubuntu to really drive the point home
* Image should also build the application in the dockerfile
* Show that it is pretty big and contain a lot of unused dependencies
* Use a scanner such as trivy to show that it is also riddled with vulnerabilities
* Explain why bloated images are an issue
* Ask question on how to improve image security
* First step introduce multistage builds, so that build dependencies are gone from runtime image
* Nice, image size reduced, but we can do even better
* Next step: switch out base image to either distroless/chainguard
* Explain what distroless/chainguard projects are
* Show that this now reduced the image size by quite a bit
* Rerun the trivy scan, show that there are now less vulnerabilities
* Go a step further and show projects such as ko/jib
* Build a minimal container image with ko, show how easy it is
    * Or maybe jib (??) I am not sure if more people will be familiar with java or golang
    * Maybe stick with ko + golang, as it is easier later with apko/melange
* There is more
* Introduce apko/melange/wolfi
    * Show that wolfi is an specialized linux distribution using the apk package manager
    * It uses glibc
    * Maintained by chainguard
* Show example how to build specialized image with melange
* Show that you can easily turn your project in to an apk package by using apko
    * Put focus on that in the end it is just a declarative yaml
    * Compare with imperative dockerfile approach
* Bonus point: show that an sbom can also easily be generated (??)
    * this needs some more research
