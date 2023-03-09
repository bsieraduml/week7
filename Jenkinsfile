pipeline {
     agent any
     triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               steps {
                    git branch: 'main', url: 'https://github.com/bsieraduml/week6.git'
                    echo 'Compile all branches'
                    sh "chmod +x gradlew"
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               when {
                    not { branch 'playground' }
               }               
               steps {
                    echo 'Unit test => branch' + env.BRANCH_NAME
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
               when {
                    not {
                         anyOf {
                              branch 'playground';
                              branch 'feature'
                         }
                    }
               }            
               steps {
                    echo 'Code coverage => branch' + env.BRANCH_NAME
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis not main branch") {
               when {
                    not { branch 'playground' }
               }                    
               steps {
                    echo 'Static code analysis=> branch' + env.BRANCH_NAME
                    sh "./gradlew checkstyleMain"
               }
          }
     }
}