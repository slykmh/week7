podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
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
      container('gradle') {
        git 'https://github.com/slykmh/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
        stage('Build a gradle project') {
          sh '''
          cd Chapter08/sample1
          chmod +x gradlew
          ./gradlew build
          mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
          '''
          stage("Feature Branch Testing") {
            if (env.BRANCH_NAME == 'feature') {
            echo "This is the ${env.BRANCH_NAME} branch"
                
                    try {
                        sh '''
                        pwd
                        cd Chapter08/sample1
                        chmod +x gradlew
                        ./gradlew checkstyleMain
                        ./gradlew test
                        ./gradlew jacocoTestReport '''
                        }
                    catch (Exception E) {
                        echo 'Feature branch test failure'
                        }
                }
          }
        
        stage("Playground Branch Testing"){
            if (env.BRANCH_NAME == 'playground') 
            {
                echo "Playground Branch has no testing"
            }
        }
            
        stage("Main Branch Testing") {
            if (env.BRANCH_NAME == 'main')
              {
                echo "This is the ${env.BRANCH_NAME} branch"
                    try {
                        sh '''
                        cd Chapter08/sample1
                        chmod +x gradlew
                        ./gradlew checkstyleMain
                        ./gradlew test
                        ./gradlew jacocoTestCoverageVerification
                        ./gradlew jacocoTestReport '''
                        }
                    catch (Exception E) {
                        echo 'Main test failure'
                        }
              }
        }
      }
    }
  }
      
    stage('Build Java Image') {
      container('kaniko') {
        stage('Kaniko Container Feature Branch'){
            if (env.BRANCH_NAME == 'feature'){
              stage('Build a gradle project') {
              sh '''
                echo 'FROM openjdk:8-jre' > Dockerfile
                echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
                echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
                mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
                /kaniko/executor --context `pwd` --destination slykmh/calculator-feature:0.1
                '''
            }
          }
        }
        
        stage('Kaniko Container Main Branch'){
          if (env.BRANCH_NAME == 'main'){
            stage('Build a gradle project') {
              sh '''
              echo 'FROM openjdk:8-jre' > Dockerfile
              echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
              echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
              mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
              /kaniko/executor --context `pwd` --destination slykmh/calculator:1.0
              '''
            }
          }
        }
      }
    }
  }
}
