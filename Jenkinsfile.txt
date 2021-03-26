pipeline {
    agent any

    environment {
        HANA_XSA_CREDS      = credentials('XSA_JENKINS')
        XSA_API_ENDPOINT    = 'https://awshn2.isr.local:30030/'
        ORGANIZATION        = 'ISR'
        CI_SPACE            = 'DEMO'
        modulePaths         = sh(script: '''awk -F: '$1 ~ /path/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true)
        mtaName             = sh(script: '''awk -F: '$1 ~ /^ID/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true).trim()

        SCHEMA_AL           = 'DW_VAL'
        DB_HOST             = '10.7.0.103'
        DB_PORT             = '30013'
        DB_NAME             = 'HN3'
        HANA_TECHN_CREDS    = credentials('TU_CICD')
        HANA_TECHNICAL_USER       = "$HANA_TECHN_CREDS_USR"
        HANA_TECHNICAL_PASSWORD   = "$HANA_TECHN_CREDS_PSW"      

        JENKINSUSER_CREDS   = credentials('ff9118fa-4ef6-4fb2-a357-9b40114b6683')
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
                    sh('/data/SAP/mbt/mbt build -p=xsa --mtar ${mtaName}.mtar -m=verbose')
            }
        }
        stage('Deploy') {
			when {
                expression {env.BRANCH_NAME == 'release'}
            }        
            steps {
	                echo 'Deploying....'
	                sh('/data/SAP/xs_client/bin/xs api $XSA_API_ENDPOINT --cacert /data/SAP/xs_client/xsa_api.cer')
	                sh('/data/SAP/xs_client/bin/xs login -u $HANA_XSA_CREDS_USR -p $HANA_XSA_CREDS_PSW -o $ORGANIZATION -s $CI_SPACE')
	                sh('/data/SAP/xs_client/bin/xs deploy -f ${WORKSPACE}/mta_archives/${mtaName}.mtar')
                }
        }
        stage('Test') {
            steps {
                git credentialsId: 'ff9118fa-4ef6-4fb2-a357-9b40114b6683',
                    url: 'https://github.com/ISR-SAP-HANA-SQL-DWH/Demo_TA.git'

                dir("/var/lib/jenkins/workspace/CICD_DEMO_development/Demo_TA"){
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