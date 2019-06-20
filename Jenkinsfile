pipeline {
  agent {
    label "xenial-intel-device-plugins"
  }
  options {
    timeout(time: 2, unit: "HOURS")
  }
  environment {
    RUNC_VERSION="v1.0.0-rc6"
    CRIO_VERSION="v1.11.5"
    OSTREE_VERSION="v2018.8"
    BUILDAH_VERSION="v1.4"
    GO_VERSION="1.11.11"
    GO_TAR="go${GO_VERSION}.linux-amd64.tar.gz"
    GOROOT="/usr/local/go"
    GOPATH="/tmp/go"
    PATH="${env.PATH}:/usr/local/go/bin:${GOPATH}/bin"
    REPO_NAME="intel-device-plugins-for-kubernetes"
    REPO_DIR="$GOPATH/src/github.com/intel/${REPO_NAME}"
  }
  stages {
    stage("Get requirements") {
      steps {
        sh "curl -O https://dl.google.com/go/${GO_TAR}"
        sh "tar -xvf $GO_TAR"
        sh "sudo mv go $GOROOT"
        sh "sudo apt-get -y install e2fslibs-dev libfuse-dev libgpgme11-dev libdevmapper-dev libglib2.0-dev libprotobuf-dev"
        sh "mkdir -p $GOPATH/src/github.com/intel"
        sh "cp -rf ${env.WORKSPACE} $REPO_DIR"
        dir(path: "$REPO_DIR") {
          sh "go get -v golang.org/x/lint/golint"
          sh "go get -v github.com/fzipp/gocyclo"
        }
        sh "mkdir -p ${GOPATH}/src/github.com/containers"
        dir(path: "${GOPATH}/src/github.com/containers") {
          sh "git clone --single-branch --depth 1 -b $BUILDAH_VERSION https://github.com/containers/buildah"
        }
        dir(path: "${GOPATH}/src/github.com/containers/buildah") {
          sh 'make buildah TAGS=""'
          sh "sudo cp buildah /usr/local/bin"
          sh "sudo mkdir -p /etc/containers"
          sh '''echo '[registries.search]' > registries.conf'''
          sh '''echo 'registries = ["docker.io"]' >> registries.conf'''
          sh "sudo mv registries.conf /etc/containers/registries.conf"
          sh "sudo curl https://raw.githubusercontent.com/kubernetes-sigs/cri-o/$CRIO_VERSION/test/policy.json -o /etc/containers/policy.json"
          sh "sudo curl -L https://github.com/opencontainers/runc/releases/download/$RUNC_VERSION/runc.amd64 -o /usr/bin/runc"
          sh "sudo chmod +x /usr/bin/runc"
        }
     }
   }
   stage("make lint") {
      steps {
        dir(path: "$REPO_DIR") {
          sh "make lint"
        }
      }
    }
    stage("make format") {
      steps {
        dir(path: "$REPO_DIR") {
          sh "make format"
        }
      }
    }
    stage("make vet") {
      steps {
        dir(path: "$REPO_DIR") {
          sh "make vet"
        }
      }
    }
    stage("make cyclomatic-check") {
      steps {
        dir(path: "$REPO_DIR") {
          sh "make cyclomatic-check"
        }
      }
    }
    stage("make test") {
      steps {
        dir(path: "$REPO_DIR") {
          sh "make test"
        }
      }
    }
    stage('make plugins') {
      parallel {
        stage("make intel-fpga-admissionwebhook") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make intel-fpga-admissionwebhook"
            }
          }
        }
        stage("make intel-fpga-plugin") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make intel-fpga-plugin"
            }
          }
        }
        stage("make intel-gpu-plugin") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make intel-gpu-plugin"
            }
          }
        }
        stage("make intel-qat-plugin") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make intel-qat-plugin"
            }
          }
        }
      }
    }
    stage('make images') {
      parallel {
        stage("make images with buildah") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make images BUILDER=buildah"
            }
          }
        }
        stage("make images with docker") {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make images"
            }
          }
        }
      }
    }
    stage('make demos') {
      parallel {
        stage('make demos') {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make demos"
            }
          }
        }
        stage('make demos BUILDER=buildah') {
          steps {
            dir(path: "$REPO_DIR") {
              sh "make demos BUILDER=buildah"
            }
          }
        }
      }
    }
  }
}
