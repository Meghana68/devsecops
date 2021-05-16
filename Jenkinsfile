
!groovy

import groovy.json.JsonSlurper
import java.net.URL

pipeline {
    agent none
    options {
        timeout(time: 1, unit: 'DAYS')
        disableConcurrentBuilds()
    }
    stages {
        stage("Init") {
            agent any
            steps { initialize() }
        }
        stage("Build App") {
            agent any 
            steps { buildApp() }
        }
/*        stage("Run App Security Scan") {
            agent any 
            steps {  runSecurityTest() }
        }
        stage("Build and Register Image") {
            agent any
            steps { buildAndRegisterDockerImage() }
        }
        stage("Deploy Image to Dev") {
            agent any
            steps { deployImage(env.ENVIRONMENT) }
        }
        stage("Test App in Dev") {
            agent any
            steps { runBrowserTest(env.ENVIRONMENT) }
        }
        stage("Deploy Image to Test") {
            agent any
            when { branch 'master' } 
            steps { deployImage('test') }
        }
        stage("Test App in Test") {
            agent any
            when { branch 'master' } 
            steps { runBrowserTest('test') }
        }
        stage("Proceed to prod?") {
            agent none
            when { branch 'master' } 
            steps { proceedTo('prod') }
        }
        stage("Deploy Image to Prod") {
            agent any
            when { branch 'master' }
            steps { deployImage('prod') }
        }*/
    }
}


// ================================================================================================
// Initialization steps
// ================================================================================================

def initialize() {
    env.SYSTEM_NAME = "DSO"
    env.AWS_REGION = "us-east-1"
    env.REGISTRY_URL = "https://912661153448.dkr.ecr.us-east-1.amazonaws.com"
    env.MAX_ENVIRONMENTNAME_LENGTH = 32
    setEnvironment()
    env.IMAGE_NAME = "hello-world:" + 
        ((env.BRANCH_NAME == "master") ? "" : "${env.ENVIRONMENT}-") + 
        env.BUILD_ID
    showEnvironmentVariables()
    }

def setEnvironment() {
    def branchName = env.BRANCH_NAME.toLowerCase()
    def environment = 'dev'
    echo "branchName = ${branchName}"
    if (branchName == "") {
        showEnvironmentVariables()
        throw "BRANCH_NAME is not an environment variable or is empty"
    } else if (branchName != "master") {
        //echo "split"
        if (branchName.contains("/")) {
            // ignore branch type
            branchName = branchName.split("/")[1]
        }
        //echo "remove '-' characters'"
        branchName = branchName.replace("-", "")
        //echo "remove JIRA project name"
        if (env.JIRA_PROJECT_NAME) {
            branchName = branchName.replace(env.JIRA_PROJECT_NAME, "")
        }
        // echo "limit length"
        branchName = branchName.take(env.MAX_ENVIRONMENTNAME_LENGTH as Integer)
        environment += "-" + branchName
    }
    echo "Using environment: ${environment}"
    env.ENVIRONMENT = environment
}

def showEnvironmentVariables() {
    sh 'env | sort > env.txt'
    sh 'cat env.txt'
}

// ================================================================================================
// Build steps
// ================================================================================================

def buildApp() {
     dir("webapp") {
        withDockerContainer("maven:3.5.0-jdk-8-alpine") { sh "mvn clean install"}
        archiveArtifacts '**/target/spring-boot-web-jsp-1.0.war'
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'] )
     }
}

def buildAndRegisterDockerImage() {
    def buildResult
    docker.withRegistry(env.REGISTRY_URL) {
        echo "Connect to registry at ${env.REGISTRY_URL}"
        dockerRegistryLogin()
        echo "Build ${env.IMAGE_NAME}"
        buildResult = docker.build(env.IMAGE_NAME)
        echo "Register ${env.IMAGE_NAME} at ${env.REGISTRY_URL}"
        buildResult.push()
        echo "Disconnect from registry at ${env.REGISTRY_URL}"
        sh "docker logout ${env.REGISTRY_URL}"
    }
}

def dockerRegistryLogin() {
    def login_command = ""
    withDockerContainer("garland/aws-cli-docker") {
        login_command = sh(returnStdout: true,
            script: "aws ecr get-login --region ${AWS_REGION} | sed -e 's|-e none||g'"
        )
    }
    sh "${login_command}"
}

