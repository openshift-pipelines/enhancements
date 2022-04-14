# [SRVKP-2021](https://issues.redhat.com/browse/SRVKP-2021) — OSP "downstream" Operator per PR

###### tags: `openshift-pipelines` `downstream`

*End goal is to have this document in https://github.com/openshift-pipelines/enhancements*

## Motivation

The main motivation is to **provide a quick way to test changes in our product** before a change is merged upstream, and thus *without having to go through the whole release process*.

- This should help QE to try things as early as possible
- This should help developers know how their changes upstream will act in our product
- This allow a quick feedback loop while testing or developing.

## Design

As described in [SRVKP-2021](https://issues.redhat.com/browse/SRVKP-2021), this can be done in several steps. We'll initially focus on inputs and outputs, independently of the medium used (slack, …). We'll use the simplest possible medium (a HTTP webhook, powered by [tektoncd/triggers](https://github.com/tektoncd/triggers) or knative), letting the more advanced use cases (slack bot, GitHub PRs, …) to a later document.

In a nutshell, the idea is :

- An HTTP service is exposed on a cluster (ospqa)
- This services takes some inputs (what to build, where to push)
- This services returns the pipelinerun name and the unique name to identify the build later
- (optional) notify success or failures
  This is slightly independent of this as it should be setup on our dogfooding cluster no matter what.


### Inputs and Outputs

- **Input(s)**
    - A list of "git reference" / link
    - A *name*
- **Output(s)**
    - **unique** name (from input *name*)
    - A set of image tagged with that unique name
    - A bundle image tagged with that unique name
      *Runnable with `operator-sdk run …`*
    - (optional) An Index image (to use without `operator-sdk`)

### Inputs

The following "inputs" should be supported
- A GitHub Pull Request (on a component we track)
    - `https://github.com/tektoncd/pipeline/pull/4638`
    - `tektoncd/pipeline#pull/4638`
- A branch on a fork (of a component we track)
    - `https://github.com/vdemeester/tektoncd-pipeline/tree/4636-no-override-on-propagation`
    - `vdemeester/tektoncd-pipeline#4636-no-override-on-propagation`
- A commit on a component repository or a fork
    - `https://github.com/vdemeester/tektoncd-pipeline/commit/b3cbe872a3231dbf0c7d7cd59ae08329db9b22cc`
    - `vdemeester/tekton-pipeline@b3cbe872a3231dbf0c7d7cd59ae08329db9b22cc`

*Note: the format supported comes from : [`docker-build` ref](https://docs.docker.com/engine/reference/commandline/build/#git-repositories).*

One component can have one "input". A "relatively" advanced input in json could look like the following:

```json
{
    "name": "test-my-thing",
    "components": {
        "tektoncd/pipeline": "tektoncd/pipeline#4638",
        "tektoncd/operator": "vdemeester/tektoncd-operator@shiny",
        "openshift-pipelines/pipelines-as-code": "https://github.com/openshift-pipelines/pipelines-as-code/pull/25"
    }
}
```

### Flow example

*Note: at this stage, the following "flow" is purely speculative*

```bash
$ echo '{
    "name": "test-my-thing",
    "components": {
        "tektoncd/pipeline": "tektoncd/pipeline#4638"
    }
}' | http build.ospqa.com
{ "tag": "test-my-thing-72a32c", "pipelinerun": "build-operator-bundle-bc98b12" }
# ^^ unique name to make sure we don't clash with each others
# … and the name of the pipelinerun to watch the execution
# […]
# Building in the background
# […]
# Once the PipelineRun is done (how we know, TBD)
$ operator-sdk run -n openshift-operators bundle \
    quay.io/openshift-pipeline/openshift-pipelines-operator-bundle:test-my-thing-72a32c
```

### Details

#### How downstream *build* works

The *source of truth* today is in [openshift-pipelines gerrit](https://code.engineering.redhat.com/gerrit/plugins/gitiles/openshift-pipelines/).

- `upstream_source.yaml` ([1.7 example](https://code.engineering.redhat.com/gerrit/plugins/gitiles/openshift-pipelines/+/refs/heads/pipelines-1.7-rhel-8/upstream_sources.yaml)) is where we track code from upstream that has to be built.
  *This is one part of what we want to change*
- `distgit/containers` is where the `Dockerfile` of all images needed are defined.
- `Makefile` has the target to build a downstream operator based on `upstream_sources.yaml` (and the content of those sources)

The usual flow is

```bash
# Make sure upstream is up-to-date
$ make sources
# Fetch operator payload
$ make sources/operator/fetch-payload
# Build and push the images as well as the bundle
# … grab a coffe ☕, it can be long…
$ make REMOTE=quay.io/vdemeest bundle/push
# Run the operator on your OCP cluster
$ make REMOTE=quay.io/vdemeest bundle/run
```

#### `Pipeline` work

In a nutshell, the pipeline need to do the following

- Update `upstream_source.yaml` based on user inputs
- Generate payload from user inputs
  *This part is tricky*
    - Generate upstream payload
      e.g. for `tektoncd/pipeline#pull/2345` we would run `ko reslove -f config` on the source to generate the payload (even if we don't use the generated image laters)
    - Use the generated payload in the operator *fetch-payload* (`sources/operator/fetch-payload` target)
      The PR [tektoncd/operator#641](https://github.com/tektoncd/operator/pull/641) should help there.
- Build images and the bundle images

Some of those can be parallelized, some might not. It also depends if we want "one task to build all `make REMOTE=… bundle/push`" (like our current nightly) or if we want to split the load into several tasks (*might require the matrix feature / custom task*).


#### Service

There is two options available here:

1. Using `tektoncd/triggers`, similar to pipelines-as-code
   *And maybe an interceptor (?)*
   - Need an interceptor to transform the payload
   - *or* need a Task that translate it into results to be used in the next tasks
2. Our own service
   - Written in Go
   - Served using knative/serving

Most likely, (1) is the most "tekton" native option (and we can still use knative serving).


## Open Questions

- What about *patches* ? What if in downstream we have a patch that is render *nil* by the PR we are trying to build ? Should we have an option to *bypass* the patches (all, one, …?).

## Thoughts

- Even though we will use a `Pipeline`, we should aim to *not* use a PVC or a shared workspace. One reason is to make it as "quick" as possible.
  It should be relatively easy, if we do the same thing nightly does, *or* if we repeat some steps in multiple tasks.
