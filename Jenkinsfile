 def app
def COLOR_MAP = ['SUCCESS': 'good', 'FAILURE': 'danger', 'UNSTABLE': 'danger', 'ABORTED': 'danger']

pipeline {
    agent none
    stages {
        stage('checkout scm') {            
            steps {
                 cleanWs();
                 cloneRepo();
                 VERSION_NUMBER = getVersionNumber();
                 currentBuild.displayName = "$VERSION_NUMBER";
            }
        }	
        stage('dotnet restore') {
            steps {
                  dotnet_restore();
            }
        }
        stage('dotnet clean & build') {
            steps {
                  dotnet_clean();
                  dotnet_build();
            }
        }
        stage('dotnet test') {
            steps {
                 dotnet_test();
            }
        }
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' 
              }
            }
            steps {
                echo "publish ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }

        post { 
        always { 
            echo 'I will always say Hello!'
        }
        aborted {
            echo 'I was aborted'
        }
        failure {
            mail to: 'aa@bb.cc',
            subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}

def cloneRepo() {
    checkout scm;
}

//Generates a version number
def getVersionNumber() {
    def out = sh(script: 'git rev-list --count HEAD', returnStdout: true);
    def array = out.split("\\r?\\n");
    def count = array[array.length - 1];

    def commitCount = count.trim();

    return commitCount;
}

def dotnet_restore(){
    	dir('API/Vita.Admin.API') {
		sh(script: 'dotnet restore Vita.Admin.API.csproj', returnStdout: true);
	}
}

def dotnet_clean(){
    	dir('API/Vita.Admin.API') {
		sh(script: 'dotnet clean Vita.Admin.API.csproj', returnStdout: true);
	}
}


def dotnet_build(){
	dir('API/Vita.Admin.API') {
		sh(script: 'dotnet build -c Release Vita.Admin.API.csproj', returnStdout: true);
	}
}


def dotnet_test(){
	dir('Tests/UnitTests/Vita.Business.Tests/') {
        try{
        sh(script: 'dotnet test --logger "trx;logfilename=unit_tests.xml" --results-directory ./UnitTestResults/ ', returnStdout: false);        }
        finally{
		  sleep(time:8,unit:"SECONDS")
        step([$class: 'MSTestPublisher', testResultsFile:"UnitTestResults/unit_tests.xml", failOnError: true, keepLongStdio: true])
        }
         
    }

}
