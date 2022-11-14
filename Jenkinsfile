pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "zee4859/app"
        registrycredential = 'dockerhub'
      
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        // stage('UNIT TEST'){
        //     steps {
        //         sh 'mvn test'
        //     }
        // }

        // stage('INTEGRATION TEST'){
        //     steps {
        //         sh 'mvn verify -DskipUnitTests'
        //     }
        // }

        // stage ('CODE ANALYSIS WITH CHECKSTYLE'){
        //     steps {
        //         sh 'mvn checkstyle:checkstyle'
        //     }
        //     post {
        //         success {
        //             echo 'Generated Analysis Result'
        //         }
        //     }
        // }

        // stage('CODE ANALYSIS with SONARQUBE') {

        //     environment {
        //         scannerHome = tool 'mysonarscanner4'
        //     }

        //     steps {
        //         withSonarQubeEnv('sonar-pro') {
        //             sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //            -Dsonar.projectName=vprofile-repo \
        //            -Dsonar.projectVersion=1.0 \
        //            -Dsonar.sources=src/ \
        //            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //            -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        //         }

        //         timeout(time: 10, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        stage('Buid App Image'){
            steps{
                script {
                    dockerImage = docker.build registry + ":v$BUILD_NUMBER"
                    
                }
            }
        }

        stage('Upload Image'){
            steps{
                script{
                    docker.withRegistry( '', registrycredential ) {
                    dockerImage.push("v$BUILD_NUMBER")
                    //dockerImage.push('latest')
                    }
                }
            }
        }

        // stage('Upload Imagebnnnnn') {
        //   steps{
        //     script {
        //       docker.withRegistry( '', registryCredential ) {
        //         dockerImage.push("$BUILD_NUMBER")
        //         dockerImage.push('latest')
        //       }
        //     }
        //   }
        // }

        stage('Remove Unused Image'){
            steps{
                sh "docker rm $registry:v$BUILD_NUMBER"
            }
        }
        stage('K8s Deploy'){
            agent {label 'KOPS'}
                steps {
                    sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:$v{BUILD_NUMBER} --namespace prod"
                }
        }

    }

}