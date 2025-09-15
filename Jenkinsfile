pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }
  environment {
    OUT_TAR = 'doc.tar.gz'
    REPO_A_URL = 'https://github.com/YOUR_USERNAME/YOUR_REPO_A.git'
    REPO_A_BRANCH = 'main'
  }
  stages {
    stage('Checkout pipelines repo') {
      steps { checkout scm }
    }

    stage('Clone RepoA fork') {
      steps {
        dir('repoA') {
          git url: "${REPO_A_URL}", branch: "${REPO_A_BRANCH}"
        }
      }
    }

    stage('Generate and adjust Doxygen config') {
      steps {
        sh '''
          set -euxo pipefail
          cd repoA
          doxygen -g Doxyfile
          sed -i 's|^INPUT .*|INPUT = src|' Doxyfile || echo "INPUT = src" >> Doxyfile
          sed -i 's|^OUTPUT_DIRECTORY .*|OUTPUT_DIRECTORY = html|' Doxyfile || echo "OUTPUT_DIRECTORY = html" >> Doxyfile
          sed -i 's|^GENERATE_LATEX .*|GENERATE_LATEX = NO|' Doxyfile || echo "GENERATE_LATEX = NO" >> Doxyfile
          sed -i 's|^WARNINGS .*|WARNINGS = YES|' Doxyfile || echo "WARNINGS = YES" >> Doxyfile
          sed -i 's|^WARN_IF_UNDOCUMENTED .*|WARN_IF_UNDOCUMENTED = YES|' Doxyfile || echo "WARN_IF_UNDOCUMENTED = YES" >> Doxyfile
        '''
      }
    }

    stage('Run Doxygen') {
      steps {
        sh '''
          set +e
          cd repoA
          doxygen Doxyfile 2> warnings.log || true
        '''
      }
    }

    stage('Package HTML as doc.tar.gz') {
      steps {
        sh '''
          set -euxo pipefail
          cd repoA
          if [ -d html/html ]; then
            tar -czf ../${OUT_TAR} -C html html
          elif [ -d html ]; then
            tar -czf ../${OUT_TAR} html
          else
            echo "No HTML output found" >&2
            exit 1
          fi
          [ -f warnings.log ] && cp warnings.log ../warnings.log || true
        '''
      }
      post {
        success {
          archiveArtifacts artifacts: 'doc.tar.gz,warnings.log', fingerprint: true
        }
      }
    }
  }
  post {
    success { echo 'PipelineB: doc.tar.gz archived (and warnings.log if present).' }
    always  { cleanWs() }
  }
}
