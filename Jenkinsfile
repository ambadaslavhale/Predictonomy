node() {
	 def app
	 def appRepoName="predictonomy/repo"
	 def repoURI="068478564052.dkr.ecr.eu-west-2.amazonaws.com"
	 def containersActive 
	 try {
  stage('Checkout'){
  checkout scm  
  echo "build id - ${env.BUILD_ID}"
  echo 'env.PATH=' + env.PATH
	}
 
 echo "out of checkout"
 
  stage('build'){
	  	echo "removing existing images first"
	  	sh (
	    """\
	    script: "docker rmi \"${repoURI}\"/\"${appRepoName}\":\"${env.BUILD_ID}\"",
	    """,
	  	)
	  	echo "images removed now onto building a new image"
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
         }
   }
  
  stage('Docker run'){
   /* First remove the existing containers but set the command to true so that even if
   there are no containers running or existing the error returned by docker rm command
   doesnt stop the pipeline process from running further */
   sh (
   script: 
   """\
   docker ps -qa | xargs docker rm -f || true\
   """,
   )
   echo "in docker run now with docker image = ${app}"
   echo "this is the build id = ${env.BUILD_ID}"
   sh (
   script: "docker run -d --name predictainer-\"${env.BUILD_ID}\" -p 80:80 \"${repoURI}\"/\"${appRepoName}\":\"${env.BUILD_ID}\"",
   	)
   	echo "now that the image is running in container, we should remove the image"
   sh (
	    script: "docker rmi \"${repoURI}\"/\"${appRepoName}\":\"${env.BUILD_ID}\"",
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