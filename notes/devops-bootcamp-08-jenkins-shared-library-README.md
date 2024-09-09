[Notes overview](https://github.com/TheAbys/devops-bootcamp-certification-project/blob/master/README.md)

# 14 - Jenkins Shared Library

Makes parts of pipelines reusable in different projects without copying everything.
Copied the repository, checked out starting-code branch and updated the existing code to my own code.

Global Pipeline Libraries section within Manage Jenkins => System allows to configure a specific git repository which holds the shared libaries.
It is good practise to define a specific version of the shared libraries and not the master branch for example.

You could use the Docker class directly within the Jenkinsfile, but it is best practise to not do that.
