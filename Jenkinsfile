pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }
  environment {
    REPO_A_URL    = 'https://github.com/Myat-m/RepoA.git'
    REPO_A_BRANCH = 'main'
    REPO_C_URL    = 'https://github.com/Myat-m/Repo-C.git'
    REPO_C_BRANCH = 'main'

    WARN_FILE = 'warnings.log'
    OUT_DIR   = 'out'
    OUT_CSV   = 'out/warnings.csv'
    OUT_TAR   = 'warnings.tar.gz'
  }

  stages {
    stage('Checkout pipelines repo') {
      steps { checkout scm }
    }

    stage('Clone RepoA') {
      steps {
        dir('repoA') {
          git url: "${REPO_A_URL}", branch: "${REPO_A_BRANCH}"
        }
      }
    }

    stage('Generate Doxygen config with WARN_LOGFILE') {
      steps {
        sh '''
          set -euxo pipefail
          cd repoA
          doxygen -g Doxyfile

          if grep -q "^INPUT " Doxyfile; then
            sed -i 's|^INPUT .*|INPUT = src|' Doxyfile
          else
            echo "INPUT = src" >> Doxyfile
          fi

          if grep -q "^OUTPUT_DIRECTORY " Doxyfile; then
            sed -i 's|^OUTPUT_DIRECTORY .*|OUTPUT_DIRECTORY = html|' Doxyfile
          else
            echo "OUTPUT_DIRECTORY = html" >> Doxyfile
          fi

          if grep -q "^GENERATE_LATEX " Doxyfile; then
            sed -i 's|^GENERATE_LATEX .*|GENERATE_LATEX = NO|' Doxyfile
          else
            echo "GENERATE_LATEX = NO" >> Doxyfile
          fi

          if grep -q "^WARNINGS " Doxyfile; then
            sed -i 's|^WARNINGS .*|WARNINGS = YES|' Doxyfile
          else
            echo "WARNINGS = YES" >> Doxyfile
          fi

          if grep -q "^WARN_IF_UNDOCUMENTED " Doxyfile; then
            sed -i 's|^WARN_IF_UNDOCUMENTED .*|WARN_IF_UNDOCUMENTED = YES|' Doxyfile
          else
            echo "WARN_IF_UNDOCUMENTED = YES" >> Doxyfile
          fi

          if grep -q "^WARN_LOGFILE " Doxyfile; then
            sed -i 's|^WARN_LOGFILE .*|WARN_LOGFILE = ${WARN_FILE}|' Doxyfile
          else
            echo "WARN_LOGFILE = ${WARN_FILE}" >> Doxyfile
          fi
        '''
      }
    }

    stage('Run Doxygen') {
      steps {
        sh '''
          set +e
          cd repoA
          doxygen Doxyfile || true
        '''
      }
    }

    stage('Clone RepoC') {
      steps {
        dir('repoC') {
          git url: "${REPO_C_URL}", branch: "${REPO_C_BRANCH}"
        }
      }
    }

    stage('Install RepoC dependencies') {
      steps {
        sh '''
          cd repoC
          python -m pip install --upgrade pip
          if [ -f requirements.txt ] && [ -s requirements.txt ]; then
            python -m pip install -r requirements.txt
          fi
        '''
      }
    }

    stage('Parse warnings and package CSV') {
      steps {
        sh '''
          cd repoC
          mkdir -p "${OUT_DIR}"
          python parse_doxygen_warnings.py --input ../repoA/${WARN_FILE} --output "${OUT_CSV}"
          tar -czf "${OUT_TAR}" -C . "${OUT_DIR}"
        '''
      }
      post {
        success {
          archiveArtifacts artifacts: 'repoC/warnings.tar.gz', fingerprint: true
        }
      }
    }
  }

  post {
    success { echo 'PipelineC complete: warnings.tar.gz archived (contains out/warnings.csv).' }
    always  { cleanWs() }
  }
}
