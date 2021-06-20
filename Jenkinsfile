def pkg
def publishVersion
def label = "jenkins-slave"

podTemplate (label: label, idleMinutes: 2160, containers: [
               containerTemplate(name: 'jnlp', image: 'aimvector/jenkins-slave'),
               containerTemplate(name: 'kubernetes', image: 'dockernaveensddp/jnlp:kubectl_pkg', ttyEnabled: true, command: 'cat'),
             ]) {

  node(label) {
    def repo = checkout scm
              {

      try {
        stage("Test if secrets exist") {
          container('kubernetes') {
            sh "kubectl cluster-info"
            sh "./deploy/sanity-check.sh"
          }
        }

        stage("Deployment") {
          if (repo.GIT_BRANCH == "master") {
           container('kubernetes') {
             sh "cd auth; find . -name '*templates*' -prune -o -name '*.yaml' -exec kubectl apply -f \\{\\} \\;"
           }
          }
        }
      }
      catch(Exception ex) { currentBuild = "FAILURE" }
      finally { 
        if (currentBuild.result == 'FAILURE') {
          office365ConnectorSend(
          message: "Build failed in project ${env.JOB_NAME}",
          status: "FAILURE",
          webhookUrl: "${env.HYPER_TEAMS_WEBHOOK}",
          color: "d31547"
          )
        }
      }
