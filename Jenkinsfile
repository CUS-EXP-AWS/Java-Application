pipeline {

    agent any
    tools {
        maven 'maven-default'
        jdk 'jdk-1.8'
    }

    environment {
        JAVA_HOME = "${tool 'jdk-1.8'}"
        PATH = "${JAVA_HOME}/bin:${PATH}"

    }

    options {
        disableConcurrentBuilds()
    }

    parameters {
        choice(choices: 'SNAPSHOT\nRELEASE', description: 'What type of build?', name: 'BUILD')
    }

    stages {
        stage('clonecode') {
            steps {
                script {
                    deleteDir()
                    checkout scm
                    sh 'git checkout ${GIT_BRANCH}'
                    sh 'echo ${GIT_BRANCH}'
                }
            }
        }

        stage('package') {
            when {
                expression { params.BUILD == 'SNAPSHOT' }
            }
            steps {
                script {

               sh "mvn package -Dmaven.test.skip=true"                    

                }
            }
        }

        stage('snapshot') {
            when {
                expression { params.BUILD == 'SNAPSHOT' }
            }
            steps {
                script {
                    def server = Artifactory.server('artifacts-den06')
                    def rtMaven = Artifactory.newMavenBuild()
                    rtMaven.deployer server: server, releaseRepo: 'Customer-Experience-maven-prod', snapshotRepo: 'Customer-Experience-maven-dev'
                    def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install -DskipTests=true -Dmaven.test.skip=true -Dartifactory.publish.buildInfo=true'
                    server.publishBuildInfo buildInfo

                }
            }
        }



        stage('release') {
            when {
                expression { params.BUILD == 'RELEASE' }
            }
            steps {
                sshagent(credentials: ['scmaccess_ssh']) {
                    script {
                        sh 'git checkout ${GIT_BRANCH}'
                        def server = Artifactory.server('artifacts-den06')
                        def rtMaven = Artifactory.newMavenBuild()
                        rtMaven.deployer.deployArtifacts = false
                        rtMaven.deployer server: server, releaseRepo: 'Customer-Experience-maven-prod', snapshotRepo: 'Customer-Experience-maven-dev'
                        def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'release:clean release:prepare -Dmaven.javadoc.skip=true -Darguments="-Dmaven.test.skip=true -Dmaven.javadoc.skip=true" release:perform -Dmaven.javadoc.skip=true -Dmaven.test.skip=true -DskipTests=true -Dartifactory.publish.buildInfo=true -Dgoals=install,-DskipTests=true,-Dmaven.test.skip=true'
                        buildInfo = rtMaven.run pom: 'target/checkout/pom.xml', goals: 'clean install -DskipTests=true -Dmaven.test.skip=true'
                        rtMaven.deployer.deployArtifacts buildInfo
                        server.publishBuildInfo buildInfo

                    }

                }

            }

        }

    }
}
