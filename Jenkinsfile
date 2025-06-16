pipeline {
    agent any

    environment {
        EXTERNAL_SERVER = 'https://external.example.com:6443'
        INTERNAL_SERVER = 'https://internal.example.com:6443'
        TOKEN_JSON = 'token.json'
        OUTPUT_JSON = 'namespaces.json'
    }

    stages {
        stage('Extract Namespaces') {
            steps {
                script {
                    def tokens = readJSON file: env.TOKEN_JSON
                    def results = []

                    tokens.each { key, token ->
                        def server = key.toLowerCase().contains('internal') ? env.INTERNAL_SERVER : env.EXTERNAL_SERVER
                        echo "Obteniendo namespace para ${key} en ${server}"

                        def ns = sh(
                            script: """
                                oc login --insecure-skip-tls-verify --server=${server} --token=${token} > /dev/null 2>&1
                                oc get project -o jsonpath="{.metadata.name}"
                            """,
                            returnStdout: true
                        ).trim()

                        results << [tokenname: key, token: token, namespace: ns]
                    }

                    writeJSON file: env.OUTPUT_JSON, json: results, pretty: 4
                }
            }
        }
    }
}