pipeline {
    agent any

    environment {
        EXTERNAL_SERVER = 'https://external.example.com:6443'
        INTERNAL_SERVER = 'https://internal.example.com:6443'
        DRS_SERVER = 'https://drs.example.com:6443'
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
                        def namespace = ""
                        def status = ""
                        // Intenta login con el interno
                        def internalLogin = sh(
                            script: """
                                oc login --insecure-skip-tls-verify --server=${INTERNAL_SERVER} --token=${token} > login_internal.log 2>&1
                                echo $?
                            """,
                            returnStdout: true
                        ).trim()

                        if (internalLogin == "0") {
                            status = "internal"
                            namespace = sh(
                                script: 'oc get project -o jsonpath="{.metadata.name}"',
                                returnStdout: true
                            ).trim()
                        } else {
                            // Intenta login con el externo
                            def externalLogin = sh(
                                script: """
                                    oc login --insecure-skip-tls-verify --server=${EXTERNAL_SERVER} --token=${token} > login_external.log 2>&1
                                    echo $?
                                """,
                                returnStdout: true
                            ).trim()
                            if (externalLogin == "0") {
                                status = "external"
                                namespace = sh(
                                    script: 'oc get project -o jsonpath="{.metadata.name}"',
                                    returnStdout: true
                                ).trim()
                            } else {
                                // Intenta login con el drs
                                def drsLogin = sh(
                                    script: """
                                        oc login --insecure-skip-tls-verify --server=${DRS_SERVER} --token=${token} > login_drs.log 2>&1
                                        echo $?
                                    """,
                                    returnStdout: true
                                ).trim()
                                if (drsLogin == "0") {
                                    status = "drs"
                                    namespace = sh(
                                        script: 'oc get project -o jsonpath="{.metadata.name}"',
                                        returnStdout: true
                                    ).trim()
                                } else {
                                    status = "failed"
                                    namespace = ""
                                }
                            }
                        }

                        results << [tokenname: key, token: token, namespace: namespace, status: status]
                    }

                    writeJSON file: env.OUTPUT_JSON, json: results, pretty: 4
                }
            }
        }
    }
}