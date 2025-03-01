pipeline{
    agent any
    environment{
        NETLIFY_SITE_ID = '3e365465-c289-4b6c-bbe2-8d21adc2eadb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }
    stages{
        // Comment
        /* 
            multi line comment
        */
        stage('Docker'){
            steps{
                sh '''
                    docker build -t my-playwright .
                '''
            }
        }
        stage('Build'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh '''
                # comment
                    echo 'small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la                    
                '''
            }
        }
        stage('Unit Tests'){
            parallel{
                stage('Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            echo "test stage"
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
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
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        // stage('Deploy Staging'){
        //     agent{
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         sh '''
        //             npm install netlify-cli node-jq
        //             node_modules/.bin/netlify --version
        //             echo "Deploying to Prod with SiteID - $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //             node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
        //         '''
        //         script{
        //             env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //         }
        //     }
            
        // }

        stage('Deploy Staging and Test E2E'){
            environment{
                CI_ENVIRONMENT_URL = "PASS"
            }
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    netlify --version
                    echo "Deploying to Prod with SiteID - $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Deploy Prod'){
        //     agent{
        //         docker{
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         sh '''
        //             npm install netlify-cli
        //             node_modules/.bin/netlify --version
        //             echo "Deploying to Prod with SiteID - $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --prod
        //         '''
        //     }
        // }

        stage('Deploy Prod and Test E2E'){
            environment{
                CI_ENVIRONMENT_URL = 'https://whimsical-paletas-c5f18a.netlify.app'
            }
            agent{
                docker{
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to Prod with SiteID - $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
    
}