Below I have attached a file, 
it will covers all the basic concept to know and get started with jenkins.
```
git restore <filename> ==> removes the files from the working area
git restore --staged <filename> ==> remove the file from stagging and send back tot he working area
git rm --cached <filename> ==> removes the file from the stagingarea and sent back to the working area


git revert <Hash>  ==> Create a new commit, taht were made on the commit that we specified, (Exactly all the opposite change to the original commit)
						if you want to undo changes, and kepp those changes in you git history.
git reset --soft HEAD~1 ==> To delete the last commit and bring the changes back to the staging area(Files are exist in working area, still has access to  the files) and point to the2nd latest commit.
git reset --hard HEAD~1 ==> To delete the last commit(No more file associated with latest commit) point to the 2nd latest commit.


:::Jenkins:::

Jobs ==>
Builds ==>
Freestyle Project ==>
Pipelines ==>
Stages ==>
Nodes ==>
Plugins ==> 

Pro's 
Open source, 
Highly Customizable,
Scriptable for advanced users
pipeline as code
mature and feature rich
scalable
Con's:
Steeper Learning curve
Maintenance Overhead
Perform Considerations
Security Concerns
Hosting Required
"""
CI/CD:
	flow: git >> Developer >> feature/branch >> commit >> pull req >> review >> here two paths 1. ci >> cd >> approve >> Staging(Test) for testing purpose,
																				               2. >> branch merge to main >> Devloper >> CI >> CD >>  
																				
Continuous Integration: Unit Testing, Dependency Scan, Build Artifact, Code Scanning

Continuous Deployment: 
	Afetr compeletion of CI, 
	1. if feature Branch: Deploy(Staging) >> Kubernetes(Continuous Deployment)
	2. If Main Branch: Manual Approval >> Deploy(Prod Kubernetes) >>Smoke (Test) (Continous Deployment)

Jenkins Architecture:
Jenkins Controller Node(Master Mind, Coordinate the entire CI/CD Process): Plugins, Jobs, Nodes, Credentials, Configurations
	Responsibilites:
		Management Task: User Authentication & Authorization
		Job Managment: Defining, Schudeling, and Monitoring the executions
		Web Interface: for configuration, setup pligins, onboard users and monitoring and managing the builds within Jenkins.

For Basic Deployment (Controller & Worker node are the same)
Advance Deployment(Sepereate controller and worker nodes) why? Prevent Accidental Damage, Boosts Performance, Scalibility for growth.

Nodes:
	formerly known as Jenkins slaves.
	Linux Node or Windows Node
	inside node we have agents, inside it we have executors
	
Overview:
	
	Jenkins Controller Node Connected  to two worker nodes.
	yopu define jobs and pipelines within the controller using its UI, CLI, or rust APIs, 
	The controller identifies avakliable executors on connected nodes and then schedules and distributes jobs to the avaliable executors.
	on nodes, the nodes execute the jobs using the allociated resources,
	once the ececution complete the controller receives results and feedback from the nodes providing the build status the logs, and the artifacts.
	

Hardware:
Min 
2 CPU cores, RAM:256MB, DISK Space:1GB
recom
4 CPU cores, RAM:4GB, DISK Space:50GB+

Software:
Web Browser, JRE or JDK

Two Types of Jenkins Project:
Free Style, limites workflow, non-code base config, complexity challenges, limited functionallity, cannot resume after failure
Pipeline Project
Multi Branch Pipeline
Maven Projevt
Multi config Project
Organization folders

Plugins:
	Source Code Managment: integrate with Git(Github, Gitlab, bitbucket)
	Build Tools: Support for Maven, Gradle
	Quality Checks: Code Coverage, Static analysis
	Notifications: Alert via Slack PagerDuty
	cloud integration: Connect to AWS, GCP, Azure
	Scalibility: Distributed builds with worker nodes
Jar files: formate .hpi(Hudson Plugin), .jpi(Jenkins Plugin)
Stored in ${jenkins_home or /var/lib/jenkins/}/plugins directory
Content: code, resources, configurations
priotity: .jpi takes priority over .hpi if duplicates exist

Jenkins Fingerprints:

Pipelines:
	scripted pipelines, Declarative pipelines
```
