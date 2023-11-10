* Rename Dockerfiles to Containerfiles (???)
* Mention jib/ko
* Show CVE table
* Define Keytake aways
    * Supply chain
        * What offers apko/melange OOTB
        * SBOM
        * What about image signing
    * CVE notification overload
        * Broken window theory
    * Attack Surface
        * e.g. curl/wget installed => easy to install C2 implant
    * Not a silver bullet
        * Defense in depth
        * Runtime security also important
* Lots to think about when using dockerfiles in a naive way
    * e.g. deletion of files only creates a marker in a layer
    * most people will be devs, not security experts
        * show how tools help you avoid simple mistakes
