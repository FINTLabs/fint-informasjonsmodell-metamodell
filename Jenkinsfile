pipeline {
  agent none
  environment {
    GITHUB = credentials('github_fint_jenkins')
  }
  stages {
    stage('Generate Model') {
      agent { 
        label 'docker'
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        sh "docker run -i -v \$PWD:/src fint/fint-model:2.2.0-alpha-1 --repo fint-informasjonsmodell-metamodell --filename FINT-metamodell.xml --tag ${TAG_NAME} generate --resource"
        stash(name: 'java', includes: 'java/**')
      }
    }
    stage('Commit Java Model') {
      agent { 
        docker {
          label 'docker'
          image 'fint/git:latest'
        }
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        script {
          VERSION = TAG_NAME[1..-1]
        }
        git 'https://github.com/FINTmodels/fint-metamodell-model-java.git'
        sh 'git clean -fdx'
        unstash 'java'
        sh 'rm -rf src/main/java/no/fint/model/*'
        sh 'mv java/* src/main/java/no/fint/model/'
        sh "echo version=${VERSION} > gradle.properties"
        sh 'git config user.email "jenkins@fintlabs.no"'
        sh 'git config user.name "FINT Jenkins"'
        sh 'git add gradle.properties src/main/java/no/fint/model/'
        sh "git commit -m 'Version ${VERSION}'"
        sh "git push 'https://${GITHUB}@github.com/FINTmodels/fint-metamodell-model-java.git' master:master"
      }
    }
    stage('Generate Consumer') {
      agent { 
        label 'docker'
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        sh "docker run -i -v \$PWD:/src fint/fint-consumer:2.2.0-rc-1 --repo fint-informasjonsmodell-metamodell --filename FINT-metamodell.xml --tag ${TAG_NAME} setup --name metamodell --component metamodell --branch newcache"
        stash(name: 'consumer', includes: 'fint-consumer-metamodell/**')
      }      
    }
    stage('Commit Consumer') {
      agent { 
        docker {
          label 'docker'
          image 'fint/git:latest'
        }
      }
      when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+(-\\w+-\\d+)?", comparator: "REGEXP"
      }
      steps {
        script {
          VERSION = TAG_NAME[1..-1]
        }
        git 'https://github.com/FINTLabs/fint-consumer-metamodell.git'
        sh 'git clean -fdx'
        sh 'git config user.email "jenkins@fintlabs.no"'
        sh 'git config user.name "FINT Jenkins"'
        unstash 'consumer'
        sh 'cp -r fint-consumer-metamodell/* . && rm -r fint-consumer-metamodell'
        sh 'sed -i -e "s/-resource-model-/-model-/" build.gradle'
        sh 'git add .'
        sh "git commit -m 'Version ${VERSION}'"
        sh "git push 'https://${GITHUB}@github.com/FINTLabs/fint-consumer-metamodell.git' master:master"
      }      
    }
  }
}
