# akuity-demo

A GitOps pipeline built on the Akuity Platform as part of a Sales Engineer interview assignment. What started as "spin up a cluster" turned into a genuinely fun deep dive into Argo CD and Kargo.

## What's in here

    applicationsets/
        guestbook-appset.yaml   # one file to rule dev, staging, and prod
    kargo/
        warehouse.yaml          # watches for new image versions
        stage-dev.yaml          # first stop
        stage-staging.yaml      # second stop
        stage-prod.yaml         # the real deal
    README.md                   # you are here

## The setup

Single local Kubernetes cluster running via Rancher Desktop, connected to an Akuity Platform trial. One Argo CD instance, one Kargo instance, and three namespaces pretending to be three different environments. It works surprisingly well.

## How it's wired together

### Argo CD
The guestbook app (classic for a reason) gets deployed across three namespaces — guestbook-dev, guestbook-staging, and guestbook-prod — via a single ApplicationSet using a List generator. 

The reason I went with ApplicationSet instead of three separate Application manifests is simple: if you ever need a fourth environment, you add one line. That's the kind of thing that makes platform teams happy at 2am.

Each app carries a kargo.akuity.io/authorized-stage annotation so Kargo has permission to trigger syncs. Learned that one the hard way.

### Kargo
A Warehouse subscribes to ghcr.io/akuity/guestbook and discovers new image versions as freight. Three stages sit downstream — dev, staging, prod — each one only accepting freight that survived the previous stage.

Dev auto-promotes because fast feedback loops matter. Staging and prod require a human to click the button, because some decisions shouldn't be automated.

## Decisions I'd make again

**ApplicationSet over manual apps** — the templating pays off immediately and keeps the repo clean.

**Namespace isolation over multiple clusters** — for a local demo this is the right call. In production you'd want real cluster separation for stronger blast radius control, but namespaces get the concept across cleanly.

**Manual gates on staging and prod** — auto-promoting all the way to prod would be a bold choice. Keeping humans in the loop on the last two stages felt like the right default.

## Things that humbled me

The original guestbook image on GCR is no longer publicly accessible. Docker Hub has rate limits that will find you eventually. ghcr.io/akuity/guestbook ended up being the reliable choice.

Rancher Desktop on an Intel Mac running Ventura will occasionally lose its network connection. When in doubt, restart it.

## Resources that actually helped

- https://docs.akuity.io
- https://docs.kargo.io
- https://github.com/argoproj/argocd-example-apps
