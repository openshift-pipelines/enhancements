# Advance auto-pruning 
- ## Actual advance auto-pruneing doc [here](https://docs.google.com/document/d/1259TGcQxVrxycrFS1YbqVUyebpc7cg8_qpogtdUpMPg/edit)

- ## Initial auto-pruning doc [here](https://docs.google.com/document/d/1DrIzaN5jm5QV_TysArJO0W4Eurfh5z_GR-eaxkGuz88/edit)

## Acceptance: 
Admin can configure the operator to automatically prune the pipelinerun with the following configurations:

- Keep last N pipelineruns for each pipeline
- Keep pipelineruns younger than D days for each pipeline
- Keep pipelineruns and taskruns that are younger than D days
- Support per namespace auto-prune

    - Do we want to to prune all the ns by default andor sikp pruning for annotated ones
    - Or Do we want to to disable prune for all the ns by default and enable pruning for annotated ones
- Do not delete pipelineruns/taskruns which are in running state.
- Should be enabled by default 


## How are we gonna tackle these
- Keep last N pipelineruns for each pipeline

        Here tkn can help out 
        tkn taskrun delete --task=abcTask --keep=2 -n foo 
        tkn pipelinerun delete --pipeline=abcPipeline --keep=2 -n foo
    - Question : 
        - How to get the abcPipeline/abcTask from user

 
- Keep pipelineruns/taskruns younger than D days for each pipeline/task

        Here also tkn will do it.
        tkn taskrun delete --task=abcTask --keep-since=<n-minutes> -n foo 
        tkn pipelinerun delete --pipeline=abcPipeline --keep-since=<n-minutes> -n foo
        The problem is we already have keep now we want to have keep-since, 
        how do we take input from the user.



- Keep pipelineruns and taskruns that are younger than D days

        Here also tkn will do it.
        tkn taskrun delete --keep-since=<n-minutes> -n foo 
        tkn pipelinerun delete --keep-since=<n-minutes> -n foo
        We may need to address keep and keep-since conflict
            In my opinion this could be mutually exclusive.
- Support per namespace auto-prune

        List all the namespaces which have specific annotation. As suggested in the issue>
            Annotation:
                pipeline.openshift.io/namespace/prunable (open for other options)
                pipeline.openshift.io/prunable
            Changed to operator.tekton.dev

            Question: Do we assume that user will add these annotations to the namespaces.
- Do not delete pipelineruns/taskruns which are in running state. (this came from a pr discussion)

        - We do have keep and keep-since, with keep-since we specify time, so 
          the user will already know that something is running.
        - But if we want to get this in , would it make sense to add this in 
          cli a flag which ignore all the pipelinerun/taskrun running 







### Tasks 
- Keep last N pipelineruns for each pipeline				- **Done** 
- Keep pipelineruns younger than D days for each pipeline		- **Done**
- Keep pipelineruns and taskruns that are younger than D days		- **Done**
- Support per namespace auto-prune
- Annotate ns with skip:true/false				-   **Done** 
- Create a separate cronjob if a diff schedule value is set in the ns annotation pipeline.openshift.io/prune.schedule”	-	**Done**
- If the annotation is deleted on ns or if skip:true is added and if there is an existing cron we should  delete it.	-	**Done**
- Do not delete pipelineruns/taskruns which are in running state.

    - Need cli release for taskrun, pipelinerun 
    - Should be enabled by default 
- Added to all, basic, lite of kubernetes and openshift. **Done** 
- CLI in review
    - https://github.com/tektoncd/cli/pull/1435/files   **Done**
    - https://github.com/tektoncd/cli/pull/1436/files    **InReview**
    - Created an issue https://github.com/tektoncd/cli/issues/1442

## Question:
    Q1 Annotation are assumed to be done by ClusterAdmin/user with some privilege  
	Its fine to assume.
    
    Q2 Is pruning done by default
	Its should not be done if there is an update.
	Its should be enabled by default, if new install
    
    Q3 Keep-since should be ignored
	Give doc about it, any one of the item should be specified. 

project-execute-extended-command
Open questions
Relation with the Results and it’s availability/support in OpenShift Pipelines (and devconsole, …) ?

## Updates on 23rd Sep
This comes after pr review for the advance prune features like pruning per namespaces, and pruning with related resource https://github.com/tektoncd/cli/pull/1445

We figured out the approach we are taking, which is to  put annotations on the namespaces, is not scalable.
"operator.tekton.dev/prune.resources.pipelineNames"
"operator.tekton.dev/prune.resources.taskNames"
The names specified in the above annotation were used as related resource names. Like delete all the pipelinerun related to pipelineNames ( with other configs like keep, keepsince)

 
## Another Approach Suggested

    We remove  the earlier defined annotation
    "operator.tekton.dev/prune.resources.pipelineNames"
    "operator.tekton.dev/prune.resources.taskNames"
    And introduce a new  annotation "operator.tekton.dev/prune.foreach-pipeline"

    If this annotation is found we get all the pipeline in the namespace and apply other configs like keep, and keepsince while deletion.
    This comes out as , Delete pipelineruns related to pipeline keep/keepsicne for all the pipelines in that namespace.

    With this instead of take specific pipeline names from users we just ask users if they want to keep x pipelineruns for each pipeline in the namespace.


