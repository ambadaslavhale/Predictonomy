node() {
	 def app
	 def buildID
	 def appRepoName="predictonomy/repo"
	 def repoURI="068478564052.dkr.ecr.eu-west-2.amazonaws.com"
	 try {
  stage('Checkout'){
  checkout scm  
  echo "build id - ${env.BUILD_ID}"
  echo 'env.PATH=' + env.PATH
	}
 
 echo "out of checkout"
 
  stage('build'){
  		echo "in build stage"
  		app = docker.build("${appRepoName}")
   		echo "docker build succeeded!!!"
  	}
  echo "out of build stage" 
  
  stage('Docker push'){
   /* First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
         docker.withRegistry("https://${repoURI}", 'ecr:eu-west-2:068478564052') {
         echo "ecr registration success!!!"
         app.push("${env.BUILD_ID}")
         buildID = "${env.BUILD_ID}"
         }
   }
  
  stage('Docker run'){
   echo "inside steps of docker run"
   sh 'docker ps -a -q; echo $? > status'
   def containersActive = readFile('status').trim()
	echo "active containers currently installed on ec2 : ${containersActive.size()}"
	if (containersActive.size()>0){
	sh (
   script: """\
   docker rm -f || true \
   """,
   )}
	
   echo "ran sh command to remove docker containers"
   echo "in docker run now with docker image = ${app}"
   echo "this is the build id = ${env.BUILD_ID}"
   /*def container=app.withRun('-it --name predictainer -p 80:80'){}
   def imageID=app.id
   echo "this is imageID = ${imageID}"
   def container=sh (
   script: 'docker run -it --name predictainer-"${env.BUILD_ID}" -p 80:80 "${app}"',
   returnStdout: true
	).trim()
   echo "docker ran with container created name ${container}"*/
    sh (
   script: "docker run -d --name predictainer-\"${env.BUILD_ID}\" -p 80:80 \"${repoURI}\"/\"${appRepoName}\":\"${env.BUILD_ID}\"",
  	)
	echo "container created"
   echo "THE END-------------------------"
   }
   
 }
    catch (err) {

        currentBuild.result = "FAILURE"

            mail body: "project build error is here: ${env.BUILD_URL}" ,
            from: 'xxxx@yyyy.com',
            replyTo: 'yyyy@yyyy.com',
            subject: 'project build failed',
            to: 'zzzz@yyyyy.com'

        throw err
    }
  
  
}