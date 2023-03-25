pipeline {
    agent any

    stages {
        stage('Selecionar el TAG del Script') {
            steps {
                script {
                    def tagMap = [
                'CP001-Validar los nombres de los 10 owners': '@TEST_XRAY-6',
                'CP002-Editar del primer Owner de la lista, validar el cambio y hacer rollback.': '@TEST_XRAY-19',
                'Regresión Automatizada': '@XRAY-21'
            ]

                    def userInput = input(
                message: 'Seleccione el tag de Cucumber:',
                parameters: [
                    choice(
                        choices: tagMap.keySet().join('\n'),
                        description: 'Seleccionar Caso de Prueba',
                        name: 'CUCUMBER_TAG'
                    )
                ]
            )

                    env.CUCUMBER_TAG = tagMap[userInput]
                }
            }
        }
        stage('Limpiar el Espacio de Trabajo') {
            steps {
                cleanWs notFailBuild: true, deleteDirs: true
            }
        }
        stage('Descargar Proyecto Automatizado') {
            steps {
                // Eliminar el directorio del repositorio si existe
                bat 'IF EXIST proyecto RMDIR /S /Q proyecto'

                // Realizar la descarga del código desde el repositorio Git usando git clone
                withCredentials([usernamePassword(credentialsId: 'arivera', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    bat 'git clone https://github.com/aariverar/tsoft-framework-web-23.git proyecto'
                }
            }
        }
        stage('Ejecutar el Script Automatizado') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    dir('proyecto') {
                        bat "mvn test -D cucumber.filter.tags=\"${env.CUCUMBER_TAG}\""
                    }
                }
            }
        }
        stage('Archivar Resultado en XML') {
            steps {
                dir('proyecto') {
                    archiveArtifacts artifacts: '**/cucumber-junit-report.xml', fingerprint: true, allowEmptyArchive: false
                }
            }
        }
        stage('Archivar Resultado en JSON') {
            steps {
                dir('proyecto') {
                    archiveArtifacts artifacts: '**/cucumber.json', fingerprint: true, allowEmptyArchive: false
                }
            }
        }
        stage('Archivar Evidencias en Jenkins') {
            steps {
                dir('proyecto') {
                    archiveArtifacts artifacts: '**/target/resultado/**/*', fingerprint: true, allowEmptyArchive: false
                }
            }
        }

        stage('Importar Resultados a XRAY') {
                    steps {
                        script {
                            build(
                                job: 'XRAY Import Results',
                                propagate: true,
                                wait: true
                            )
                        }
                    }
        }

        stage('Comprimir Directorio Resultados') {
            steps {
                dir('proyecto') {
                    script {
                        // Cambia 'your-zip-file-name' al nombre que prefieras
                        bat "powershell Compress-Archive -Path target\\resultado -DestinationPath automationTest.zip"
                    }
                }
            }
        }
    }
}

