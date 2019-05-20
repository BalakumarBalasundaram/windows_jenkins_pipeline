# windows_jenkins_pipeline

# Syntax to call the Jenkins pipeline from workflow
def job = build job: 'say-hello', parameters: [[$class: 'StringParameterValue', name: 'who', value: 'DZone Readers']]

pipeline {
    agent any

    parameters {
        booleanParam(defaultValue: true, description: '', name: 'booleanExample')
        string(defaultValue: "TEST", description: 'What environment?', name: 'stringExample')
        text(defaultValue: "This is a multiline\n text", description: "Multiline Text", name: "textExample")
        choice(choices: 'US-EAST-1\nUS-WEST-2', description: 'What AWS region?', name: 'choiceExample')
        password(defaultValue: "Password", description: "Password Parameter", name: "passwordExample")
    }

    stages {
        stage("my stage") {
            steps {
                echo "booleanExample: ${params.booleanExample}"
                echo "stringExample: ${params.stringExample}"
                echo "textExample: ${params.textExample}"
                echo "choiceExample: ${params.choiceExample}"
                echo "passwordExample: ${params.passwordExample}"
            }
        }
    }
}


# Other links
https://blog.sonatype.com/continuous-integration-in-pipeline-as-code-environment-with-jenkins-jacoco-nexus-and-sonarqube

https://github.com/amarsingh3d/jenkins-pipeline
