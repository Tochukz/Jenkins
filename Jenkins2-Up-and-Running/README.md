# Jenkins 2 Up and Running
#### by Brent Laster (O’Reilly)
[Example Code Repo](https://resources.oreilly.com/examples/0636920064602)

## Chapter 1: Introducing Jenkins 2
Instead of filling in web forms to define jobs for Jenkins, users can now write programs using the Jenkins _Domain-Specific Language_ (DSL) and Groovy to define their pipelines and do other tasks.  

__The JobConfigHistory Plugin__  
It tracks the history of the XML configuration changes over time and allows you to look at what changed each time. It is available on the [Jenkins wiki](https://wiki.jenkins.io/display/JENKINS/JobConfigHistory+Plugin)


__Feature of Jenkins2__
* _Jenkinsfile_
* Declarative pipelines-as-code
* _Blue Ocean_ visual interface
* New Job types and Plugins  

__New Jobs/Project types__  
* _Pipeline_: Pipelines can either be written in a "scripted" syntax style or a "declarative" syntax style.  
* _Folder_: This allows for grouping projects together under a shared namespace and shared environment.
* _Organization_: Repositories are grouped into "organizations". When changes are make in a repository and Jenkins is notified, it detected the `Jenkinsfile` as a marker in the repository and executed the command in the `Jenkinsfile` to run the pipeline.
* _Multibranch Pipline_: This provides easy automated job creation per branch and continuous integration, all triggered by `Jenkinsfiles` residing in the branch.  

You can still create jobs using web-based forms using the _Freestyle project_ but _Pipeline_ jobs is  the emphasis of Jenkins 2.

Legacy Jenkins typically relied on web forms to input data and stored it in XML configuration files in its home directory.

__Pivotal__'s Concourse, uses containerization to do jobs and allows pipelines to be described in YAML files.

__Global Configuration__  
In the current version of Jenkins, global configuration is split between _Configure System_ and _Global Tool Configuration_ pages. The former may generally be used for server setup or similar tasks and the later is for standalone executable application such as Git, Gradle.  

To check for the compatibility of plugins go to [Github](https://github.com/jenkinsci/pipeline-plugin/blob/master/COMPATIBILITY.md) or [Jenkens.io](https://jenkins.io/doc/pipeline/steps/)

In writing your code for a pipeline, you can choose from the traditional, more flexible _Scripted Pipeline_ or the more structured _Declarative Pipeline_ syntax.  

## Chapter 2: The Foundations  
__Syntax: Scripted Pipelines Versus Declarative Pipelines__  
_Script syntax_ refers to the initial way that pipelines-as-code have been done in Jenkins. It is an imperative style. It is also more dependent on the Groovy language and Groovy constructs.    
_Declarative syntax_  is a new option in Jenkins, the code is arranged in clear sections that describe the state and outcome desired rather then focusing on the logic to accomplish it.

__Choosing Between Scripted and Declarative Syntax__  
The declarative model is structured, easier to learn and maintain.
The scripted model offers more flexibility, allowing users to do more custom operation with less imposed structure.

__Systems: Masters, Nodes, Agents, and Executors__  
Every Jenkins pipeline has to have one or more systems to execute code on. There can be multiple instances of Jenkins on any given system or machine.
__Master__  
A Jenkins master is the primary controlling system for a Jenkins instance. It is not intended for running any heavyweight tasks.  
__Node__  
Node is the generic term that is used in Jenkins 2 to mean any system that can run Jenkins jobs. This covers both masters and agents. Furthermore, a node might be a container, such as one for Docker.
__Agent__  
Same as _slave_ in earlier versions of Jenkins, it refers to any non-master system. These systems are managed by the master system and allocated as needed, or as specified, to handle processing the individual jobs.

As far as the relationship between agents and nodes goes, agents run on nodes.  
In a Scripted Pipeline, "node" is used as the term for a system with agent. In a Declarative Pipeline, specifying a particular agent to use allocates a node.  
__Directives Versus Steps__  
_node_ is associated with Scripted Pipeline. _agent_ on the other hand, is a directive in a Declarative Pipeline.   
__Executor__  
Basically, an _executor_ is just a slot in which to run a job on a node/agent. The number of executors defines hoe many concurrent jobs can be run on that node.   
When the master funnels jobs to a particular node, there must be an available executor slot in order for the job to be processed immediately. Otherwise, it will wait until an executor becomes available.  

__Leveraging Other Languages__  
If you need to access/use functions written in Groovy or another language, or ones that involve a more iterative workflow, you can make them part of a shared library. That way they will be abstracted out from your main pipeline code base.  


__Node and Agents__
Each node has a Jenkins agent installed on it to execute jobs.  

Simple pipeline expressed in Jenkins DSL:  
```
node ('node_label') {
  stage('Source') {
    //Get some code form out Git repository
    git 'https://github.com/brentlaster/gradle-greeting.git'
  }
}
```
If you omit supplying the node_label then Jenkins will handle the job in one of two ways:
* run the job on master if master has not been configured to not run any jobs.  
* run the job on the first executor that becomes available on any node.  
Omitting the node_label is same as having _agent any_ in declarative syntax.  
It is also valid to use multiple names with logic operators.   

[Groovy documentation](http://groovy-lang.org/)  

__Supporting Environment: Developing a Pipeline Script__  
A pipeline script in Jenkins can either be created within a Jenkins job of type _Piepline_ or as an external file named _Jenkinsfile_.

__Working with the Snippet Generator__  
The Snippet Generator content is seeded and updated based on definitions of pipeline steps added by plugins.
The content of the Snippet Generator on any particular Jenkins instance is a function of what plugins are installed on that instance.  

If the _poll_ parameter in the _git_ step is set to true, then changes in the remote repository will be detected and trigger another run of the job.  

__Running a Pipeline__  
One of the common visualization plugin is the _Build Pipeline_ plugin. It allows for setting up special views that displays a series of jobs in a pipeline as connected sets of boxes with color code:
* blue for job not run yet
* yellow for job in progress
* green for successful run
* red for failed run

The color processing legend of the stage tiles in the stage view have the following meaning:  
* Blue stripes: Processing in progress  
* White: Stage has not been run yet  
* Rose stripes: Stage failed  
* Green: Stage succeeded  
* Rose: Stage succeeded but some other stage failed downstream.  

Go to [Github](https://github.com/jenkinsci/JenkinsPipelineUnit) to see Jenkins Pipeline Unit testing framework.  

## Chapter 3: Pipeline Execution Flow  
See the [Conditional BuildStep](https://plugins.jenkins.io/conditional-buildstep) plugin.

Multibranch Pipeline, Github organization, or Bitbucket team/project jobs are identified by having `Jenkinsfiles` and are triggered in a special way such as by a webhook that notifies Jenkins when a change is made.  

### Triggering Jobs  
#### Build After Other Projects Are Built
```
properties([
  pipelineTriggers([
      upstream(
        threashold: hudson.model.Result.SUCCESS,
        upstreamProjects: 'Job1'  
      )
    ])    
])
```
For multiple upstream jobs, separate them with commas. To specify a branch (as for a multibranch job), add a slash after the job name as in `Job1/master`.  
#### Build Periodically  
This option it not optimal for continuous integration, where the builds are based on detecting updates in source management.  
```
properties([
  pipelineTriggers([
      cron('0 9 * * 1-5')
    ])  
])
```
In this Scripted Pipeline, the job runs at 9 am, Monday -Friday.  
For Declarative Pipeline, the syntax will look like this:  
```
triggers { cron(0 9 * * 1-5)
```  
__Cron Syntax__
The five fields of the cron syntax are:  
`MINUTE(0-59) HOURS(0-23) DAYMONTH(1-31) MONTH(1-12) DAYWEEK(0-7)`   
On `DAYWEEK(0-7)` both 0 and 7 represents Sunday.  
Also the ``*/<value>`` syntax means "every <value>" as in `*/5` means every minute.  

Exercise:
Interpret the following cron and check your answer with the answer 0n page 62.
```
triggers { cron(15 * * * *) }
triggers { pollSCM(*/20 * * * *) }
triggers { cron(H(0,30) * * * *) }
triggers { cron(0 9 * * 1-5) }
```
The `H` symbol is used for range. It assigns a random value between a range. `H` also represents hash of the project name.  
#### Github Hook Trigger for GitSCM Polling  
Go the [Jenkins Wiki](https://wiki.jenkins.io/display/JENKINS/GitHub+Plugin#GitHubPlugin-GitHubhooktriggerforGITScmpolling) to learn more about Github hook triggers.  
The syntax fr setting the property in a Scripted Pipeline is:
```
properties([
  pipelineTriggers([
    githubPush()
  ])
])
```
Currently, a specific syntax may not be available for Declarative Pipelines.  
#### Poll SCM
The can be a very expensive operation depending on the SCM, size of repo and frequency of poll.
The syntax for Scripted Pipeline is
```
properties([
  pipelineTriggers([
    pollSCM('*/30 * * * *')  
  ])  
])
```
This will check the source repo for changes every 30mins and build if there is a change  
The corresponding syntax for Declarative Pipeline is
```
triggers { pollSCM(*/30 * * * *)}
```
#### Quiet Period  
This value serves as the "wait time" between when the build is triggered and Jenkins acts on it.  
The pipeline build step has a `quietPeriod` option but there isn't a direct pipeline option or step to do this. You may achieve a similar effect using the [Throttle Concurrent Builds](https://github.com/jenkinsci/throttle-concurrent-builds-plugin) plugin.  

#### Trigger Builds Remotely  
You can trigger a build by accessing a specific URL for the given job on the Jenkins system.  You can trigger a build via hook or a script. An authorization token is required.

#### User Input  
The DSL step `input` is the way we get user input through a pipeline. For Declarative Pipeline, there is a special `parameters` directive that supports a subset of those parameters.  
The `input` step allow your pipeline to stop and wait for a user response:  
```
input 'Continue to next stage?'
```
Using the `input` step in a `node` block ties up the executor for the node until the input step is done.  
The `input` step can have the following parameters - `message`, `id`, `ok`, `submitter`
```
input id: 'ctns-prompt', message: 'Continue to the next stage?'
```

## Chapter 7: Declarative Pipelines  
## Chapter 8: Understanding Project Types
## Chapter 9: The Blue Ocean Interface
## Chapter 10: Conversions  


Regression ?
