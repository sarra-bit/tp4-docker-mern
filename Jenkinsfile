pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build images') {
            steps { bat 'docker compose build' }
        }
        stage('Run') {
            steps { bat 'docker compose up -d' }
        }
        stage('Check') {
            steps { bat 'docker ps' }
        }
    }
}