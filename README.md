# akuity-demo

A GitOps pipeline built on the Akuity Platform as part of a Sales Engineer interview assignment for your next sales engineer (me).
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

Single local Kubernetes cluster running via Rancher Desktop, connected to an Akuity Platform trial. One ArgoCD instance, one Kargo instance, and three namespaces pretending to be three different environments. It works surprisingly well.

I had to use Racher instead of OrbStack, which is what I would have preferred, because I decided to use my wife's Macbook. Which, as it turns out, is older than I thought and could only be updated to Ventura. This meant that OrbStack was not compatible. 

## How it's wired together

### Argo CD
The guestbook app (classic for a reason) gets deployed across three namespaces: guestbook-dev, guestbook-staging, and guestbook-prod via a single ApplicationSet using a List generator. 

The reason I went with ApplicationSet instead of three separate Application manifests is simple: if you ever need a fourth environment, say some testing environment for a custom solution, you just add one line. That's the kind of thing that makes platform teams happy.

Each app carries a kargo.akuity.io/authorized-stage annotation so Kargo has permission to trigger syncs. Getting the permissions right for each stage was a key. 

### Kargo
A Warehouse subscribes to ghcr.io/akuity/guestbook and discovers new image versions as freight. Three stages sit downstream: dev, staging, prod. Each one only accepting freight that survived the previous stage.

Dev auto-promotes because fast feedback loops matter. Staging and prod require a human to click the button, because some decisions shouldn't be automated.

## Decisions I'd make again

**ApplicationSet over manual apps** — the templating pays off immediately and keeps the repo clean.

**Namespace isolation over multiple clusters** — for a local demo this is the right call. In production you'd want real cluster separation for stronger control, but namespaces get the concept across cleanly.

**Manual gates on staging and prod** — auto-promoting all the way to prod would be a choice. Keeping humans in the loop on the last two stages felt like the right default. I think any movement of freight should be blessed by a supervisor/manager/engineering lead.

## Things that humbled me

The original guestbook image on GCR is no longer publicly accessible. Docker Hub has rate limits that will find you eventually. ghcr.io/akuity/guestbook ended up being the reliable choice.

Rancher Desktop is not great. Running on an Intel Mac running Ventura - it will occasionally (very often) lose its network connection. So, hen in doubt, restart it. There wasn't any native feature that I saw that would reconnect containers without a restart. But I wasn't super familiar with it, so I could have missed it. 

## Resources that actually helped

- https://docs.akuity.io
- https://docs.kargo.io
- https://github.com/argoproj/argocd-example-apps
