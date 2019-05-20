# windows_jenkins_pipeline

# Syntax to call the Jenkins pipeline from workflow
def job = build job: 'say-hello', parameters: [[$class: 'StringParameterValue', name: 'who', value: 'DZone Readers']]

# Other links
https://blog.sonatype.com/continuous-integration-in-pipeline-as-code-environment-with-jenkins-jacoco-nexus-and-sonarqube

https://github.com/amarsingh3d/jenkins-pipeline
