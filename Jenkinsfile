#!groovy

properties([
  parameters([
    string(
      name: 'K8S_CLUSTER',
      defaultValue: 'kubernetes.demo',
      description: 'The Kubernetes Cluster you want to deploy to',
    ),
    string(
      name: 'K8S_NAMESPACE',
      defaultValue: 'kube-system',
      description: 'The Kubernetes namespace you want to deploy to',
    ),
    string(
      name: 'K8S_APP',
      defaultValue: 'myservice',
      description: 'The app name of this service',
    ),
    string(
      name: 'APP_NAME',
      defaultValue: 'myservice-proxy',
      description: 'The name of this service',
    ),
    string(
      name: 'APP_HOSTNAME',
      defaultValue: 'service.alpha.ace.evry.services',
      description: 'The hostname you want to access the app over',
    ),
    string(
      name: 'OAUTH_GITHUB_ORG',
      defaultValue: 'evry-bergen',
      description: 'GitHub Organisation for OAuth authentication',
    ),
    string(
      name: 'OAUTH_GITHUB_TEAM',
      defaultValue: 'ace',
      description: 'GitHub Team for OAuth authentication',
    ),
    string(
      name: 'SERVICE_NAME',
      defaultValue: 'kubernetes-service',
      description: 'Kubernetes Service Name',
    ),
  ])
])

@Library('utils')

import no.evry.Config

node('jenkins-docker-3') {
  ws {
    try {
      // Jenkins pipelines are "dumb" in that regards that it does not do
      // anything unless you explicitly tells it so. Here we checkout the
      // repository in order to the source code into the workspace.
      stage('Checkout') {
        checkout scm
      }

      // Configure which branches should get the following configurations. The
      // configurations are stored in the /build directory and controles the
      // Jenkins build and Kubernetes deployment.
      def envPatterns = [
        [env: 'prod', regex: /^master$/],
      ]

      def config = new Config(this).branchProperties(envPatterns)

      // Move Jenkins build parameters to git config properties
      [ 'K8S_CLUSTER',
        'K8S_NAMESPACE',
        'K8S_APP',
        'APP_NAME',
        'APP_HOSTNAME',
        'OAUTH_GITHUB_ORG',
        'OAUTH_GITHUB_TEAM',
        'SERVICE_NAME',
      ].eachWithIndex { item, index -> config[item] = env[item] }

      // Deploy to Kubernetes
      stage("Deploy") {
        def Boolean dryrun = config.JENKINS_DEPLOY != 'true'

        kubernetesDeploy(config, [k8sCluster: config.K8S_CLUSTER, dryrun: dryrun])
      }

    // Catch abort build interrupts and possible handle them differently. They
    // should not be reported as build failures to Slack etc.
    } catch (InterruptedException e) {
      throw e

    // Catch all build failures and report them to Slack etc here.
    } catch (e) {
      throw e

    // Clean up the workspace before exiting. Wait for Jenkins' asynchronous
    // resource disposer to pick up before we close the connection to the worker
    // node.
    } finally {
      step([$class: 'WsCleanup'])
      sleep 10
    }
  }
}
