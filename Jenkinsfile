/*
 * Jenkinsfile CORRIGÉ - Student Management DevOps
 * Version sans erreurs de syntaxe
 */

pipeline {
    agent any

    tools {
        maven 'M3'
    }

    environment {
        // Configuration SonarQube
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'squ_0d59b21ba4d8a2a7944e9848065cc69e91ecd18a'

        // Info projet
        PROJECT_NAME = 'Student Management'
        PROJECT_VERSION = '1.0.0'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        // ÉTAPE 1: INITIALISATION
        stage('Initialisation') {
            steps {
                echo "🚀 Pipeline CI/CD - ${PROJECT_NAME}"
                echo "📊 Build #${BUILD_NUMBER}"
                echo "🔗 SonarQube: ${SONAR_HOST_URL}"
            }
        }

        // ÉTAPE 2: CRÉATION PROJET
        stage('Créer projet test') {
            steps {
                sh '''
                    echo "📁 Création du projet de test..."

                    # Nettoyer
                    rm -rf test-project
                    mkdir -p test-project/src/main/java/com/example

                    # Créer fichier Java
                    cat > test-project/src/main/java/com/example/Student.java << EOF
                    package com.example;

                    public class Student {
                        private String name;
                        private int age;

                        public Student(String name, int age) {
                            this.name = name;
                            this.age = age;
                        }

                        public String getName() {
                            return name;
                        }

                        public void setName(String name) {
                            this.name = name;
                        }

                        public int getAge() {
                            return age;
                        }

                        public void setAge(int age) {
                            if (age > 0) {
                                this.age = age;
                            }
                        }

                        public boolean isAdult() {
                            return age >= 18;
                        }

                        @Override
                        public String toString() {
                            return "Student{name='" + name + "', age=" + age + "}";
                        }
                    }
                    EOF

                    # Créer fichier de test
                    mkdir -p test-project/src/test/java/com/example
                    cat > test-project/src/test/java/com/example/StudentTest.java << EOF
                    package com.example;

                    import org.junit.Test;
                    import static org.junit.Assert.*;

                    public class StudentTest {
                        @Test
                        public void testStudentCreation() {
                            Student student = new Student("Alice", 20);
                            assertEquals("Alice", student.getName());
                            assertEquals(20, student.getAge());
                        }

                        @Test
                        public void testIsAdult() {
                            Student adult = new Student("Bob", 25);
                            assertTrue(adult.isAdult());

                            Student minor = new Student("Charlie", 16);
                            assertFalse(minor.isAdult());
                        }
                    }
                    EOF

                    # Créer pom.xml
                    cat > test-project/pom.xml << EOF
                    <project>
                        <modelVersion>4.0.0</modelVersion>
                        <groupId>com.example</groupId>
                        <artifactId>student-management-test</artifactId>
                        <version>1.0.0</version>

                        <properties>
                            <maven.compiler.source>11</maven.compiler.source>
                            <maven.compiler.target>11</maven.compiler.target>
                            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
                        </properties>

                        <dependencies>
                            <dependency>
                                <groupId>junit</groupId>
                                <artifactId>junit</artifactId>
                                <version>4.13.2</version>
                                <scope>test</scope>
                            </dependency>
                        </dependencies>

                        <build>
                            <plugins>
                                <plugin>
                                    <groupId>org.apache.maven.plugins</groupId>
                                    <artifactId>maven-compiler-plugin</artifactId>
                                    <version>3.11.0</version>
                                </plugin>
                            </plugins>
                        </build>
                    </project>
                    EOF

                    echo "✅ Projet créé avec succès"
                '''
            }
        }

        // ÉTAPE 3: BUILD
        stage('Build et Tests') {
            steps {
                sh '''
                    echo "📦 Compilation du projet..."
                    cd test-project
                    mvn clean compile -q
                    echo "✅ Compilation réussie"

                    echo "🧪 Exécution des tests..."
                    mvn test -q
                    echo "✅ Tests terminés"
                '''

                // Publier résultats JUnit
                junit 'test-project/target/surefire-reports/**/*.xml'
            }
        }

        // ÉTAPE 4: ANALYSE SONARQUBE
        stage('Analyse SonarQube') {
            steps {
                script {
                    echo "🔍 Lancement de l'analyse SonarQube..."

                    // Vérifier connexion
                    sh """
                        echo "Test de connexion à SonarQube..."
                        curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" ${SONAR_HOST_URL}/api/system/status || echo "Connexion impossible"
                    """

                    // Configurer SonarQube
                    withSonarQubeEnv('SonarQube-Server') {
                        sh """
                            cd test-project

                            # Version SANS backslashes problématiques
                            mvn sonar:sonar \
                                -Dsonar.host.url=${SONAR_HOST_URL} \
                                -Dsonar.login=${SONAR_TOKEN} \
                                -Dsonar.projectKey=student-management-${BUILD_NUMBER} \
                                -Dsonar.projectName='${PROJECT_NAME}' \
                                -Dsonar.projectVersion=${PROJECT_VERSION} \
                                -Dsonar.sources=src/main/java \
                                -Dsonar.java.binaries=target/classes
                        """
                    }

                    echo "✅ Analyse SonarQube lancée"
                }
            }
        }

        // ÉTAPE 5: QUALITY GATE
        stage('Quality Gate') {
            steps {
                script {
                    echo "⏳ Attente des résultats..."

                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }

                    echo "🎉 Quality Gate réussi !"
                }
            }
        }

        // ÉTAPE 6: RÉSULTATS
        stage('Résultats') {
            steps {
                sh '''
                    echo "🎉 ANALYSE TERMINÉE AVEC SUCCÈS !"
                    echo "================================="
                    echo ""
                    echo "📊 RÉSULTATS DISPONIBLES SUR :"
                    echo "   ${SONAR_HOST_URL}"
                    echo ""
                    echo "🔍 PROJET ANALYSÉ :"
                    echo "   • Nom : ${PROJECT_NAME}"
                    echo "   • Clé : student-management-${BUILD_NUMBER}"
                    echo "   • Version : ${PROJECT_VERSION}"
                    echo ""
                    echo "✅ INTÉGRATION JENKINS + SONARQUBE FONCTIONNELLE !"
                    echo ""
                    echo "🎯 Prochaines étapes :"
                    echo "   1. Connectez-vous à SonarQube"
                    echo "   2. Vérifiez les métriques de qualité"
                    echo "   3. Corrigez les problèmes identifiés"
                    echo "   4. Améliorez la couverture de tests"
                '''

                // Archivage
                archiveArtifacts artifacts: 'test-project/target/*.jar, test-project/pom.xml', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "🏁 Pipeline terminé - Statut : ${currentBuild.result}"
            echo "⏱️ Durée : ${currentBuild.durationString}"

            // Nettoyage
            sh '''
                echo "🧹 Nettoyage des fichiers temporaires..."
                rm -rf test-project/target/classes 2>/dev/null || true
            '''
        }

        success {
            echo "✅ ✅ ✅ SUCCÈS COMPLET ! ✅ ✅ ✅"
            echo "FÉLICITATIONS ! Votre pipeline CI/CD est opérationnel."

            script {
                currentBuild.description = "Succès - Analyse SonarQube complétée"
            }
        }

        failure {
            echo "❌ Échec du pipeline"
            echo "🔧 Vérifiez :"
            echo "   • SonarQube est-il démarré ?"
            echo "   • Le token est-il valide ?"
            echo "   • Maven est-il configuré ?"

            // Test de diagnostic
            sh '''
                echo "Diagnostic rapide :"
                echo "1. Test SonarQube :"
                curl -I ${SONAR_HOST_URL} 2>/dev/null || echo "SonarQube inaccessible"
                echo ""
                echo "2. Test Maven :"
                mvn --version 2>/dev/null || echo "Maven non trouvé"
            '''
        }
    }
}