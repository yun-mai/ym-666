def getServer(ip){
    def remote = [:]
    remote.name = "server-${ip}"
    remote.host = ip
    remote.port = 22
    remote.allowAnyHosts = true
    // 需要先在jenkins中创建一个id为cicd的凭据（用户名和密码）
    withCredentials([usernamePassword(credentialsId: "cicd", passwordVariable: "password", usernameVariable: "userName")]) {
    //withCredentials([sshUserPrivateKey(credentialsId: "ServiceServer", keyFileVariable: "identity", passphraseVariable: "", usernameVariable: "userName")]) {
        remote.user = "${userName}"
        remote.password = "${password}"
    }
    return remote
} 
// 此处使用参数化构建，需要在Jenkins的Job中指定构建参数deploy_host
def deploy_host="${params.deploy_host}"

pipeline {
  agent any
  options {
    parallelsAlwaysFailFast()  // https://stackoverflow.com/q/54698697/4480139
  }
  stages {
    stage("创建工作目录") {
      steps {
        sshCommand remote: getServer("${deploy_host}"), command: "rm -Rf ${JOB_NAME}" ,failOnError:false      
        sshCommand remote: getServer("${deploy_host}"), command: "mkdir ${JOB_NAME}"  
      }
    }
    stage("启动服务端") {
      parallel {
        stage("serve") {
          steps {
            sshPut remote: getServer("${deploy_host}"), from: "./server", into: "./${JOB_NAME}"
            sshCommand remote: getServer("${deploy_host}"), command: "ls -l  ./${JOB_NAME}"
            sshCommand remote: getServer("${deploy_host}"), command: "docker-compose -f ${JOB_NAME}/server/docker-compose.yml down --remove-orphans",failOnError:false       
            sshCommand remote: getServer("${deploy_host}"), command: "docker-compose -f ${JOB_NAME}/server/docker-compose.yml up -d"              
          }
        }
        stage("启动admin-ui") {
          steps {
            sh "sleep 2"
            sshPut remote: getServer("${deploy_host}"), from: "./admin-ui", into: "./${JOB_NAME}"
            sshCommand remote: getServer("${deploy_host}"), command: "ls -l  ./${JOB_NAME}/admin-ui"
            sshCommand remote: getServer("${deploy_host}"), command: "docker build --build-arg REACT_APP_SERVER_URL='${params.REACT_APP_SERVER_URL}' -t ccict/ym666-admin-ui ${JOB_NAME}/admin-ui",failOnError:false  
            sshCommand remote: getServer("${deploy_host}"), command: "docker stop ${JOB_NAME} &&  docker rm ${JOB_NAME}",failOnError:false                
            sshCommand remote: getServer("${deploy_host}"), command: "docker run -d -p 3019:80  --name ${JOB_NAME} --restart=always  ccict/ym666-admin-ui ",failOnError:false              
          }
        }
      }
    }
  }

  post { 
    always {
        sshCommand remote: getServer("${deploy_host}"), command: "echo 全部任务任务执行完毕！"              
        // sshCommand remote: getServer("${deploy_host}"), command: "docker-compose down"        
    }
  }
}
