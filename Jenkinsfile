node {
    def server = Artifactory.server 'ART8080GCP'
	def rtMaven = Artifactory.newMavenBuild()
	def artDocker = Artifactory.docker server: server, host: "tcp://localhost:2375"
	def image = 'docker.artifactory.bruce/onboard/hello'
	def buildImage = image + ":" + env.BUILD_NUMBER
	def buildInfo
	
    stage('Checkout') {
    		// Get some code from a GitHub repository
		git 'https://github.com/brucefrog/JettyWorld'
    }
    stage('Package') {
		// Setup Artifactory resolution
		rtMaven.tool = 'Maven3.5.2'
		rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
		buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean package' 
		buildInfo.env.capture = true
		buildInfo.retention maxBuilds: 10
		// rtMaven.deployer.deployArtifacts buildInfo
		// server.publishBuildInfo buildInfo
    }
    stage('Docker Image') {
    		sh 'printenv'
		def dockerImage = docker.build(buildImage)
		dockerImage.tag("latest")
		// dockerImage.push() 
		def dockInfo = artDocker.push buildImage, 'docker', buildInfo 
		// dockerImage.push("latest")
		artDocker.push image+":latest", 'docker'
		// buildInfo.append dockerInfo
		server.publishBuildInfo(buildInfo)
    }
    stage('Xray Scan') {
          def xrayConfig = [
            //Mandatory parameters
            'buildName'         : env.JOB_NAME,
            'buildNumber'       : env.BUILD_NUMBER,

            //Optional
            'failBuild'        : false
          ]

          // Scan xray build
          def xrayResults = server.xrayScan xrayConfig
          // Print full report from xray
          echo xrayResults as String
    }
    stage('Verify Image') {
        sh 'docker rmi ' + image
        sh 'docker rmi ' + buildImage

    		docker.image(buildImage).withRun('-p 6800:6800') {c ->
                sleep 5
                sh 'curl "http://localhost:6800/"'
        }
        sh 'docker rmi ' + buildImage
    }
}