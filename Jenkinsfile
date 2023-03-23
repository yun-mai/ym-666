def getServer(ip){
    def remote = [:]
    remote.name = "server-${ip}"
    remote.host = ip
    remote.port = 22
    remote.allowAnyHosts = true
    withCredentials([usernamePassword(credentialsId: 'ym106', passwordVariable: 'password', usernameVariable: 'userName')]) {
    //withCredentials([sshUserPrivateKey(credentialsId: 'ServiceServer', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
        remote.user = "${userName}"
        remote.password = "${password}"
    }
    return remote
} 

pipeline {
  agent any
  options {
    parallelsAlwaysFailFast()  // https://stackoverflow.com/q/54698697/4480139
  }
  stages {
    stage('启动服务') {
      parallel {
        stage('serve') {
          steps {
            sh 'pwd && ls'
            sshCommand remote: getServer('119.3.41.106'), command: "pwd"              
            sshCommand remote: getServer('119.3.41.106'), command: "ls -a"  
            sshCommand remote: getServer('119.3.41.106'), command: "rm -Rf code",failOnError:false 
            sshCommand remote: getServer('119.3.41.106'), command: "mkdir code",failOnError:false    
            sshPut remote: getServer('119.3.41.106'), from: './server', into: './code'
            sshCommand remote: getServer('119.3.41.106'), command: "ls -l  ./code"
            sshCommand remote: getServer('119.3.41.106'), command: "docker-compose -f code/server/docker-compose.yml down",failOnError:false       
            sshCommand remote: getServer('119.3.41.106'), command: "docker-compose -f code/server/docker-compose.yml up"              
          }
        }
        stage('admin-ui') {
          steps {
            sh 'sleep 120'
            sshPut remote: getServer('119.3.41.106'), from: './admin-ui', into: './code'
            sshCommand remote: getServer('119.3.41.106'), command: "ls -l  ./code/admin-ui"
            sshCommand remote: getServer('119.3.41.106'), command: "docker build -t ccict/ym666-admin-ui code/admin-ui",failOnError:false              
          }
        }
      }
    }
  }

  post { 
    always {
    sshCommand remote: getServer('119.3.41.106'), command: "echo 全部任务任务执行完毕！"        
    // sshCommand remote: getServer('119.3.41.106'), command: "docker-compose down"        
    }
  }
}
