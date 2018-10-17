pipeline {
  agent { label 'windows-slave' }
  stages {
    stage('Build') {
      steps {
        notifyBuild('STARTED')
        bat '''
          mkdir peru          
        '''      
      }
    }
    stage('Package Artifact') {
      steps {
        bat '''
          mkdir chile
        '''
        powershell '''
          mkdir tuu
        '''
        bat '''
          cd %workspace%
          mkdir asas
        '''
        withAWS(credentials:'devops-s3',region:'us-east-1') {
            s3Upload(file:"${env.BUILD_TAG}.zip", bucket:'belcorp-codedeploy', 'path':"Encore/${env.JOB_NAME}/${env.BUILD_TAG}.zip")        
        }
        
      }
    }
  }
  post {
      always {
        notifyBuild(currentBuild.result)
      }
  }
}
def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'
    String buildPhase = (buildStatus == 'STARTED') ? 'STARTED' : 'FINALIZED'
    commit = (buildStatus == 'STARTED') ? 'null' : powershell(returnStdout: true, script: "git log -n 1 --pretty=format:'%H'")
    
    powershell  """
        \$params = @{'name'= '${env.JOB_NAME}';
        'type'='pipeline';
        'build'= @{
            'phase'='${buildPhase}';
            'status'='${buildStatus}';
            'number'='${env.BUILD_ID}';
            'scm'=@{
              'commit'='${commit}';
            }
            'artifacts'=@{
              'bucket'='belcorp-codedeploy';
              'key'='Encore/${env.JOB_NAME}/${env.BUILD_TAG}.zip';
              'bundleType'='zip';
            }
          };
        }
        Invoke-WebRequest -Uri https://devops.belcorp.biz/gestionar_despliegues_qa -UseBasicParsing -Method POST -Body (\$params|ConvertTo-Json) -ContentType 'application/json'
    """
}
