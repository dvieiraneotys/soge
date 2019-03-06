pipeline {
  agent none
  stages {
    stage('Launch Infrastructure') {
      parallel{
        stage('Launch application'){
          agent { label 'master'}
          steps {
            git(branch:'master',
              url:'https://github.com/microservices-demo/microservices-demo')
            sh 'docker-compose -f deploy/docker-compose/docker-compose.yml up -d'
          }
        }
        stage('Start NeoLoad infrastructure') {
          agent { label 'master' }
          steps {
            sh 'docker-compose -f neoload/lg/docker-compose.yml up -d'
            stash includes: 'neoload/lg/lg.yaml', name: 'LG'
            stash includes: 'neoload/test/microservices.yaml', name: 'microservices'
            stash includes: 'neoload/test/data/tokens.csv', name: 'tokens'
          }
        }
      }
    }
    stage('Join Load Generators to Application') {
      agent{label 'master'}
      steps{
        sh 'docker network connect docker-compose_default docker-lg1'
      }

    }
    stage('API Tests') {
      agent {
        dockerfile {
          args '--user root -v /tmp:/tmp --network=lg_default'
          dir 'neoload/controller'
        }
      }
      steps {
        git(branch: "master",
            url: 'http://jenkins:9090/git/tester/BaseProject.git')
        unstash 'LG'
        unstash 'microservices'
        unstash 'tokens'
        script {
          neoloadRun executable: '/home/neoload/neoload/bin/NeoLoadCmd',
            project: "$WORKSPACE/BaseProject.nlp",
            testName: 'Catalogue Limit Test (build ${BUILD_NUMBER})',
            testDescription: 'Baseline catalogue limit test on build',
            commandLineOption: "-project $WORKSPACE/neoload/test/microservices.yaml -nlweb -loadGenerators $WORKSPACE/neoload/lg/lg.yaml -nlwebToken a8e8f0c5a4f90c02bfddcb6881e7f6811da26864879a7bd6",
            scenario: 'CatalogueLimit', sharedLicense: [server: 'NeoLoad Demo License', duration: 2, vuCount: 200],
            trendGraphs: [
                [name: 'API Response time', curve: ['CatalogueList>Actions>Get Catalogue List'], statistic: 'average'],
                'ErrorRate'
                ]
        }
        script {
          neoloadRun executable: '/home/neoload/neoload/bin/NeoLoadCmd',
            project: "$WORKSPACE/BaseProject.nlp",
            testName: 'Catalogue Standard Test (build ${BUILD_NUMBER})',
            testDescription: 'Baseline catalogue load test on build',
            commandLineOption: "-project $WORKSPACE/neoload/test/microservices.yaml -nlweb -loadGenerators $WORKSPACE/neoload/lg/lg.yaml -nlwebToken a8e8f0c5a4f90c02bfddcb6881e7f6811da26864879a7bd6",
            scenario: 'CatalogueStandard', sharedLicense: [server: 'NeoLoad Demo License', duration: 2, vuCount: 50],
            trendGraphs: [
                [name: 'API Response time', curve: ['CatalogueList>Actions>Get Catalogue List'], statistic: 'average'],
                'ErrorRate'
                ]
        }
      }
    }
    stage('Stop Infrastructure') {
      parallel{
        stage('Stop NeoLoad infrastructure') {
          agent { label 'master' }
          steps {
            sh 'docker-compose -f neoload/lg/docker-compose.yml down'
          }
        }
        stage('Stop application'){
          agent { label 'master'}
          steps {
            git(branch:'master',
              url:'https://github.com/microservices-demo/microservices-demo')
            sh 'docker-compose -f deploy/docker-compose/docker-compose.yml down'
          }
        }
      }
    }

  }
}
