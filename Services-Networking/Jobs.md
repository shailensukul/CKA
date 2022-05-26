# Jobs

A [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

You can also use a Job to run multiple Pods in parallel.

If you want to run a Job (either a single task, or several in parallel) on a schedule, see [CronJob.](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

Running an example Job
```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
      metadata:
        labels:
         app: batch-job // pods will be selected based on this template
    spec:
      containers:
      - name: main 
        image: liksa/batch-job
      restartPolicy: OnFailure // Job cannot use the Always restart policy
  backoffLimit: 4
```

## Job details

Get jobs
`kubectl get jobs`

Get pods for the job
`kubectl get pods --show-all`

View logs in the completed pod
`kubectl logs pod-a`

Delete job
`kubectl delete job batch-job`

## Parallel execution of jobs

1. Non-parallel Jobs - the job is complete as soon as its Pod completes
2. Parallel Jobs with a fixed completion count - `.spec.completions` determines the number of successful Pods for the Job to complete
3. Parallel jobs with a work queue - `.spec.parallelism` once at least one Pod has terminated with success, all Pods are terminated
4. Parallel jobs with a work queue and a fixed completion count - set `.spec.completions` AND `.spec.parallelism`. For ex:  `.spec.completions = 5` and `.spec.parallelism = 2` will run 2 pods in parallel and when one finishes, another one is run, until 5 pods finish successfully.

## Programatically scaling a job

`kubectl scale job batch-job --replicas 3`

## Job Termination and Cleanup

`.spec.activeDeadlineSeconds` - fails a jobs when this durartion is passed

`.spec.backoffLimit` - specify the number of retries before considering a jobs as failed
