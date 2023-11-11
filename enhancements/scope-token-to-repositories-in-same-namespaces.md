---
title: Extend GitHub token scope to a list of provided repositories within and outside namespaces
authors:
  - "@savita"
creation-date: 2023-04-17
status: implementable
---

# Extend GitHub token scope to a list of provided repositories within and outside namespaces

## Summary

The proposal helps user to scope GitHub token to a list of provided repositries which exist in same namespace(By providing configuration at Repository level) as well as different namespace(Global Configuration).

## Motivation/UseCase

Their is a use case where CI Repos Differ from CD Repos, and the teams would like the generated GitHub Token from Pipelines As Code to allow control over these secondary repos, even if they were not the one triggering the pipeline.

Ex: 

CD Repos are private and CI Repos have `.tekton/pr.yaml` and payload coming from CI repos where pr.yaml fetches some tasks from CD Repos which is Private.

story :

<https://issues.redhat.com/browse/SRVKP-2911>

## Proposal

There are 2 ways to scope GH token to a list of provided Repos

1. Scoping GH token to a list of Repos provided by global configuration
2. Scoping GH token to a list of Repos provided by Repository level configuration

**Pre-requisite:**

 Disable `secret-github-app-token-scoped` to `false` from `pipelines-as-code` configmap in order to scope GH token to private and public repos listed under Global and Repo level configuration.

### Scoping GH token to a list of Repos provided by global configuration

* When list of Repos provided by global configuration then scope all those Repos by a Github Token irrespective of the namespaces.

* The configuration exist in `pipelines-as-code` configmap.

* The key which used to have list of Repos is `secret-github-app-scope-extra-repos`

  Ex:

  ```
  apiVersion: v1
  data:
    secret-github-app-scope-extra-repos: "owner2/project2, owner3/project3"
  kind: ConfigMap
  metadata:
    name: pipelines-as-code
    namespace: pipelines-as-code
  ```

### Scoping GH token to a list of Repos provided by Repository level configuration

* Scope token to a list of Repos provided by `repo_list_to_scope_token` spec configuration within the Repository custom resource

* Repos can be private or public

  ```
  apiVersion: "pipelinesascode.tekton.dev/v1alpha1"
  kind: Repository
  metadata:
    name: test
    namespace: test-repo
  spec:
    url: "https://github.com/linda/project"
    repo_list_to_scope_token: 
    - "owner/project"
    - "owner1/project1"
  ```

  Now PAC will read `test` Repository custom resource and scope token to `owner/project`, `owner1/project1` and `linda/project` as well

  **Note:** 

  1. Both `owner/project` and `owner1/project1` Repository should be in same namespace where `test` Repository exist which is `test-repo` ns.

  2. If any one of the `owner/project` or `owner1/project1` doesn't exist then scoping token will fail

     Ex: 
     
     If `owner1/project1` does not exist in the namespace

     Then below error will be displayed

     ```
     repo owner1/project1 does not exist in namespace test-repo
     ```

### Scenarios when both global and Repo level configurations provided

1. When Repos are provided by both `secret-github-app-scope-extra-repos` and `repo_list_to_scope_token` then token will be scoped to all the Repos from both configuration

    Ex:

    * List of Repos provided by `secret-github-app-scope-extra-repos` in cm

        ```
        apiVersion: v1
        data:
          secret-github-app-scope-extra-repos: "owner2/project2, owner3/project3"
        kind: ConfigMap
        metadata:
          name: pipelines-as-code
          namespace: pipelines-as-code
        ```

    * List of Repos provided by `repo_list_to_scope_token` in Repository spec

        ```
        apiVersion: "pipelinesascode.tekton.dev/v1alpha1"
        kind: Repository
        metadata:
          name: test
          namespace: test-repo
        spec:
          url: "https://github.com/linda/project"
          repo_list_to_scope_token: 
          - "owner/project"
          - "owner1/project1"
        ```

    Now the GH token will be scoped to `owner/project`, `owner1/project1`, `owner2/project2`, `owner3/project3`, `linda/project`

2. If only Global `secret-github-app-scope-extra-repos` set then token will be scoped to all the provided repos 

3. If only repos are provided by Repository spec using `repo_list_to_scope_token` then token will be scoped to all provided repos only when all repos exist in the same namespace where Repository created. 

4. If no Github App is installed for the provided Repos in both global and repo level configuration then scoping token will fail with below error

```
failed to scope token to repositories in namespace article-pipelines with error : could not refresh installation id 36523992's token: received non 2xx response status \"422 Unprocessable Entity\" when fetching https://api.github.com/app/installations/36523992/access_tokens: Post \"https://api.github.com/repos/savitaashture/article/check-runs\
```

5. If repos are given by `repo_list_to_scope_token` or `secret-github-app-scope-extra-repos` failed to scope token for any reason then CI will not run.

6. repo `owner5/project5` is given globally as well as at Repo level using  `secret-github-app-scope-extra-repos` and `repo_list_to_scope_token` 

    Ex:

    ```
    apiVersion: v1
    data:
      secret-github-app-scope-extra-repos: "owner5/project5"
    kind: ConfigMap
    metadata:
      name: pipelines-as-code
      namespace: pipelines-as-code
    ```

    ```
    apiVersion: "pipelinesascode.tekton.dev/v1alpha1"
    kind: Repository
    metadata:
      name: test
      namespace: test-repo
    spec:
      url: "https://github.com/linda/project"
      repo_list_to_scope_token: 
      - "owner5/project5"
    ```

    still failed to scope token with below error
    ```
    repo owner5/project5 does not exist in namespace test-repo
    ```
    because `owner5/project5` doesn't exist in namespace `test-repo`.