// ================================================================================================
// Deploy steps
// ================================================================================================
/*
def deployImage(environment) {
    def context = getContext(environment)
    def ip = findIp(environment)
    echo "Deploy ${env.IMAGE_NAME} to '${environment}' environment (in context: ${context})"
    sshagent (credentials: ["${env.SYSTEM_NAME}-${context}-helloworld"]) {
        sh """ssh -o StrictHostKeyChecking=no -tt \"ec2-user@${ip}\" \
            sudo /opt/dso/deploy-app  \"${env.IMAGE_NAME}\" \"${env.REGISTRY_URL}\"
"""
    }
    echo "Ensure site is up"
     // TODO: Replace with wait loop that tests if the siste is responsive
    sleep time: 10, unit: 'SECONDS'
}

def getContext(environment) {
    return (env.BRANCH_NAME == 'master') ? environment : 'dev'
}


// ================================================================================================
// Test steps
// ================================================================================================

def runSecurityTest() {
    def sonarReportDir = "target/sonar"
    def jenkinsIP = findJenkinsIp()
    dir("webapp") {
        withDockerContainer("maven:3.5.0-jdk-8-alpine")  {
            sh "mvn sonar:sonar -Dsonar.host.url=http://${jenkinsIP}:9000"
        }
        sh "ls -al ${sonarReportDir}"                                     */
        //archiveArtifacts "**/${sonarReportDir}/*.txt"
 //    }
//}

/*

def runBrowserTest(environment) {
    def ip = findIp(environment)
    def workDir = "src/test"
    def testsDir="${workDir}/python"
    def resultsDir = "target/browser-test-results/${environment}"
    def resultsPrefix = "${resultsDir}/results-${env.BUILD_ID}"
    def sitePackagesDir="${workDir}/resources/lib/python2.6/site-packages"
    def script = """
        export PYTHONPATH="${sitePackagesDir}:${testsDir}"
        mkdir -p ${resultsDir}
        /usr/bin/python -B -u ./${testsDir}/helloworld/test_suite.py \
            "--base-url=http://${ip}" \
            --webdriver-class=PhantomJS\
            --reuse-driver \
            --environment ${environment} \
            --default-wait=15 \
            --verbose \
            --default-window-width=800 \
            --test-reports-dir=${resultsDir} \
            --results-file=${resultsPrefix}.csv
"""
    dir("webapp") {
        withDockerContainer("killercentury/python-phantomjs") { sh "${script}" } */
//        step([$class: 'JUnitResultArchiver', testResults: "**/${resultsDir}/TEST-*.xml"]) 
 //       sh "ls -lhr ${resultsDir}"
 //       archiveArtifacts "**/${resultsPrefix}.*"
 /*   }
}

// ================================================================================================
// Utility steps
// ================================================================================================
/*

def proceedTo(environment) {
    def description = "Choose 'yes' if you want to deploy to this build to " + 
        "the ${environment} environment"
    def proceed = 'no'
    timeout(time: 4, unit: 'HOURS') {
        proceed = input message: "Do you want to deploy the changes to ${environment}?",
            parameters: [choice(name: "Deploy to ${environment}", choices: "no\nyes",
                description: description)]
        if (proceed == 'no') {
            error("User stopped pipeline execution")
        }
    }
}

def findIp(environment) {
    def ip = ""
    withDockerContainer("garland/aws-cli-docker") {
       ip = sh(returnStdout: true,
            script: """aws ec2 describe-instances \
                --filters "Name=instance-state-name,Values=running" \
                "Name=tag:Name,Values=${env.SYSTEM_NAME}-${environment}-helloworld" \
                --query "Reservations[].Instances[].{Ip:PrivateIpAddress}" \
                --output text --region ${env.AWS_REGION} | tr -d '\n'
"""
        )
    }
    echo "ip=[${ip}]"
    return ip
}

def findJenkinsIp() {
    def ip = ""
    withDockerContainer("garland/aws-cli-docker") {
       ip = sh(returnStdout: true,
            script: """aws ec2 describe-instances \
                --filters "Name=instance-state-name,Values=running" \
                "Name=tag:Name,Values=${env.SYSTEM_NAME}-shared-jenkins" \
                --query "Reservations[].Instances[].{Ip:PrivateIpAddress}" \
                --output text --region ${env.AWS_REGION} | tr -d '\n'
"""
        )
    }
    echo "ip=[${ip}]"
    return ip
}

// def startSonarQube() {
//     if (!isSonarQubeRunning()) {
//         echo "Restarting SonarQube"
//         stopSonarQube()
//         sh "docker run -d --name ${env.SONARQUBE_IMAGE_NAME} -p 9000:9000 -p 9092:9092 sonarqube"
//     } else {
//         echo "SonarQube already running"
//     }
// }

// def stopSonarQube() {
//     if (isSonarQubeRunning()) {
//         sh "docker stop ${env.SONARQUBE_IMAGE_NAME}"
//         sh "docker rm ${env.SONARQUBE_IMAGE_NAME}"
//     }
// }

// def isSonarQubeRunning() {
//     def imageID = sh(returnStdout: true, script: """
//         docker ps | grep '${env.SONARQUBE_IMAGE_NAME}' | awk -F" " '{print \$1;}' | tr -d '\n'
// """
//     )
//     echo "SonarQube ImageID=${imageID}"
//     return (imageID?.trim())
// }


*/




//New section of jenkins file








