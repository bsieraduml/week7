podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:8-jdk8
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt        
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    stage('Build a gradle project') {
      git branch: 'main', url: 'https://github.com/bsieraduml/week6.git'
      container('gradle') {
        stage('Build a gradle project') {
          sh '''
          pwd
          echo 'java version'
          java -version
          echo 'javac version'
          javac -version
          echo 'call chmod +x gradlew'
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
        }

        stage("Code coverage") {
            //playground runs no tests, feature runs all tests except Code Coverage
            if (env.BRANCH_NAME != 'feature' && env.BRANCH_NAME != 'playground') {
              try {
                sh '''
                ./gradlew jacocoTestCoverageVerification
                ./gradlew jacocoTestReport
                  '''
              } catch (Exception E) {
                  echo 'Failure detected'
              }

              // from the HTML publisher plugin
              // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
              publishHTML (target: [
                  reportDir: 'build/reports/tests/test',
                  reportFiles: 'index.html',
                  reportName: "JaCoCo Report"
              ])       
            } else {
              echo 'Skip Code coverage for branch ' + env.BRANCH_NAME
            }
        }  

        stage("Checkstyle") {
              //playground runs no tests, feature runs all tests except Code Coverage
              if (env.BRANCH_NAME != 'playground') {               
                try {
                    sh '''
                  chmod +x gradlew    
                  ./gradlew checkstyleMain
                  ./gradlew jacocoTestReport
                    '''
                } catch (Exception E) {
                    echo 'Failure detected'
                }

                // from the HTML publisher plugin
                // https://www.jenkins.io/doc/pipeline/steps/htmlpublisher/
                publishHTML (target: [
                    reportDir: 'build/reports/checkstyle',
                    reportFiles: 'main.html',
                    reportName: "JaCoCo Checkstyle"
                ]) 
              } else {
                echo 'Skip Checkstyle for branch ' + env.BRANCH_NAME 
              }                      
          }   

      }
    }



                

    stage('Build Java Image') {
      //only if the build succeeds; never for the playground branch
      if (env.BRANCH_NAME != 'playground') {
        container('kaniko') {
          stage('Build a gradle project') {
            script {
              tagName = 'bsieraduml/calculator'
              if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'release') {
                  tagName = 'bsieraduml/calculator:1.0'
              } else if (env.BRANCH_NAME == 'feature') {
                  tagName = 'bsieraduml/calculator-feature:0.1'
              }
              else 
              {
                error "Unsupported Branch Name = " + env.BRANCH_NAME  
              }
              echo 'tagName=' + tagName
            }
            sh '''
            echo 'Creating Docker Container...'
            echo 'FROM openjdk:8-jre' > Dockerfile
            echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
            echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
            mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
            /kaniko/executor --context `pwd` --destination ${tagName}
            '''
          }
        }
      } else {
        echo 'Not running Build Java Image for branch => ' + env.BRANCH_NAME 
      }
      
    }

  }
}