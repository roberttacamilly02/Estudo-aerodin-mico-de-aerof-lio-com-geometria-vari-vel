# Estudo Aerodinâmico de Aerofólio com Geometria Variável

> Plano de projeto do Grupo C focado no estudo aerodinâmico de aerofólios de geometria variável. Por meio de simulações em CFD de um flap simples, o trabalho analisa a relação entre sustentação e arrasto para avaliar a eficiência aerodinâmica, estabilidade e redução no consumo de combustível.

## 👥 Integrantes
- Elisa Morais Nunes
- Ian Vicente Tavares
- Lavínia Gomes
- Roberta Camilly Freitas de Carvalho

## 🎯 Objetivos
- **Geral:** Analisar a influência da geometria variável no desempenho aerodinâmico.
- **Específicos:** Investigar princípios físicos por simulação e aprofundar os conhecimentos em física aeronáutica.

## 🛠️ Ferramentas Utilizadas
- Simulação em CFD (Dynamics of Fluids Computacional)
- XFLR5

- # Guia Completo: Simulação de Aerofólio no OpenFOAM

## Resumo
Este documento apresenta um guia detalhado para simulação do comportamento de aerofólios utilizando o OpenFOAM. O fluxo de trabalho é dividido em 5 etapas principais, desde a preparação da geometria até a análise dos resultados.

---

## Visão Geral do Processo

1. Preparação da Geometria e Malha
2. Escolha das Condições Físicas e Modelos
3. Configuração dos Arquivos de Caso (*setup*)
4. Execução da Simulação
5. Pós-Processamento e Análise dos Resultados

---

## 1. Preparação da Geometria e Malha

Esta é a etapa mais crucial. Uma boa malha é fundamental para resultados precisos.

### 1.1 Geometria do Aerofólio
* **Formato:** Arquivo CAD do aerofólio (ex: `.stl`, `.iges`, `.step`)
* **Formato comum:** STL
* **Fonte de geometrias:** Bancos de dados como o *UIUC Airfoil Coordinates Database* fornecem arquivos de coordenadas (`.dat`)
* **Ferramentas de conversão:** OpenSCAD, Blender ou Salome para converter coordenadas em arquivo `.stl`

### 1.2 Criação da Malha
Duas abordagens principais no OpenFOAM:

#### Abordagem 1: `blockMesh` + `snappyHexMesh` (Recomendada)
* `blockMesh`: Cria um bloco hexagonal inicial (domínio computacional) muito grande ao redor da área onde estará o aerofólio.
* `snappyHexMesh`: "Corta" o aerofólio dentro do bloco, refinando a malha perto das superfícies.

#### Abordagem 2: `cfMesh` (Alternativa mais amigável)
* Conjunto de utilitários externos.
* Considerado mais fácil e rápido para gerar malhas complexas.
* Processo semelhante, mas com menos configuração.

### 1.3 Pontos Críticos da Malha
* **Camada Limite (*Inflation Layer*):** É ESSENCIAL ter várias camadas de elementos prismáticos junto ao aerofólio. Calcule o $y^+$ desejado (geralmente $y^+ \approx 1$ para simulações com resolução da subcamada laminar).
* **Refinamento Local:** Malha mais fina ao redor do aerofólio, no rastro (*wake*) e regiões de gradientes altos.
* **Domínio Computacional:** Fronteiras a pelo menos 10-20 cordas do aerofólio.

---

## 2. Escolha das Condições Físicas e Modelos

### 2.1 Solucionador (*Solver*)

| Tipo de Escoamento | Solucionador Recomendado |
| :--- | :--- |
| **Incompressível e Estacionário** | `simpleFoam` (RANS) |
| **Incompressível e Transiente** | `pisoFoam` (RANS/LES) ou `pimpleFoam` |
| **Compressível** | `rhoSimpleFoam` ou `rhoPimpleFoam` |

> **Para começar:** `simpleFoam` é ótimo para escoamentos subsônicos em ângulos de ataque não muito altos.

### 2.2 Modelo de Turbulência (para RANS)
* **k-omega SST:** Popular para escoamentos externos com gradientes adversos de pressão.
* **Spalart-Allmaras:** Usado na indústria aeronáutica, modelo de uma equação.

### 2.3 Propriedades do Fluido
Definir a viscosidade cinemática (`nu`) do ar no arquivo `constant/transportProperties`.

---

## 3. Configuração dos Arquivos de Caso (*setup*)

### 3.1 `system/controlDict`
Controla a execução da simulação. **Crucial:** Adicione uma *function object* para calcular forças e coeficientes aerodinâmicos.

```foam
functions
{
    forces
    {
        type        forces;
        libs        ("libforces.so");
        writeControl writeTime;
        patches     (airfoil);      // Nome do patch do aerofólio
        rho         rhoInf;         // Usar densidade de referência
        rhoInf      1.225;          // Densidade do ar [kg/m³]
        CofR        (0 0 0);        // Centro de rotação
        liftDir     (0 1 0);        // Direção da sustentação
        dragDir     (1 0 0);        // Direção do arrasto
        pitchAxis   (0 0 1);        // Eixo do momento de arfagem
        magUInf     50;             // Velocidade de referência [m/s]
        lRef        1.0;            // Corda de referência [m]
        Aref        1.0;            // Área de referência [m²]
    }
}
