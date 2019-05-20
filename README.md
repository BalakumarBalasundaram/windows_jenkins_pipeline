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

pipeline {
    agent any
    triggers {
        cron('H */4 * * 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}

pipeline {
    agent any
    parameters {
      string(name: 'PLANET', defaultValue: 'Earth', description: 'Which planet are we on?')
      string(name: 'GREETING', defaultValue: 'Hello', description: 'How shall we greet?')
    }
    triggers {
        cron('* * * * *')
        parameterizedCron('''
# leave spaces where you want them around the parameters. They'll be trimmed.
# we let the build run with the default name
*/2 * * * * %GREETING=Hola;PLANET=Pluto
*/3 * * * * %PLANET=Mars
        ''')
    }
    stages {
        stage('Example') {
            steps {
                echo "${GREETING} ${PLANET}"
                script { currentBuild.description = "${GREETING} ${PLANET}" }
            }
        }
    }
}


# Other links
https://blog.sonatype.com/continuous-integration-in-pipeline-as-code-environment-with-jenkins-jacoco-nexus-and-sonarqube

https://github.com/amarsingh3d/jenkins-pipeline
