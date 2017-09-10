/*
 * This color converter software is part of a micro-service architecture written for demonstration purposes.
 * Copyright (C)  2017  Gianni Bombelli @ Intr? S.r.l.
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 *
 */

node('nodejs') {

    def test_port = 3100 + env.BUILD_ID.toInteger()

    def array = env.JOB_NAME.split("/")
    def pipelineName = array[array.length - 2];

    try {
        stage("Checkout"){
            if (env.BRANCH_NAME == 'master') {
                checkout scm
            } else {
                print "Checkout branch : ${env.BRANCH_NAME} and merge to master"
                checkout scm
                sh "git checkout master"
                print "Create new branch : master-${env.BRANCH_NAME}"
                sh "git branch master-${env.BRANCH_NAME}"
                print "Checkout branch : master-${env.BRANCH_NAME}"
                sh "git checkout master-${env.BRANCH_NAME}"
                print "Merging feature branch : ${env.BRANCH_NAME}"
                sh "git merge origin/${env.BRANCH_NAME}"
            }
        }

        stage('Build Docker Image') {
            sh "sudo docker build -t 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:${env.BUILD_ID} -t 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:latest ."
        }

        stage('Push to Docker Resgistry') {
            sh "sudo docker push 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:${env.BUILD_ID}"
            sh "sudo docker push 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:latest"
        }

        stage ('Deploy and Run') {
            if (env.BRANCH_NAME == 'master') {
                print "Deploy docker container to : staging environmet"
                sh "ansible-playbook /vagrant/ansible/${pipelineName}-staging.yml -i /vagrant/ansible/hosts/staging --extra-vars 'branch=${env.BRANCH_NAME}'"
            } else {
                print "Deploy docker container to : testing environmet"
                sh "ansible-playbook /vagrant/ansible/${pipelineName}-testing.yml -i /vagrant/ansible/hosts/testing --extra-vars 'branch=${env.BRANCH_NAME} port=${test_port}'"
            }
        }

        stage('Cleanup') {
            if (env.BRANCH_NAME != 'master') {
                sh "git checkout master"
                sh "git branch -D master-${env.BRANCH_NAME}"
            }

            sh "sudo docker rmi 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:${env.BUILD_ID}"
            sh "sudo docker rmi 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:latest"

        }

        if (env.BRANCH_NAME != 'master') {
            properties([pipelineTriggers([upstream(threshold: hudson.model.Result.SUCCESS, upstreamProjects: '${pipelineName}/master')])])
        }
    } catch (err){
        if (env.BRANCH_NAME != 'master') {
            sh "git checkout master"
            sh "git branch -D master-${env.BRANCH_NAME}"
        }

        sh "sudo docker rmi 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:${env.BUILD_ID}"
        sh "sudo docker rmi 192.168.50.91:5000/${pipelineName}-${env.BRANCH_NAME}:latest"

        throw err
    }
}
