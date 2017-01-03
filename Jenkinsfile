#!groovy

// Jenkinsfile for compiling, testing, and packaging the Dashel libraries.
// Requires CMake plugin from https://github.com/davidjsherman/aseba-jenkins.git in global library.

pipeline {
	agent any // use any available Jenkins agent
	stages {
		stage('Prepare') {
			steps {
				// checkout scm -- done automatically
				stash excludes: '.git', name: 'source'
			}
		}
		stage('Compile') {
			parallel {
				stage("Compile on debian") {
					agent {
						label 'debian'
					}
					steps {
						unstash 'source'
						CMake([label: 'debian'])
						stash includes: 'dist/**', name: 'dist-debian'
						stash includes: 'build/**', name: 'build-debian'
					}
				}
				stage("Compile on macos") {
					agent {
						label 'macos'
					}
					steps {
						unstash 'source'
						CMake([label: 'macos'])
						stash includes: 'dist/**', name: 'dist-macos'
					}
				}
				stage("Compile on windows") {
					agent {
						label 'windows'
					}
					steps {
						unstash 'source'
						CMake([label: 'windows'])
						stash includes: 'dist/**', name: 'dist-windows'
					}
				}
			}
		}
		stage('Test') {
			parallel {
				stage("Test on debian") {
					agent {
						label 'debian'
					}
					steps {
						unstash 'build-debian'
						dir('build/debian') {
							sh 'LANG=C ctest'
						}
					}
				}
			}
		}
		stage('Package') {
			parallel {
				stage("Build debian package") {
					agent {
						label 'debian'
					}
					steps {
						unstash 'dist-debian'
						unstash 'source'
						dir('dashel') {
							sh 'which debuild && debuild -i -us -uc -b'
						}
						sh 'mv libdashel*.deb libdashel*.changes libdashel*.build dist/debian/'
						stash includes: 'dist/**', name: 'dist-debian'
					}
				}
			}
		}
		stage('Archive') {
			steps {
				unstash 'dist-debian'
				unstash 'dist-macos'
				unstash 'dist-windows'
				archiveArtifacts artifacts: 'dist/**', fingerprint: true, onlyIfSuccessful: true
			}
		}
	}
}
