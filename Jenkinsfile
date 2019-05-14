//user class
class Colors_pick{
  Integer fg, bg
  /*
  +~~~~~~+~~~~~~+~~~~~~~~~~~+
  |  fg  |  bg  |  color    |
  +~~~~~~+~~~~~~+~~~~~~~~~~~+
  |  30  |  40  |  black    |
  |  31  |  41  |  red      |
  |  32  |  42  |  green    |
  |  33  |  43  |  yellow   |
  |  34  |  44  |  blue     |
  |  35  |  45  |  magenta  |
  |  36  |  46  |  cyan     |
  |  37  |  47  |  white    |
  |  39  |  49  |  default  |
  +~~~~~~+~~~~~~+~~~~~~~~~~~+
   */
}
//Function without fixed  parameters
def printInfo(color, my_str){
  def style = "${(char)27}[$color.fg;$color.bg"+"m"
  def reset_colors = "${(char)27}[39;49"+"m"
  ansiColor('xterm') {
      println(style+my_str+reset_colors)
    }
}

def generateStage(job, node_name) {
    return {
        node("${node_name}") {
          def red_color = new Colors_pick(fg: 31, bg: 49)
          def green_color = new Colors_pick(fg: 32, bg: 49)
          def blue_color = new Colors_pick(fg: 34, bg: 49)
          def magneta_color = new Colors_pick(fg: 35, bg: 49)

          stage ("Clean env to run test clearly for ${job}") {

            printInfo(green_color, 'Check if venv folder exist')

            printInfo(magneta_color, 'PATH = ' + WORKSPACE + '/' + TESTDIR)
            def folder = WORKSPACE + '/' + TESTDIR
            def NEEDCLEANING = false
            if( !fileExists(folder) ) {
              printInfo(green_color, 'Folder not exist, continue...')
            } else {
              printInfo(red_color, 'Folder exist, need to cleanup')
              NEEDCLEANING = true
            }
            if (NEEDCLEANING) {
              printInfo(red_color, 'Delete folder')
              dir ("${WORKSPACE}/${TESTDIR}") {
                  deleteDir()
                }
            }
          }

          stage('Get fresh code from master') {
            dir(WORKSPACE) {
              unstash "git-stash"
            }
          }

          stage ("Setup Python virtual environment ${job}") {
            ansiColor('xterm') {
              sh """
                # Green
                echo "\033[32m"
                virtualenv ${TESTDIR}
                echo "\033[0m"

                # Green
                echo "\033[32m"
                . ${TESTDIR}/bin/activate > /dev/null 2>&1
                echo "\033[0m"

                #Gray
                echo "\033[37m"
                pip install --upgrade ansible molecule docker
                echo "\033[0m"
              """
            }
          }

          stage ("Display versions  ${job}") {
            ansiColor('xterm') {
              sh """
                # Green
                echo "\033[32m"
                . ${TESTDIR}/bin/activate > /dev/null 2>&1
                echo "\033[0m"

                cd ${TESTDIR}

                # Blue
                echo "\033[34m"
                docker -v
                python -V
                ansible --version
                molecule --version
                echo "\033[0m"
              """
            }
          }

          stage("Run molecule test  ${job}") {
            ansiColor('xterm') {
              sh """
                . ${TESTDIR}/bin/activate > /dev/null 2>&1
                molecule test -s ${job}
              """
            }
          }
        }
    }
}

def scenarios = [debian9 : 'node2']
// def scenarios = [default : 'node1', debian9 : 'node2']
def parallelStagesMap = scenarios.collectEntries { sn, node ->
    ["${sn}", generateStage(sn, node)]
}



pipeline {

  agent {
    label 'master'
  }

  parameters {
        string(name: 'TESTDIR', defaultValue: 'virtenv')
      }

  environment {
        def TESTDIR = "${params.TESTDIR}"
      }

  options {
        parallelsAlwaysFailFast()
    }


  stages {
    stage('Put files to slaves') {
      steps {
        echo "${GIT_BRANCH}"
        stash name: "git-stash", excludes: "virtenv/*"
      }
    }

    stage('parallel stage') {
      steps {
        script {
          parallel parallelStagesMap
        }
      }
    }
  }
}
