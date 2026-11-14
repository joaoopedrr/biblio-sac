# sacpy — Framework unificado para análise SAC-DM / SAC-AM

Biblioteca em Python para análise de sinais caóticos com base nas técnicas **SAC-DM** (*Signal Analysis based on Chaos using Density of Maxima*) e **SAC-AM** (*Signal Analysis based on Chaos using Amplitude of Maxima*), incluindo testes de validação de caos, geradores de sinais sintéticos canônicos e ferramentas de análise de sensibilidade.

Este projeto preenche a lacuna de **reprodutibilidade computacional** dos fundamentos das técnicas de SAC-DM e SAC-AM, reunindo em um único arcabouço aberto:

- O núcleo das métricas SAC-DM e SAC-AM;
- Os testes de validação de caos (teste 0-1 e Kulp–Smith);
- Geradores de sinais caóticos, periódicos e estocásticos;
- Análise de sensibilidade ao ruído (SNR) e ao comprimento de janela;
- Painéis gráficos de demonstração automatizados.

---

## Sumário

- [Origem e contexto](#origem-e-contexto)
- [Instalação](#instalação)
- [Uso rápido](#uso-rápido)
- [Referência da API](#referência-da-api)
  - [1. Núcleo: SAC-DM e SAC-AM](#1-núcleo-sac-dm-e-sac-am)
  - [2. Teste 0-1 (Gottwald–Melbourne)](#2-teste-0-1-gottwaldmelbourne)
  - [3. Teste de Kulp–Smith](#3-teste-de-kulpsmith)
  - [4. Geradores de sinais](#4-geradores-de-sinais)
  - [5. Análise de sensibilidade](#5-análise-de-sensibilidade)
- [Demonstração gráfica](#demonstração-gráfica)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Resultados esperados](#resultados-esperados)
- [Reprodutibilidade](#reprodutibilidade)
- [Referências](#referências)
- [Licença](#licença)

---

## Origem e contexto

O SAC-DM foi proposto originalmente por **Bazeia et al. (2017, *Scientific Reports*)** no contexto de física estatística e ecologia teórica, como estimador do comprimento de correlação em simulações estocásticas de competição de espécies. A densidade de máximos escala com a raiz do expoente de Lyapunov médio:

$$
\rho \;\propto\; \sqrt{\langle \lambda \rangle}
$$

A técnica foi posteriormente levada à engenharia por **Medeiros et al. (2018, *Drones*; 2019, *IEEE TIE*)**, que consolidaram formalmente o **SAC-DM** e o **SAC-AM** como ferramentas de diagnóstico de falhas em máquinas elétricas. A partir de 2019, a técnica ramificou-se para rolamentos, barras quebradas, VANTs, falhas de ignição, estimação *sensorless* e condicionadores de ar.

O `sacpy` reconstrói essa trajetória em código aberto, documentando cada decisão de implementação de forma auditável.

### Definições formais

**SAC-DM** — densidade de máximos por unidade de tempo (Hz):

$$
D = \frac{|M|}{T}, \qquad T = \frac{N}{f_s}
$$

**SAC-AM** — amplitude média dos máximos locais:

$$
A = \frac{1}{|M|}\sum_{m \in M} |x[m]|
$$

**Propriedades:**
- $D$ é invariante a transformações afins com ganho positivo ($y = \alpha x + \beta$, $\alpha > 0$);
- $D$ captura mudanças de dinâmica; $A$ captura mudanças de energia/severidade.

---

## Instalação

### Requisitos

- Python ≥ 3.9
- NumPy
- SciPy
- Matplotlib

### Instalação das dependências

```bash
pip install numpy scipy matplotlib
