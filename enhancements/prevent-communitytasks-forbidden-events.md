# Enhance community task fetch from external/public repo

## Issue: [#SRVKP-1729](https://issues.redhat.com/browse/SRVKP-1729)

The issue states to add mechanism to disable fetching community tasks because there are hug logs for forbidden events.
To elaborate more if the operator is installed in an air-gapped environment then many of the external URL's are not reachable and when the operator try to access those URLs it logs forbidden errors.
The community tasks are directly coming from upstream github catalog source and in air-gapped environment they are not reachable.


## Proposal
### Solution 1
Add a toggle in the tektonconfig which a user can set to `false` if they don't want to install community tasks from repo

- Implementation 1.1 
    Add <span style="color: green"> enableCommunityTask: true </span> as spec element of addon

```
spec:
  addon:
    enablePipelinesAsCode: true
    enableCommunityTask: true 
    params:
    - name: pipelineTemplates
      value: "true"
    - name: clusterTasks
      value: "true"
  config: {}
  dashboard:
    readonly: false
```
- Implementation 1.2
    Add <span style="color: green"> enableCommunityTask: true </span> as Params of addon


```
spec:
  addon:
    enablePipelinesAsCode: true
    params:
    - name: pipelineTemplates
      value: "true"
    - name: clusterTasks
      value: "true"
    - name: enableCommunityTask  
      value: "true"
  config: {}
  dashboard:
    readonly: false
```
#### Pros
- Easy implementation (less code change)
- We don't even try to fetch the urls

#### Cons
- Needs a person to disable it manually setting the enableCommunityTask to false
- Increase burden of the tektonconfig for one more toggle.
- We don't even try to fetch the urls




### Solution 2
As the issue is only about getting too many `forbidden` events and its logs, we don't need to completly disable fetching the community tasks, what we can do is add a `limit` or `timeout` to the fetch calls.
This way we reduce the `forbidden` calls and so the logs.

#### Implementation 2.1
Add `try counter` label to <i>TektonAddon</i> whenever we try to fetch the community tasks from the url we increase the counter, if the counter reaches certain limits E.g. - `5` we skip trying to fetch the community tasks after that.

#### Pros
- No intervention from user
- Less `forbidden` events
- No additional burden on the `tektonconfig`

#### Cons
- What if the user don't want the community tasks.

### Solution 3
Use a combination of `toggle` and `retry time` mechanism, so even if the community tasks installation is enabled and the urls are not reachable in that case this retry mechanism would help to reduce the  frequent `forbidden` logs. The next try to fetch urls would happen only after given time. 
Kept as `15 minutes` for now.
`Note: keeping wait time for 15 minutes`

#### Implementation 3.1
Check for the `toggle` value from the config.
- If disabled, do not fetch the urls and move ahead to install other cluster tasks.
- If enabled, check if operator should try to fetch.
    - To do this we check for label in addons
        - `operator.tekton.dev/tryAfter`
    - If the `operator.tekton.dev/tryAfter` value is nil we try to fetch the url
    - If value if not nil we check if the current time is greater than the set time
        - If yes, try to fetch
            - else no
- If operator fetch the url and gets and error. Its sets a current time + 15 minutes as Label  `operator.tekton.dev/tryAfter` in the Addon
    

#### Pros
- Less `forbidden` events
- No additional burden on the `tektonconfig`
- User get feature to completely disable/enable community tasks 

#### Cons
- 

