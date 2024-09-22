# MATLAB CI/CD Pipeline

Este repositorio implementa un pipeline de **Integración Continua (CI)** y **Despliegue Continuo (CD)** para código MATLAB utilizando herramientas como **GitHub Actions** o **GitLab CI**. Este README explica cómo configurar y utilizar el pipeline.

## Requisitos

- [MATLAB](https://www.mathworks.com/products/matlab.html)
- Repositorio Git (por ejemplo, GitHub, GitLab o Bitbucket)
- Acceso a MATLAB Online o MATLAB Production Server (opcional para CD)
- MATLAB Unit Test framework para pruebas unitarias

## Estructura del Proyecto

Organiza tu proyecto de la siguiente manera:

/src # Código fuente /tests 

```matlab
% Definición de parámetros
RM = 3.53E-3;
M = 0.0045;
SSUP = 2.6E4;
RHO = 1.68;
C = 1000;
S = 1.521E-3;
BL = 8;
Q = 15;
FO = 800;
A = 7.13E-5;
VOL = 2.715E-6;
RG = 0;
LE = 0;
RE = 8;
I = 1;
L = 0.625;

% Constantes
PI = 3.141592654;
SVOL = (A * A * RHO * C * C) / VOL;
WO = 2 * PI * FO;
ALPHA = WO / (2 * Q * C);

% Rango de frecuencias
f0 = FO;
frequencies = linspace(f0 / 4, (3 * f0) / 2, 300);
VOLTAGE = zeros(1, 300);
EFFICIENCY = zeros(1, 300);

for J = 1:300
    F = frequencies(J);
    W = 2 * PI * F;
    K = W / C;
    LCOIL = W * LE;
    
    % Cálculo de las partes real e imaginaria de la impedancia mecánica ZM
    REZM = RM + ((RHO * C * S * ALPHA * cos(K * L) * sin(K * L)) + ...
        (K * RHO * C * S * sinh(ALPHA * L) * cosh(ALPHA * L))) / ...
        (K * (1 + (ALPHA / K)^2) * (((sin(K * L))^2 * (cosh(ALPHA * L))^2) + ...
        ((cos(K * L))^2 * (sinh(ALPHA * L))^2)));

    IMZM = (W * M) - (SSUP / W) - (SVOL / W) + ((RHO * C * S * ALPHA * sinh(ALPHA * L) * ...
        cosh(ALPHA * L)) - (K * RHO * C * S * cos(K * L) * sin(K * L))) / ...
        (K * (1 + ALPHA / K)^2 * (((sin(K * L))^2 * (cosh(ALPHA * L))^2) + ...
        ((cos(K * L))^2 * (sinh(ALPHA * L))^2)));
    
    ZM = complex(REZM, IMZM);
    
    % Impedancia total
    ZE = (BL^2) / ZM;
    ZCOIL = complex(RE, LCOIL);
    ZTOT = RG + ZCOIL + ZE;
    
    % Voltaje y eficiencia acústica
    V = I * ZTOT;
    FORCE = BL * I;
    U = FORCE / ZM;
    MAGU = abs(U);
    
    % Potencias y eficiencia
    P = (ZTOT * U) / S;
    PU = 0.5 * real(P * conj(U)) * S;
    PWR = 0.5 * real(I * conj(V));
    PUPWR = PU / PWR;
    MAGV = abs(V);
    
    % Almacenar valores
    VOLTAGE(J) = MAGV;
    EFFICIENCY(J) = PUPWR;
end

% Gráfica Voltaje vs Frecuencia
figure;
subplot(2, 1, 1);
plot(frequencies, VOLTAGE, 'LineWidth', 2);
xlabel('Frecuencia (Hz)');
ylabel('Voltaje (V)');
title('Voltaje vs Frecuencia');
grid on;

% Gráfica Eficiencia Acústica vs Frecuencia
subplot(2, 1, 2);
plot(frequencies, EFFICIENCY, 'LineWidth', 2);
xlabel('Frecuencia (Hz)');
ylabel('Eficiencia Acústica');
title('Eficiencia Acústica vs Frecuencia');
grid on;
```

# Pruebas unitarias
### Ejemplo de prueba unitaria en MATLAB

Crea casos de prueba usando **MATLAB Unit Test** en la carpeta `tests/`:

```matlab
classdef TestMiCodigo < matlab.unittest.TestCase
    methods(Test)
        function testFuncionPrincipal(testCase)
            resultado = funcionPrincipal(10);  % llama a tu función MATLAB
            testCase.verifyEqual(resultado, valorEsperado);
        end
    end
end
```
# Configuración del pipeline CI/CD
 GitHub Actions
 Para configurar un pipeline CI en GitHub Actions, crea el archivo .github/workflows/matlab-ci.yml:

```yaml

name: MATLAB CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up MATLAB
      uses: matlab-actions/setup-matlab@v1

    - name: Run MATLAB Tests
      run: |
        matlab -batch "results = runtests('tests'); assertSuccess(results);"
 end
    end
end
```
# Este pipeline:

- Clona tu repositorio en cada push o pull_request.
- Configura MATLAB utilizando matlab-actions/setup-matlab.
- Ejecuta las pruebas unitarias en la carpeta tests/ y asegura que todas pasen.

# GitLab CI
Para configurar un pipeline en GitLab CI, crea el archivo .gitlab-ci.yml en la raíz del repositorio:

```yaml
image: matlab:latest

stages:
  - test
  - deploy

test:
  stage: test
  script:
    - matlab -batch "results = runtests('tests'); assertSuccess(results);"
  artifacts:
    when: always
    reports:
      junit: test-results.xml
  only:
    - master

deploy:
  stage: deploy
  script:
    - echo "Deploying MATLAB code"
    - scp -r ./src/ user@server:/path/to/deployment/
  only:
    - master
```
# Este pipeline:

Ejecuta las pruebas unitarias en la fase de test.
Si las pruebas son exitosas, despliega el código en la fase de deploy, copiando los archivos a un servidor usando scp.

# Jenkins
Si prefieres usar Jenkins para CI/CD, puedes crear un archivo Jenkinsfile similar:

```groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                script {
                    sh 'matlab -batch "results = runtests(\'tests\'); assertSuccess(results);"'
                }
            }
        }
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                script {
                    sh 'scp -r ./src/ user@server:/path/to/deployment/'
                }
            }
        }
    }
}
```
# Despliegue Continuo (CD)
Dependiendo del proyecto, puedes optar por un despliegue continuo tras pasar las pruebas:

- Automatización en servidor: Ejecutar scripts MATLAB automáticamente en un servidor.
- MATLAB Production Server: Si usas MATLAB Production Server, puedes desplegar funciones MATLAB para que estén accesibles como servicios web.
  El ejemplo anterior utiliza SCP para copiar los archivos al servidor remoto en la fase de despliegue.

#  Ejecución de Pruebas
Puedes ejecutar las pruebas manualmente utilizando la función runtests en MATLAB:

```matlab
results = runtests('tests');
assertSuccess(results);```

# Referencias
- MATLAB Unit Test Framework
- MATLAB Code Analyzer
- MATLAB Production Server
