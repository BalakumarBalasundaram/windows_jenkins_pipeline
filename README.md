# windows_jenkins_pipeline

DEPLOY TO PROD CHOICE VARIABLE HANDLING

https://github.com/davsuapas/DanceSchool-CloudPipeline/blob/e6fe6facea22b8eaa37796bb3d48014d73b338fa/declarative-pipeline/src/main/resources/Jenkinsfile-sample

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




def repositoryUrl = "https://github.com/MNT-Lab/mntlab-pipeline.git"
    def branch = "ilakhtenkov"
    def groupId = "com/epam/mntlab/pipeline"
    def artifactId = "gradle-simple"
    def version = "${env.BUILD_NUMBER}"
    def nexusServer = "http://10.6.205.59:8081"
    def repository = "Artifact_storage"
    def GRADLE_HOME = tool name: 'gradle3.3', type: 'hudson.plugins.gradle.GradleInstallation'

    stage('PREPARATION') {
        try {
            step([$class: 'WsCleanup'])
            git branch: branch, url: repositoryUrl
        }
        catch (Exception error){
            println ("PREPARATION Failed")
            postToSlack ("${env.BUILD_TAG} PREPARATION Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('BUILD') {
        try {
            sh "${GRADLE_HOME}/bin/gradle build"
        }
        catch (Exception error){
            println("BUILD Failed")
            postToSlack ("${env.BUILD_TAG} BUILD Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('TEST') {
        try {
            parallel junitTest: {
                sh "${GRADLE_HOME}/bin/gradle test"
            }, jacocoTest: {
                sh "${GRADLE_HOME}/bin/gradle jacocoTestReport"
            }, cucumberTest: {
                sh "${GRADLE_HOME}/bin/gradle cucumber"
            }
        }
        catch (Exception error){
            println("TEST Failed")
            postToSlack ("${env.BUILD_TAG} TEST Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('TRIGGER-CHILD') {
        try {
            build job: 'EPBYMINW2033/MNTLAB-ilakhtenkov-child1-build-job', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: branch]], wait: true
            copyArtifacts(projectName: 'EPBYMINW2033/MNTLAB-ilakhtenkov-child1-build-job', filter: '*_dsl_script.tar.gz')
        }
        catch (Exception error){
            println("TRIGGER-CHILD Failed")
            postToSlack ("${env.BUILD_TAG} TRIGGER-CHILD Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('PUBLISHING-RESULTS') {
        try {
            sh "tar -xzf  ${branch}_dsl_script.tar.gz"
            sh "tar -czf  pipeline-${branch}-${version}.tar.gz ./dsl/dsl_script.groovy ./ilakhtenkov.jenkinsfile.groovy ./build/libs/gradle-simple.jar"
            archiveArtifacts "pipeline-${branch}-*.tar.gz"
            sh "curl -v --user 'jenkins:jenkins' --upload-file './pipeline-${branch}-${version}.tar.gz' '${nexusServer}/repository/${repository}/${groupId}/${artifactId}-${version}/${version}/${artifactId}-${version}-${version}.tar.gz'"
            //sh "curl -v --user 'jenkins:jenkins' --upload-file './pipeline-${branch}-${env.BUILD_NUMBER}.tar.gz' 'http://nexus.local/repository/Artifact_storage/com/epam/mntlab/pipeline/gradle-simple-${env.BUILD_NUMBER}/${env.BUILD_NUMBER}/gradle-simple-${env.BUILD_NUMBER}-${env.BUILD_NUMBER}.tar.gz'"
        }
        catch (Exception error){
            println("PUBLISHING-RESULTS Failed")
            postToSlack ("${env.BUILD_TAG} PUBLISHING-RESULTS Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('APPROVAL') {
        try {
            input message: 'Do you want to deploy?', ok: 'Yes'
        }
        catch (Exception error){
            println("APPROVAL Failed")
            postToSlack ("${env.BUILD_TAG} APPROVAL Failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('DEPLOYING') {
        try {
            sh "java -jar build/libs/${artifactId}.jar"
            }
        catch (Exception error){
            println("DEPLOYING Failed")
            postToSlack ("${env.BUILD_TAG} DEPLOYING failed. See details:${env.BUILD_URL}")
            throw error
        }
    }
    stage('STATUS') {
        println "SUCCESS"
        postToSlack ("${env.BUILD_TAG} Successfully deployed. See details:${env.BUILD_URL}")
    }
}
def postToSlack (String message) {
    def webhookUrl = "https://hooks.slack.com/services/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"
    def channel = "#general"
    def userName = "bot.ilakhtenkov"
    sh "curl -X POST --data-urlencode \'payload={\"channel\": \"${channel}\", \"username\": \"${userName}\", \"text\": \"${message}\", \"icon_emoji\": \":chicken:\"}\' \"${webhookUrl}\""
}
