pipeline {

    agent any

 

    environment {

        PROJECT_WORKSPACE = "/var/jenkins_home/workspace/jenkins-multibranch-poc"

    }

 

    stages {

        stage('service changes found') {

            steps {
                echo "${env.CHANGE_ID}"
                echo "${env.BRANCH_NAME}"
            }

        }

        stage('Git checkout') {

            steps {

                dir("$env.PROJECT_WORKSPACE"){

                checkout scm

                }

            }

        }

 


 

        stage('Get service change id') {

            when {

                expression {

                    env.CHANGE_ID == null

                }

            }

            steps {

                dir("$env.PROJECT_WORKSPACE"){

                sh """
                   echo "print commit id: ${env.GIT_COMMIT}"
                   echo `git diff-tree --no-commit-id --name-only -r ${env.GIT_COMMIT} | cut -d/ -f1| sort -u`  > abc.txt
                   cat abc.txt
                """

                script {
                    service_dir_name = readFile('abc.txt').trim()
                    }
                    echo "${service_dir_name}"

                }

            }

        }

 

        stage('Get service name for PR') {
            when {
                expression {
                    env.CHANGE_ID != null
                }
            }

            steps {

                dir("$env.PROJECT_WORKSPACE"){

                sh """
                   echo "print commit id: ${env.GIT_COMMIT}"
                   git checkout origin/${BRANCH_NAME}
                   latest_commit_id=`git log -n 1 --pretty=format:"%H"`
                   echo $latest_commit_id
                   echo `git diff-tree --no-commit-id --name-only -r $latest_commit_id | cut -d/ -f1| sort -u`  > abc.txt
                   cat abc.txt
                """
                script {
                    service_dir_name = readFile('abc.txt').trim()
                    }
                    echo "${service_dir_name}"

                }

            }

        }

       

        stage('Check service having latest commit') {

            steps {

                dir("$env.PROJECT_WORKSPACE"){

                    script {
                        service = findservice("${service_dir_name}")
                    }

                }

            }

        }

    }

}

 

def findservice(project_name) {

 

    def is_Service = false
    List<String> services = new ArrayList<String>();
    
    services = ['data-ingestion','data-integration'];

    String commit_name = project_name;
    List commit_files = commit_name.split(" ");
    SERVICE_DIR = env.BRANCH_NAME.replace("/", "%2F")

    if (commit_files.intersect(services) == []) /*For commits in external files*/ {

        for (String item : services) {

            build job: "${item}/${SERVICE_DIR}"

        }

    } else {

        for (String item : commit_files)  {

            if (services.contains(item) == false) {

                for (String svc_item : services)  {

                    build job: "${svc_item}/${SERVICE_DIR}"

                }

                break;

            } else

                is_Service = true

        }

    }

    if (is_Service == true) {

        List<String> commit_services = new ArrayList<String>();

        commit_services = commit_files.intersect(services)

       

        for (String item : commit_services)  {

                build job: "${item}/${SERVICE_DIR}"

            }

    }

}