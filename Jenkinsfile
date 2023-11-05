pipeline {
  agent any

  stages {
      stage('Build Artifact') {
          steps {
            sh "mvn clean package -DskipTests=true"
            archive 'target/*.jar' //so that they can be downloaded later time
          }
      }
      stage('Unit Tests') {
          steps {
            sh "mvn test"
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
      }

      stage('Mutation Tests - PIT') {
          steps {
            sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
          post {
            always {
                pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
          }
      }

      stage('SonarQube - SAST') {
          steps {
            mvn clean verify sonar:sonar \
              -Dsonar.projectKey=numeric-application \
              -Dsonar.projectName='numeric-application' \
              -Dsonar.host.url=http://devsecops-weyew.eastus.cloudapp.azure.com:9000 \
              -Dsonar.token=sqp_d2e9b1019467ef0f499530bfff90465679a2d059
          }
      }

      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'docker build -t samishken/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push samishken/numeric-app:""$GIT_COMMIT""'
          }
        }
      }

      stage('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
            echo env.GIT_COMMIT
            sh "sed -i 's#replace#samishken/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }
        }
      }
    }
}