/* 

properties ([
  parameters ([
    string(name: 'appRepoURL', value: "", description: "Application's git repository"),
    string(name: 'dockerImage', value: "", description: "docker Image with tag"),
    string(name: 'targetURL', value: "", description: "Web application's URL"),
    choice(name: 'appType', choices: ['Java', 'Node', 'Angular'], description: 'Type of application'),
    string(name: 'hostMachineName', value: "", description: "Hostname of the machine"),
    string(name: 'hostMachineIP', value: "", description: "Public IP of the host machine")
   // password(name: 'hostMachinePassword', value: "", description: "Password of the target machine")
    ])
])

def repoName="";
def app_type="";
def workspace="";

node {
        stage ('Checkout SCM') 
        {
	  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            checkout scm
            workspace = pwd ()
	  }
        }
  
        stage ('pre-build setup')
        {
	  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	      sh """
              docker-compose -f Sonarqube/sonar.yml up -d
              docker-compose -f Anchore-Engine/docker-compose.yaml up -d
              """
	  }
        }
        
        stage ('Check secrets')
        {
	  catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
            sh """
            rm trufflehog || true
            docker run gesellix/trufflehog --json --regex ${appRepoURL} > trufflehog
            cat trufflehog
            """
	  
	    def truffle = readFile "trufflehog"
		   
	    if (truffle.length() == 0){
              echo "Good to go" 
            }
            else {
              echo "Warning! Secrets are committed into your git repository."
	      throw new Exception("Secrets might be committed into your git repo")
            }
	  }
        } 
        
	stage ('Source Composition Analysis')
        {
	   catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
	    sh "git clone ${appRepoURL} || true" 
            repoName = sh(returnStdout: true, script: """echo \$(basename ${appRepoURL.trim()})""").trim()
            repoName=sh(returnStdout: true, script: """echo ${repoName} | sed 's/.git//g'""").trim()
	  
	    if (appType.equalsIgnoreCase("Java")) {
	      app_type = "pom.xml"	
	    }
	    else {
	      app_type = "package.json"
	      dir ("${repoName}") {
	        sh "npm install"
              }
	    }
	  
            snykSecurity failOnIssues: false, projectName: '$BUILD_NUMBER', severity: 'high', snykInstallation: 'SnykSec', snykTokenId: 'snyk-token', targetFile: "${repoName}/${app_type}"
		   
	    def snykFile = readFile "snyk_report.html"
	    if (snykFile.exists()) {
		throw new Exception("Vulnerable dependencies found!")    
	    }
	    else {
		echo "Please enter the app repo URL"
	    	currentBuild.Result = "FAILURE"
	    }
   	    
	  }
	}

        
        stage ('SAST')
        {
	  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	    if (appType.equalsIgnoreCase("Java")) {
	      withSonarQubeEnv('sonarqube') {
	        dir("${repoName}"){
	          sh "mvn clean package sonar:sonar"
	        }
	      }
	    
	    timeout(time: 1, unit: 'HOURS') {   
	      def qg = waitForQualityGate() 
	      if (qg.status != 'OK') {     
	        error "Pipeline aborted due to quality gate failure: ${qg.status}"    
	        }	
	      }
	     }
            }
	  }
        
        stage ('Container Image Scan')
        {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	    sh "rm anchore_images || true"
            sh """ echo "$dockerImage" > anchore_images"""
            anchore 'anchore_images'
	  }
        }
        
        stage ('DAST')
        {
          catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
	    sh """
	      rm -rf Archerysec-ZeD/zap_result/owasp_report || true
	      docker run -v `pwd`/Archerysec-ZeD/:/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py \
    	      -t ${targetURL} -J owasp_report
	    """
          }
	}
	
	stage ('Inspec')
	{
	  
	  
  	  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {

	    /*to install inspec as a package
	    curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P inspec*/

/*

	    sh """
	      rm inspec_results || true
	      inspec exec Inspec/hardening-test -b ssh --host=${hostMachineIP} --user=${hostMachineName} -i ~/.ssh/id_rsa --reporter json:./inspec_results
	      cat inspec_results | jq
	    """
	  }	
	}
	
        stage ('Clean up')
        {
	  catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            sh """
	      rm -r ${repoName} || true
	      mkdir -p reports/trufflehog
	      mkdir -p reports/snyk
	      mkdir -p reports/Anchore-Engine
	      mkdir -p reports/OWASP
	      mkdir -p reports/Inspec
              mv trufflehog reports/trufflehog || true
	      mv *.json *.html reports/snyk || true        */
//	      cp -r /var/lib/jenkins/jobs/${JOB_NAME}/builds/${BUILD_NUMBER}/archive/Anchore*/*.json ./reports/Anchore-Engine ||  true
//	      mv inspec_results reports/Inspec || true
/*            """
		//cp Archerysec-ZeD/owasp_report reports/OWASP/ || ture	    
		  
	    sh """
	    docker system prune -f
	    docker-compose -f Sonarqube/sonar.yml down
            docker-compose -f Anchore-Engine/docker-compose.yaml down -v
	    """
	  }
        }
}
       */
    

