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
                    not { branch 'main' }
               }               
               steps {
                    echo 'Unit test not main branch'
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
                               
               steps {
                    echo 'Code coverage only main branch'
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis not main branch") {
               when {
                    not { branch 'main' }
               }                 
               steps {
                    echo 'Static code analysis'
                    sh "./gradlew checkstyleMain"
               }
          }
     }
}