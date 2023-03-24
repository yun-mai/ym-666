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
            sh 'pwd && ls && echo ${JOB_NAME}'
              sshCommand remote: getServer(${params.deploy_host}), command: "pwd"              
            sshCommand remote: getServer(${params.deploy_host}), command: "ls -a"  
            sshCommand remote: getServer(${params.deploy_host}), command: "rm -Rf ${JOB_NAME}",failOnError:false 
            sshCommand remote: getServer(${params.deploy_host}), command: "mkdir ${JOB_NAME}",failOnError:false    
            sshPut remote: getServer(${params.deploy_host}), from: './server', into: './${JOB_NAME}'
            sshCommand remote: getServer(${params.deploy_host}), command: "ls -l  ./${JOB_NAME}"
            sshCommand remote: getServer(${params.deploy_host}), command: "docker-compose -f ${JOB_NAME}/server/docker-compose.yml down",failOnError:false       
            sshCommand remote: getServer(${params.deploy_host}), command: "docker-compose -f ${JOB_NAME}/server/docker-compose.yml up"              
          }
        }
        stage('admin-ui') {
          steps {
            sh 'sleep 120'
            sshPut remote: getServer(${params.deploy_host}), from: './admin-ui', into: './${JOB_NAME}'
            sshCommand remote: getServer(${params.deploy_host}), command: "ls -l  ./${JOB_NAME}/admin-ui"
            sshCommand remote: getServer(${params.deploy_host}), command: "docker build -t ccict/ym666-admin-ui ${JOB_NAME}/admin-ui",failOnError:false              
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


//           stage('部署镜像'){
//             ansiColor('xterm') {                 
//               docker.withRegistry(REGISTRY_URL, REGISTRY_CREDENTIALS_ID){
//                 def imgName = "${REGISTRY_DOMAIN}/${DOCKER_NAMESPACE}/${project_name}:${tagName}";

//                 for (item in ipList.tokenize(',')){                
//                   def sshServer = getServer(item)

//                   // 更新或下载镜像
//                   sshCommand remote: sshServer, command: "docker pull ${imgName}"

//                   try{
//                     // 停止容器
//                     sshCommand remote: sshServer, command: "docker stop ${project_name}"
//                     // 删除容器
//                     sshCommand remote: sshServer, command: "docker rm -f ${project_name}"
//                   }catch(ex){}

//                   // 启动容器
//                   sshCommand remote: sshServer, command: "docker run -d --name ${project_name} -e TZ=Asia/Shanghai ${imgName}"

//                   // 清理none镜像
//                   def clearNoneSSH = "n=`docker images | grep  '<none>' | wc -l`; if [ \$n -gt 0 ]; then docker rmi `docker images | grep  '<none>' | awk '{print \$3}'`; fi"
//                   sshCommand remote: sshServer, command: "${clearNoneSSH}"
//                 }
//               }
//             }
// -----------------------------------
// ©著作权归作者所有：来自51CTO博客作者catoop的原创作品，请联系作者获取转载授权，否则将追究法律责任
// Jenkins 插件之 SSH Pipeline Steps
// https://blog.51cto.com/u_1472521/5050322
