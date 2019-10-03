#!/usr/bin/env groovy

node {
  stage('Checkout Code') {
    checkout scm
  }
  stage('Fortify Scan') {
    fortifyScan()
  }
  stage('Translate Results') {
    translateResults()
  }
  stage('Sonarqube Analysis') {
    sonarqubeScan()
  }
}

def fortifyScan() {
  def pom = readMavenPom()
  def plugin = "com.hpe.security.fortify.maven.plugin:sca-maven-plugin:17.10"
  String filename = "${pom.getArtifactId}-${pom.getVersion()}"
  docker.image('maven-fortify:1.0').inside() {
      sh "mvn -B ${plugin}:clean -Duser.home=/home/jenkins "
      sh "mvn -B ${plugin}:translate -DskipTests -Duser.home=/home/jenkins"
      sh "mvn -B ${plugin}:scan -Dfortify.sca.Xmx=8G -Duser.home=/home/jenkins"
      sh "FPRUtility -information -listIssues -project target/fortify/${filename}.fpr -outputFormat CSV -f target/fortify/fprcsv.csv"
  }
}

def translateResults() {
  docker.image('fortify-to-sonarqube:1.0').inside() {
    sh 'transform --input target/fortify/fprcsv.csv --output target/fortify/fprresults.json'
  }
}

def sonarqubeScan() {
  withEnv(['MAVEN_HOME=/usr/share/maven', 'JAVA_HOME=/usr/local/openjdk-8']) {
    docker.image('maven:3.6.2-jdk-8').inside() {
      withSonarQubeEnv('sonarqube') {
        sh "mvn -B sonar:sonar -Duser.home=/home/jenkins -Dsonar.externalIssuesReportPaths=target/fortify/fprresults.json"
      }
    }
  }
}
