//secret_name生成方式;  
// 1.k8s集群master节点上登陆dockerhub（docker login -u username -p password）
// 2.k8s集群master节点执行（kubectl create secret docker-registry registry-auth-secret --docker-username=username --docker-password=password --docker-email=2580072139@qq.com）
def secret_name = "registry-auth-secret"  //docker凭证（k8s集群和dockerhub之间凭证），拉取镜像用
def k8s_auth = "kubeconfig"   //k8s集群认证信息，jenkins凭证生成，cat /root/.kube/config
podTemplate(                  // pod模板
    cloud: "kubernetes",
    namespace: "jenkins",
    label: 'jenkins-slave',
    // 配置容器信息
    containers: [
        containerTemplate(
            name: "jnlp",
            image: "jenkins/inbound-agent:4.11.2-4"
        ),
        containerTemplate(
            name: "docker",
            image: "docker:stable",
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: "maven",
            image: "maven:3.8.1-openjdk-8",   //maven镜像，注意jdk版本
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes:[
        hostPathVolume(mountPath:'/var/run/docker.sock',hostPath:'/run/docker.sock'),
        persistentVolumeClaim(claimName: 'maven-repo', mountPath: '/root/.m2', readOnly: false)  //maven打包所需依赖持久化
    ]
){
    node('jenkins-slave'){
        stage("echo"){
            echo "test"
        }
        stage('代码拉取') {
            git credentialsId: 'cd4dfa5f-49d6-4908-a344-fe418d28dcbd', url: 'http://192.168.40.210:31323/fff/java-springboot-test.git'
        }
        stage('编译打包构建镜像'){
            container('maven'){
                sh "pwd && cp settings.xml /usr/share/maven/conf/settings.xml"
                sh "mvn clean install dockerfile:build dockerfile:push"
            }
        }
        
        //def image_name = "fuyongsheng/k8s-repo:nginx-test"
        def image_name = "fuyongsheng/k8s-repo:java-test"  //镜像
        //部署到K8S
        sh "pwd && ls"
        //sh """sed -i 's#\$IMAGE_NAME#${image_name}#' nginx.yml 
        //      sed -i 's#\$SECRET_NAME#${secret_name}#' nginx.yml
        //"""
        //kubernetesDeploy configs: "nginx.yml",kubeconfigId: "${k8s_auth}"
        sh """
            sed -i 's#\$IMAGE_NAME#${image_name}#' springboot.yml
            sed -i 's#\$SECRET_NAME#${secret_name}#' springboot.yml
        """
        kubernetesDeploy configs: "springboot.yml",kubeconfigId: "${k8s_auth}"
    }
}
