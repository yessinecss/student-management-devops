pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8'
    }
    
    stages {
        // Étape 1: Vérification
        stage('Check Tools') {
            steps {
                sh '''
                    echo "=== TOOLS CHECK ==="
                    mvn --version
                    java -version
                '''
            }
        }
        
        // Étape 2: Création projet
        stage('Create Project') {
            steps {
                sh '''
                    # Création structure minimale
                    mkdir -p myapp/src/main/java/com/demo
                    mkdir -p myapp/src/test/java/com/demo
                    
                    # Classe simple
                    cat > myapp/src/main/java/com/demo/Calculator.java << 'EOF'
                    package com.demo;
                    
                    public class Calculator {
                        public int add(int a, int b) {
                            return a + b;
                        }
                        
                        public int subtract(int a, int b) {
                            return a - b;
                        }
                    }
                    EOF
                    
                    # Test simple
                    cat > myapp/src/test/java/com/demo/CalculatorTest.java << 'EOF'
                    package com.demo;
                    
                    import org.junit.Test;
                    import static org.junit.Assert.*;
                    
                    public class CalculatorTest {
                        @Test
                        public void testAdd() {
                            Calculator calc = new Calculator();
                            assertEquals(5, calc.add(2, 3));
                        }
                        
                        @Test
                        public void testSubtract() {
                            Calculator calc = new Calculator();
                            assertEquals(1, calc.subtract(3, 2));
                        }
                    }
                    EOF
                    
                    # pom.xml minimal
                    cat > myapp/pom.xml << 'EOF'
                    <project>
                        <modelVersion>4.0.0</modelVersion>
                        <groupId>com.demo</groupId>
                        <artifactId>myapp</artifactId>
                        <version>1.0</version>
                        
                        <properties>
                            <maven.compiler.source>11</maven.compiler.source>
                            <maven.compiler.target>11</maven.compiler.target>
                        </properties>
                        
                        <dependencies>
                            <dependency>
                                <groupId>junit</groupId>
                                <artifactId>junit</artifactId>
                                <version>4.13.2</version>
                                <scope>test</scope>
                            </dependency>
                        </dependencies>
                    </project>
                    EOF
                '''
            }
        }
        
        // Étape 3: Build
        stage('Build & Test') {
            steps {
                sh '''
                    cd myapp
                    mvn clean compile test
                '''
                
                // Rapports tests
                junit 'myapp/target/surefire-reports/**/*.xml'
            }
        }
        
        // Étape 4: SonarQube (SÉCURISÉ)
        stage('SonarQube Analysis') {
            steps {
                // ✅ TOKEN SÉCURISÉ - pas dans le code
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        cd myapp
                        mvn sonar:sonar \
                            -Dsonar.projectKey=myapp-${BUILD_NUMBER} \
                            -Dsonar.projectName="My Application"
                    '''
                }
            }
        }
        
        // Étape 5: Vérification qualité
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline réussi!'
        }
        failure {
            echo '❌ Pipeline échoué'
        }
    }
}
