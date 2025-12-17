pipeline {
    agent any

    environment {
        CODEQL_URL = "https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz"
        CODEQL_DIR = "${env.WORKSPACE}/codeql"
        SOURCE_DIR = "${env.WORKSPACE}/vulnerable-source-code"
        DB_NAME = "my-app-db-2"
        SARIF_OUTPUT = "result1.sarif.json"
        GO_VERSION = "1.21.5"
        GO_DIR = "${env.WORKSPACE}/go"
        GOROOT = "${env.WORKSPACE}/go"
        GOPATH = "${env.WORKSPACE}/go-packages"
        PATH = "${env.WORKSPACE}/go/bin:${env.PATH}"
        PYTHON_DIR = "${env.WORKSPACE}/python"  // Use the same Python path as first pipeline
        PYTHON_URL = "https://github.com/indygreg/python-build-standalone/releases/download/20240107/cpython-3.11.7+20240107-x86_64-unknown-linux-gnu-install_only.tar.gz"
        VENV_DIR = "${env.WORKSPACE}/venv"
    }

    stages {
    //     stage('Install Go') {
    //         steps {
    //             echo "⬇️ Installing Go..."
    //             sh '''
    //                 export GO_TEMP_DIR=$(mktemp -d)
    //                 curl -LO "https://go.dev/dl/go1.21.5.linux-amd64.tar.gz"
    //                 tar -xzf go1.21.5.linux-amd64.tar.gz -C "$GO_TEMP_DIR"
    //                 rm -rf "$GO_DIR"
    //                 mv "$GO_TEMP_DIR/go" "$GO_DIR"
    //                 echo "✅ Go installed at $GO_DIR"
    //             '''
    //         }
    //     }

    stage('Install which') {
      steps {
        sh 'apt-get update && apt-get install -y which'
      }
  }

    stage('Ensure Python 3.11') {
      steps {
        echo "🐍 Checking if Python 3.11 exists..."
        sh '''
          if command -v python3.11 >/dev/null 2>&1; then
              echo "✅ System Python 3.11 found: $(python3.11 --version)"
          elif [ -x "$PYTHON_DIR/bin/python3.11" ]; then
              echo "✅ Prebuilt Python 3.11 already installed at $PYTHON_DIR"
              "$PYTHON_DIR/bin/python3.11" --version
          else
              echo "⬇️ Installing prebuilt Python 3.11..."
              mkdir -p "$PYTHON_DIR"
              cd "$PYTHON_DIR"
              curl -L -o python.tar.gz "$PYTHON_URL"
              tar -xzf python.tar.gz --strip-components=1
              echo "✅ Python extracted to: $PYTHON_DIR"
          fi
        '''
      }
    }

        stage('Download and Extract CodeQL') {
    steps {
        echo "⬇️ Downloading CodeQL bundle..."
        sh '''
            rm -rf "$CODEQL_DIR"        # Clean old bundles
            mkdir -p "$CODEQL_DIR"
            curl -L "$CODEQL_URL" -o codeql-bundle.tar.gz
            tar -xzf codeql-bundle.tar.gz -C "$CODEQL_DIR" --strip-components=1
            echo "✅ CodeQL installed to $CODEQL_DIR"
        '''
    }
}

        stage('Create CodeQL Database') {
            steps {
                echo "📦 Creating CodeQL database from source..."
                sh '''
                    rm -rf "$DB_NAME"
                    export PATH="$GO_DIR/bin:$PATH"
                    export GOROOT="$GO_DIR"
                    export GOPATH="$GOPATH"
                    "$CODEQL_DIR/codeql" database create "$DB_NAME" \
                      --language=python \
                      --source-root="$SOURCE_DIR"
                '''
            }
        }

        stage('Analyze Code with CodeQL') {
            steps {
                echo "🔍 Running CodeQL analysis..."
                sh '''
                    "$CODEQL_DIR/codeql" database analyze "$DB_NAME" \
                      codeql/go-queries \
                      --format=sarifv2.1.0 \
                      --output="$SARIF_OUTPUT"
                '''
            }
        }

        stage('Publish SARIF to Dashboard') {
            steps {
                echo "📄 Publishing SARIF Report..."
                archiveArtifacts artifacts: "$SARIF_OUTPUT", allowEmptyArchive: true, fingerprint: true
            }
        }
    }

    post {
        always {
            echo "✅ Build completed"
        }
    }
}
     
