pipeline {
    agent any
    
    environment {
        DATABRICKS_HOST = 'https://adb-7405607132000806.6.azuredatabricks.net'
    }
    
    stages {
        stage('Descargar Código') {
            steps {
                echo 'Sincronizando la última versión de la Arquitectura Medallón...'
                checkout scm
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
                    echo "Validando credenciales de despliegue en Azure..."
                    curl -s -X POST https://login.microsoftonline.com/$TENANT_ID/oauth2/token \
                        -d "grant_type=client_credentials" \
                        -d "client_id=$CLIENT_ID" \
                        -d "client_secret=$CLIENT_SECRET" \
                        -d "resource=https://management.azure.com/" > /dev/null
                    echo "¡Conexión a la nube de Microsoft verificada!"
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
                    
                    # 1. Extraemos el ID del repositorio limpiando cualquier espacio o carácter extra
                    REPO_ID=$(curl -s -X GET \
                        -H "Authorization: Bearer $DATABRICKS_TOKEN" \
                        "$DATABRICKS_HOST/api/2.0/repos" | grep -o -E '"id": ?[0-9]+' | head -1 | tr -dc '0-9')
                    
                    echo "-> ID de Carpeta Databricks detectado: $REPO_ID"
                    
                    if [ -z "$REPO_ID" ]; then
                        echo "ERROR: No se pudo obtener el ID del repositorio."
                        exit 1
                    fi
                    
                    echo "-> Ordenando a Databricks que extraiga (Pull) el código de GitHub..."
                    
                    # 2. Enviamos la orden de actualización y MOSTRAMOS la respuesta
                    curl -s -X PATCH \
                        -H "Authorization: Bearer $DATABRICKS_TOKEN" \
                        -H "Content-Type: application/json" \
                        -d '{"branch": "main"}' \
                        "$DATABRICKS_HOST/api/2.0/repos/$REPO_ID"
                        
                    echo "\\n\\n✅ ¡ÉXITO TOTAL! Todo tu código está sincronizado en producción."
                    '''
                }
            }
        }
    }
}
