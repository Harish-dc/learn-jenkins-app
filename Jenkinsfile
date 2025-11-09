pipeline {
  agent any 

  environment {
    NETLIFY_SITE_ID= '9f041764-fdd5-4c51-87fc-d109408cc98e'
    NETLIFY_AUTH_TOKEN= credentials('netlify-token')
  }
  stages{
      stage('Build'){
          agent {
              docker {
                image 'node:18-alpine'
                reuseNode true
              }
          }
          steps {
              sh '''
                ls -la
                npm --version
                node --version
                npm ci
                npm run build
                ls -la
              '''
          }
      }

      stage("deploy") {
        parallel {
            stage('unit test') {
                agent{
                    docker {
                      image "node:18-alpine"
                      reuseNode true

                    }
                }
                steps{
                    sh '''
                        test -f build/index.html
                        npm test
                    '''
                } 
                post{
                  always {
                      junit 'jest-results/junit.xml'
                  }
                }
            }

            stage('E2E') {
                agent{
                    docker {
                      image "mcr.microsoft.com/playwright:v1.39.0-jammy"
                      reuseNode true

                    }
                }
                steps{
                    sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10                
                        npx playwright test --reporter=html
                    '''
                } 
                post{
                  always {
                      publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Report', reportTitles: '', useWrapperFileDirectly: true])
                  }
                }
            }
        }
      }

          agent {
              docker {
                image 'node:18-alpine'
                reuseNode true
              }
          }
          steps {
              sh '''
              npm install netlify-cli@20.1.1

              node_modules/.bin/netlify --version
              echo "deploying  to netlify site id: $NETLIFY_SITE_ID"
              node_modules/.bin/netflix status
              node_modules/.bin/netflix deploy --dir=build --prod
              '''
          }
      }
  }
