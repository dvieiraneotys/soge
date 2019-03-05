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
            stash includes: 'neoload/test/demotest.yaml', name: 'scenario'
            stash includes: 'neoload/test/data/tokens.csv', name: 'tokens'
          }
        }
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
        unstash 'scenario'
        unstash 'tokens'
        script {
          neoloadRun executable: '/home/neoload/neoload/bin/NeoLoadCmd',
            project: "$WORKSPACE/BaseProject.nlp",
            testName: 'Jenkins Test (build ${BUILD_NUMBER})',
            testDescription: 'NeoLoad as Code test',
            commandLineOption: "-project $WORKSPACE/neoload/test/demotest.yaml -nlweb -loadGenerators $WORKSPACE/neoload/lg/lg.yaml -nlwebToken a8e8f0c5a4f90c02bfddcb6881e7f6811da26864879a7bd6",
            scenario: 'DemoScenario', sharedLicense: [server: 'NeoLoad Demo License', duration: 2, vuCount: 51],
            trendGraphs: [
                [name: 'API Response time', curve: ['Component Testing_REST>Actions>API 10 calls>REST API call'], statistic: 'average'],
                [name: 'MySQL Response time (Select a post)', curve: ['Component Testing_MySQL>Actions>MySQL'], statistic: 'average'],
                'ErrorRate'
                ]
          }
   /*       script {
            neoloadRun executable: '/home/neoload/neoload/bin/NeoLoadCmd',
              project: "$WORKSPACE/CPVWeatherCrisis.nlp",
              testName: 'API Nominal Test (build ${BUILD_NUMBER})',
              testDescription: 'WeatherCrisis API Nomonal Testing (Mysql + Rest API)',
              commandLineOption: "-project $WORKSPACE/neoload/test/$CPV_ENV/scenario.yaml -nlweb -L API=$LG $APM_LG -nlwebToken <NeoLoad Web Token>",
              scenario: 'API Nominal Test', sharedLicense: [server: 'NeoLoad Demo License', duration: 2, vuCount: 51],
              trendGraphs: [
                  [name: 'API Response time', curve: ['Component Testing_REST>Actions>API 10 calls>REST API call'], statistic: 'average'],
                  [name: 'MySQL Response time (Select a post)', curve: ['Component Testing_MySQL>Actions>MySQL'], statistic: 'average'],
                  'ErrorRate'
                  ]
            }*/
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
