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

def red_color = new Colors_pick(fg: 31, bg: 49)
def green_color = new Colors_pick(fg: 32, bg: 49)
def blue_color = new Colors_pick(fg: 34, bg: 49)
def magneta_color = new Colors_pick(fg: 35, bg: 49)

pipeline {

  parameters {
        string(name: 'TESTDIR', defaultValue: 'molecule-test')
      }

  environment {
        def TESTDIR = "${params.TESTDIR}"
      }

  agent {
    // Node setup : minimal centos7, plugged into Jenkins, and
    // git config --global http.sslVerify false
    // sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm
    // sudo yum -y install python36u python36u-pip python36u-devel git curl gcc
    // git config --global http.sslVerify false
    // sudo curl -fsSL get.docker.com | bash
    label 'linux'
  }

  stages {

    stage ('Clean env to run test clearly') {
      steps {
        script {
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
      }

    }

    stage ('Setup Python virtual environment') {
      steps {
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
    }

    stage ('Get latest code') {
      steps {
        script {
          println 'test script'
        }
        sh '''
        echo "checkout scm"
        echo "-------------"
        '''
      }
    }



    stage ('Display versions') {
      steps {
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

    stage ('Molecule test') {
      steps {
        sh '''
          #source virtenv/bin/activate
          echo "molecule test"
        '''
      }
    }

  }

}
