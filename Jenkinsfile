pipeline {
    agent any
    
    stages {
        stage('Descargar Código') {
            steps {
                echo 'Descargando la última versión de tu rama main...'
                checkout scm
            }
        }
        
        stage('Conexión a Azure') {
            steps {
                echo 'Solicitando acceso a la nube de Microsoft...'
                
                // Extraemos las llaves secretas de la bóveda de Jenkins
                withCredentials([
                    string(credentialsId: 'azure-tenant-id', variable: 'TENANT_ID'),
                    string(credentialsId: 'azure-client-id', variable: 'CLIENT_ID'),
                    string(credentialsId: 'azure-client-secret', variable: 'CLIENT_SECRET')
                ]) {
                    sh '''
                    # Llamada directa a la API de Azure para autenticarnos
                    RESPUESTA=$(curl -s -X POST https://login.microsoftonline.com/$TENANT_ID/oauth2/token \\
                        -d "grant_type=client_credentials" \\
                        -d "client_id=$CLIENT_ID" \\
                        -d "client_secret=$CLIENT_SECRET" \\
                        -d "resource=https://management.azure.com/")
                    
                    # Verificamos si Azure nos entregó el token de acceso
                    if echo "$RESPUESTA" | grep -q "access_token"; then
                        echo "¡ÉXITO! Autenticación aceptada. Jenkins ahora tiene control sobre Azure."
                    else
                        echo "ERROR: Azure rechazó la conexión. Revisa las credenciales."
                        exit 1
                    fi
                    '''
                }
            }
        }
    }
}
