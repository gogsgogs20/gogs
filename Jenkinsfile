node(){
def root = tool name: '1.8.7', type: 'go'
def scannerHome = tool 'sonar';
try{
notifyBuild()
    
stage('Checkout'){
    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'src/github.com/gogs/gogs/']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/gogsgogs20/gogs.git']]])
}

stage('SonarQube analysis') {

    withSonarQubeEnv('sonar'){
        sh "${scannerHome}/bin/sonar-scanner -Dsonar.host.url=http://10.26.34.152:32128 -Dsonar.projectName=Gogsproject -Dsonar.projectVersion=1.0 -Dsonar.projectKey=gogsproject -Dsonar.sources=./src/github.com/gogs/gogs -Dsonar.projectBaseDir=/var/jenkins_home/workspace/pipeline -Dsonar.analysis.mode=publish"
    }
    }

/*
stage("Quality Gate"){

   sleep(20)
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
   if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"}

    }

stage('Test'){
    withEnv(["GOROOT=${root}", "GOPATH=${WORKSPACE}", "PATH+GO=${root}/bin", "GOGS_SRC=${WORKSPACE}/src/github.com/gogs/gogs"]) {

    sh 'cd $GOGS_SRC && make govet'
    sh 'cd $GOGS_SRC && make test'
    
}
}

*/
stage('Build'){

    withEnv(["GOROOT=${root}", "GOPATH=${WORKSPACE}", "PATH+GO=${root}/bin", "OUTPUT_PATH=${WORKSPACE}/artifacts/build${BUILD_ID}", "GOGS_SRC=${WORKSPACE}/src/github.com/gogs/gogs"]) {
        sh 'cd $GOGS_SRC && make '
        
    }

}

stage('Release'){
    
    withEnv(["GOROOT=${root}", "GOPATH=${WORKSPACE}", "PATH+GO=${root}/bin", "OUTPUT_PATH=${WORKSPACE}/artifacts/build${BUILD_ID}", "GOGS_SRC=${WORKSPACE}/src/github.com/gogs/gogs"]) {
    
    timeout(time:5, unit:'DAYS') 
    {
        //sh 'cd $GOGS_SRC && ./gogs web'
        input message:'Approve deployment?', submitter: 'it-ops'
        
    }
    sh 'cd $GOGS_SRC/scripts && ./build.sh'
    sh 'mkdir -p artifacts'
    sh 'mv $GOGS_SRC/scripts/output $OUTPUT_PATH'
    sh 'cp $GOGS_SRC/scripts $OUTPUT_PATH -r'
    sh 'cd $WORKSPACE/artifacts && tar -cvf build$BUILD_ID.tar.gz build$BUILD_ID'
    }
    sh 'curl -v -u admin:g6JZ7AMX --upload-file $WORKSPACE/artifacts/build$BUILD_ID.tar.gz http://10.26.34.133:8081/repository/GoArtifacts/release/$BUILD_ID/build$BUILD_ID.tar.gz'
}

}
finally{}
    
}
def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
    env.TOKEN="905574275:AAGAU_FYPvoaHFIV4s1GPEYEWuFG_HXQVRw"
    env.CHAT_ID="682576870"
    env.MESSAGE=buildStatus
    env.API_URL="https://api.telegram.org/bot$TOKEN/sendMessage"

    sh '''curl -s -X POST $API_URL -d chat_id=$CHAT_ID -d text="$MESSAGE"'''

}
    
