pipeline {
    agent any

    environment {
        HANA_XSA_CREDS      = credentials('TU_CICD')
        XSA_API_ENDPOINT    = <<XSA_API_Endpoint>>
        ORGANIZATION        = <<XSA_Organization>>
        CI_SPACE            = <<XSA_Space>>
        modulePaths         = sh(script: '''awk -F: '$1 ~ /path/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true)
        mtaName             = sh(script: '''awk -F: '$1 ~ /^ID/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true).trim()

        SCHEMA_VAL           = 'DW_VAL'
        DB_HOST             = <<HANA_IP_Address>>
        DB_PORT             = <<HANA_DB_Tenant_Port>>
        DB_NAME             = <<HANA_DB_Tenant_Name>>
        HANA_TECHN_CREDS    = credentials('TU_CICD')
        HANA_TECHNICAL_USER       = "$HANA_TECHN_CREDS_USR"
        HANA_TECHNICAL_PASSWORD   = "$HANA_TECHN_CREDS_PSW"      

    }
    stages {
        stage('Build') {
			when {
                expression {env.BRANCH_NAME == 'release'}
            }
            steps {
                echo 'Building..'
                // create local npmrc file
                    sh('''cat <<EOF > .npmrc
                    registry=https://registry.npmjs.org/
                    @sap:registry=https://registry.npmjs.org
                    EOF''')

                // add .npmrc to the modulePaths
                    sh('''for path in ${WORKSPACE}/${modulePaths}; do
                        if [ -d "$path" ];
                            then ln -sft $path ../.npmrc
                        fi
                        done''')

                // build mtar package
                    sh('<<mbt_installation_path>>/mbt build -p=xsa --mtar ${mtaName}.mtar -m=verbose')
            }
        }
        stage('Deploy') {
			when {
                expression {env.BRANCH_NAME == 'release'}
            }        
            steps {
	                echo 'Deploying....'
	                sh('<<xs_client_installation_path>>/bin/xs api $XSA_API_ENDPOINT --skip-ssl-validation')
	                sh('<<xs_client_installation_path>>/bin/xs login -u $HANA_XSA_CREDS_USR -p $HANA_XSA_CREDS_PSW -o $ORGANIZATION -s $CI_SPACE')
	                sh('<<xs_client_installation_path>>/bin/xs deploy -f ${WORKSPACE}/mta_archives/${mtaName}.mtar')
                }
        }
        stage('Test') {
            steps {
                git clone 'https://github.com/ISR-SAP-HANA-SQL-DWH/Demo_TA.git'

                dir("${WORKSPACE}/<<Jenkins_Pipeline_Workspace>>/Demo_TA"){
                    sh('''
                        export NVM_DIR="$HOME/.nvm" 
                        [ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"
                        [ -s "$NVM_DIR/bash_completion" ] && \\. "$NVM_DIR/bash_completion"
                        nvm use --lts
                        npm install
                        npm test      
                    ''')
                }
            }
        }
    }
}
