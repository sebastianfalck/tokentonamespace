pipeline {
    agent any

    environment {
        EXTERNAL_SERVER = 'https://external.example.com:6443'
        INTERNAL_SERVER = 'https://internal.example.com:6443'
        DRS_SERVER = 'https://drs.example.com:6443'
        TOKEN_JSON = 'tokens.json'
        OUTPUT_JSON = 'namespaces.json'
        TOKEN_REPO = 'https://github.com/usuario/otro-repo-con-token.git' // Cambia esta URL por la real
        TOKEN_REPO_CREDENTIALS = 'master-credentials' // Cambia este valor por el ID real de tus credenciales
    }

    stages {
        stage('Checkout Token Repository') {
            steps {
                script {
                    cleanWs()
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/master']],
                        userRemoteConfigs: [[
                            url: env.TOKEN_REPO,
                            credentialsId: env.TOKEN_REPO_CREDENTIALS
                        ]]
                    ])
                }
            }
        }
        stage('Extract Namespaces') {
            steps {
                script {
                    def tokens = readJSON file: env.TOKEN_JSON
                    def results = []

                    tokens.each { key, value ->
                        if (!value?.trim()) {
                            echo "Token vacío para ${key}, se omite."
                            return // No se puede usar 'continue' en Groovy closure, así que usamos 'return' aquí
                        }
                        echo "Procesando ${key}..."
                        def namespace = ""
                        def status = ""
                        wrap([$class: 'MaskPasswordsBuildWrapper', varPasswordPairs: [[password: value]]]) {
                            // Intenta login con el interno
                            def internalLogin = sh(
                                script: """
                                    oc login --insecure-skip-tls-verify --server=${INTERNAL_SERVER} --token=${value} > login_internal.log 2>&1
                                """,
                                returnStatus: true
                            )

                            if (internalLogin == 0) {
                                status = "internal"
                                namespace = sh(
                                    script: 'oc get project -o jsonpath="{.metadata.name}"',
                                    returnStdout: true
                                ).trim()
                                if (!namespace) {
                                    namespace = sh(
                                        script: 'oc project -q',
                                        returnStdout: true
                                    ).trim()
                                }
                                if (!namespace) {
                                    namespace = "No se pudo obtener namespace"
                                }
                                echo "Login exitoso en INTERNAL para ${key}. Namespace: ${namespace}"
                                sh 'oc logout'
                            } else {
                                echo "Login fallido en INTERNAL para ${key}."
                                // Intenta login con el externo
                                def externalLogin = sh(
                                    script: """
                                        oc login --insecure-skip-tls-verify --server=${EXTERNAL_SERVER} --token=${value} > login_external.log 2>&1
                                    """,
                                    returnStatus: true
                                )
                                if (externalLogin == 0) {
                                    status = "external"
                                    namespace = sh(
                                        script: 'oc get project -o jsonpath="{.metadata.name}"',
                                        returnStdout: true
                                    ).trim()
                                    if (!namespace) {
                                        namespace = sh(
                                            script: 'oc project -q',
                                            returnStdout: true
                                        ).trim()
                                    }
                                    if (!namespace) {
                                        namespace = "No se pudo obtener namespace"
                                    }
                                    echo "Login exitoso en EXTERNAL para ${key}. Namespace: ${namespace}"
                                    sh 'oc logout'
                                } else {
                                    echo "Login fallido en EXTERNAL para ${key}."
                                    // Intenta login con el drs
                                    def drsLogin = sh(
                                        script: """
                                            oc login --insecure-skip-tls-verify --server=${DRS_SERVER} --token=${value} > login_drs.log 2>&1
                                        """,
                                        returnStatus: true
                                    )
                                    if (drsLogin == 0) {
                                        status = "drs"
                                        namespace = sh(
                                            script: 'oc get project -o jsonpath="{.metadata.name}"',
                                            returnStdout: true
                                        ).trim()
                                        if (!namespace) {
                                            namespace = sh(
                                                script: 'oc project -q',
                                                returnStdout: true
                                            ).trim()
                                        }
                                        if (!namespace) {
                                            namespace = "No se pudo obtener namespace"
                                        }
                                        echo "Login exitoso en DRS para ${key}. Namespace: ${namespace}"
                                        sh 'oc logout'
                                    } else {
                                        echo "Login fallido en DRS para ${key}."
                                        echo "Todos los intentos de login fallaron para ${key}."
                                        status = "expired"
                                        namespace = ""
                                    }
                                }
                            }
                        }

                        results << [tokenname: key, tokens: value, namespace: namespace, status: status]
                    }

                    writeJSON file: env.OUTPUT_JSON, json: results, pretty: 4
                }
            }
        }
        stage('Publicar artefacto JSON') {
            steps {
                archiveArtifacts artifacts: env.OUTPUT_JSON, onlyIfSuccessful: true
            }
        }
    }
}