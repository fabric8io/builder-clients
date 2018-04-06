#!/usr/bin/groovy
/**
 * Copyright (C) 2015 Red Hat, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *         http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def utils = new io.fabric8.Utils()
def flow = new io.fabric8.Fabric8Commands()

dockerTemplate {
  clientsNode {
    ws {
      checkout scm

      def version = utils.isCI() ? "SNAPSHOT-${env.BRANCH_NAME}-${env.BUILD_NUMBER}" : getNewVersion()
      def image = "fabric8/builder-clients:${version}"

      echo "Building image ${image}"

      container('docker') {

        stage('Build docker image') {
          sh "docker build -t ${image} ."
        }

        stage('Push docker image') {
          sh "docker push ${image}"
        }
      }

      stage('Test') {

        // Spawn the image just created
        clientsNode(clientsImage:image) {
          container('clients') {
            echo 'Container spawned successfully'
            sh 'cat /etc/redhat-release'
            sh 'hostname'
            sh 'oc version'
            sh 'git --version'
          }
        }
      }

      if (utils.isCI()) {
        stage('Notify the PR') {
          def pr = env.CHANGE_ID
          if (!pr) {
            error "No pull request number found so cannot comment on PR"
          }
          def message = "Snapshot builder-clients image is available for testing.  `docker pull ${image}`"
          flow.addCommentToPullRequest(message, pr, 'fabric8io-images/builder-clients')
        }
      }
    }
  }
}
