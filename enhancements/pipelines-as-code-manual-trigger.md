# pac direct triggerring - fire at will!

* We need to let the user fire a pipelinerun without going by Github or others.

* Pipelinerun still located ina a repository and a SHA.

* For github-app installation method we can generate a github temporary token at will

* For other methods they will  still need to attach a secret to repo.

* We need a nother field in spec for not allowing everyone to fire at will. a
  webhook secret directly in spec.

* We can automate thing with tkn-pac

* tkn-pac will :

 - grab the current revision from current dir
 - remind user to push the change to the remote repo.
 - generate a json payload (see below)
 - generate a webhook signature
 - post it to the eventlistenner.

* event coming to el from tkn-pac cli :

  - "header: X-Pac-Signature-256"
  - Same header as the one coming directly from git provider platform
  - Payload request handled is as what it is expected from the git provider platform repo
    ```json
    {
        "repository": {
            "owner": {
                "login": "openshift-pipelines"
            },
            "name": "pipelines-as-code",
            "default_branch": "main",
            "html_url": "https://github.com/openshift-pipelines/pipelines-as-code"
        },
        "sender": {
            "login": "chmouel"
        },
        "ref": "refs/heads/nightly",
        "head_commit": {
            "id": "f1c7ba995f0d494f71f69b1b6f764fc37fcb099b"
        }
    }
    ```
  - validation function (using go-github on any git provider platform )
    https://github.com/google/go-github/blob/25f8b0c90d4c75a4c41a5b6e0ac9bb983783fda0/github/messages.go#L162

* perhaps add a `event-type: manual` to be able to filter on those in pipelinerun annotations.

