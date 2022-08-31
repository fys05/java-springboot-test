def secret_name = "registry-auth-secret"
def k8s_auth = "kubeconfig"
podTemplate(
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
            image: "maven:3.8.1-openjdk-8",
            ttyEnabled: true,
            command: 'cat'
        )
    ],
    volumes:[
        hostPathVolume(mountPath:'/var/run/docker.sock',hostPath:'/run/docker.sock'),
        persistentVolumeClaim(claimName: 'maven-repo', mountPath: '/root/.m2', readOnly: false)
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
        def image_name = "fuyongsheng/k8s-repo:java-test"
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