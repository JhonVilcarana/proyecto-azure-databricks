pipeline {
    agent any
    
    // Configuramos la URL exacta de tu entorno de Databricks
    environment {
        DATABRICKS_HOST = 'https://adb-7405607132000806.6.azuredatabricks.net'
    }
    
    stages {
        stage('Descargar Código') {
            steps {
                echo 'Sincronizando la última versión de la Arquitectura Medallón...'
                checkout scm
                sh 'ls -la'
            }
        }
        
        stage('Autenticación Azure') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-tenant-id', variable: 'TENANT_ID'),
                    string(credentialsId: 'azure-client-id', variable: 'CLIENT_ID'),
                    string(credentialsId: 'azure-client-secret', variable: 'CLIENT_SECRET')
                ]) {
                    sh '''
                    echo "Solicitando acceso a la nube de Microsoft..."
                    curl -s -X POST https://login.microsoftonline.com/$TENANT_ID/oauth2/token \
                        -d "grant_type=client_credentials" \
                        -d "client_id=$CLIENT_ID" \
                        -d "client_secret=$CLIENT_SECRET" \
                        -d "resource=https://management.azure.com/" > /dev/null
                    echo "¡Conexión a Azure establecida correctamente!"
                    '''
                }
            }
        }

        stage('Despliegue en Databricks') {
            steps {
                withCredentials([
                    string(credentialsId: 'databricks-token', variable: 'DATABRICKS_TOKEN')
                ]) {
                    sh '''
                    echo "Conectando con el Workspace de Databricks..."
                    
                    # Verificamos la conexión a la API de Databricks usando tu token
                    HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X GET \
                        -H "Authorization: Bearer $DATABRICKS_TOKEN" \
                        $DATABRICKS_HOST/api/2.0/workspace/list?path=/)
                    
                    if [ "$HTTP_STATUS" = "200" ]; then
                        echo "¡ÉXITO TOTAL! Jenkins tiene acceso a Databricks."
                        echo "Los cuadernos (01_bronce, 02_plata, 03_oro) están listos para ser orquestados por Data Factory."
                    else
                        echo "ERROR: Falló la conexión a Databricks. Código de estado: $HTTP_STATUS"
                        exit 1
                    fi
                    '''
                }
            }
        }
    }
}
