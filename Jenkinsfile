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

def prepareEnv(){
  echo GIT_BRANCH
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

def initEnv(){
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

def displayVers(){
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

def moleculeTest(scenario_name){
  sh """
    #source virtenv/bin/activate
    echo "molecule test -s ${scenario_name}"
  """
}

def stagesDo(molecule_scenario){
  stages {
    stage ('Clean env to run test clearly') {
      steps {
        prepareEnv()
      }
    }
    stage ('Setup Python virtual environment') {
      steps {
        initEnv()
      }
    }
    stage ('Display versions') {
      steps {
        displayVers()
      }
    }
    stage('Run molecule test') {
      steps {
        moleculeTest(molecule_scenario)
      }
    }
  }
}

def red_color = new Colors_pick(fg: 31, bg: 49)
def green_color = new Colors_pick(fg: 32, bg: 49)
def blue_color = new Colors_pick(fg: 34, bg: 49)
def magneta_color = new Colors_pick(fg: 35, bg: 49)

pipeline {

  agent any

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
    parallel {
      stages {
        // when {
        //   branch 'master'
        // }
        // when { equals expected: 2, actual: currentBuild.number }

        stage('default scenario') {
          agent {
            label "node1"
          }
          stagesDo('default')
        }
        stage('rhel7 scenario') {
          agent {
            label "node2"
          }
          stagesDo('rhel7')
        }
      }

    }
  }
