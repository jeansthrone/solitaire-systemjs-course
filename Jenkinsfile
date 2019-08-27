node {
    checkout scm
    stage('1') {
       
       // git branch: 'jenkins2-course', 
            url: 'https://github.com/jeansthrone/solitaire-systemjs-course'
    }
    // pull dependencies from npm
    // on windows use: bat 'npm install'
    stage('2') {
        
     bat 'npm install'
    
    
        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything', 
              excludes: 'test-results/**', 
              includes: '**'
        
            // test with PhantomJS for "fast" "generic" results
            // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
            bat 'npm run test-single-run -- --browsers PhantomJS'
       
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
              
    }      
}

node('wind'){
    
    bat 'dir'
    bat 'del /S /Q *'
    unstash 'everything'
    bat 'dir'
}

// Parallel Integration testing

stage(' Browser Testing'){
    
    parallel chrome: {
        runTest("Chrome");
    }
    
    
}

def runTest(browser){
    node {
        bat 'del /S /Q *'
        unstash 'everything'
        bat "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }
}

node{
    notify('Deploy to staging?')
}

input 'Deploy to staging?'

// Limit concurrency so we don´t perform simultaneous deploys
// and if multiple pipelines are executing,
// newest is only that will be allowed throught, rest will be canceled
stage name: 'Deploy', concurrency: 1

node{
    // write build number to index page so we can see this update
    // on windows linux use: sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    bat 'echo <h1>${env.BUILD_DISPLAY_NAME}</h1> >> app/index.html'
    
    // deploy to a docker container mapped to port 3000
    // on windows use: sh 'docker-compose up -d --build'
    bat 'docker-compose up -d --build'
    
    notify 'Solitaire Deployed!'

}





def notify(status){
    emailext (
      to: "jeanron@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
