pipeline {
	agent { dockerfile true }
	environment {
		DMD_VERSION = 'v2.074.1'
	}
	stages {
		stage('fetch') {
			steps {
				script {
					if (env.JOB_NAME.endsWith("_pull-requests"))
						setGitHubPullRequestStatus state: 'PENDING', context: "${env.JOB_NAME}", message: "Fetching dependencies"
				}
				ansiColor('xterm') {
					sh '''

					if [ -e dmd ]; then
						pushd dmd
						git fetch
						git checkout -f ${DMD_VERSION}
						popd
					else
						git clone https://github.com/dlang/dmd.git --branch ${DMD_VERSION}
					fi

					pushd dmd

					git am <../0001-Added-PowerNex-to-the-backend.patch
					git am <../0002-Added-makefiles-for-building-a-PowerNex-CC.patch
					git am <../0003-Force-PowerNex-to-be-the-target.patch

					popd
					'''
				}
			}
		}

		stage('build') {
			steps {
				script {
					if (env.JOB_NAME.endsWith("_pull-requests"))
						setGitHubPullRequestStatus state: 'PENDING', context: "${env.JOB_NAME}", message: "Building powernex-dmd"
				}
				ansiColor('xterm') {
					sh '''
					pushd dmd
					make -f powernex.mak
					popd
					'''
        }
			}
		}

		stage('archive') {
			steps {
				script {
					if (env.JOB_NAME.endsWith("_pull-requests"))
						setGitHubPullRequestStatus state: 'PENDING', context: "${env.JOB_NAME}", message: "Archiving powernex-dmd"
				}
				ansiColor('xterm') {
					archiveArtifacts artifacts: 'dmd/powernex-dmd', fingerprint: true
				}
			}
		}
	}

  post {
    success {
			script {
				if (env.JOB_NAME.endsWith("_pull-requests"))
					setGitHubPullRequestStatus state: 'SUCCESS', context: "${env.JOB_NAME}", message: "powernex-dmd building successed"
			}
    }
		failure {
			script {
				if (env.JOB_NAME.endsWith("_pull-requests"))
					setGitHubPullRequestStatus state: 'FAILURE', context: "${env.JOB_NAME}", message: "powernex-dmd building failed"
			}
		}
  }
}
