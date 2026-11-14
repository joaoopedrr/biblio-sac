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
```

### Clonagem do repositório

```bash
git clone https://github.com/joaoopedrr/biblio-sac.git
cd biblio-sac
```

O `sacpy.py` é um módulo autocontido; basta colocá-lo no mesmo diretório do seu script ou adicioná-lo ao `PYTHONPATH`.

---

## Uso rápido

### Como biblioteca

```python
import sacpy as sp

# Gera um sinal caótico de Lorenz (20 s a 200 Hz)
x = sp.lorenz(T=20, fs=200)

# Aplica SAC-DM e SAC-AM
res = sp.sac_full(x, fs=200)
print(f"SAC-DM = {res.D:.3f} Hz")
print(f"SAC-AM = {res.A:.3f}")

# Testes de caos
k = sp.test_01(x)["K_median"]
delta = sp.test_kulp_smith(x)["delta"]
print(f"Teste 0-1 (K)      = {k:+.3f}")
print(f"Kulp-Smith (delta) = {delta:.3f}")
```

### Como demonstração gráfica

```bash
python sacpy.py
```

Isso gera automaticamente quatro arquivos PNG na pasta atual:

| Arquivo | Conteúdo |
|---|---|
| `sacpy_dashboard.png` | Painel principal (4×3): séries temporais, SAC-DM em janelas deslizantes, histogramas do teste 0-1 e comparações |
| `sacpy_snr.png` | Sensibilidade do SAC-DM ao ruído aditivo (SNR) |
| `sacpy_window.png` | Convergência do SAC-DM com o comprimento da janela |
| `sacpy_test01.png` | Trajetórias $(p_c, q_c)$ do teste 0-1 |

---

## Referência da API

### 1. Núcleo: SAC-DM e SAC-AM

#### `sac_dm(x, fs) -> float`

Calcula a densidade de máximos locais (Hz).

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `x` | `np.ndarray` | Série temporal |
| `fs` | `float` | Frequência de amostragem (Hz) |

**Implementação:** comparações vetorizadas `x[1:-1] > x[:-2]` e `x[1:-1] > x[2:]` — custo linear em $N$, muito mais rápido que um laço explícito.

---

#### `sac_am(x, fs) -> float`

Calcula a amplitude média dos máximos locais.

**Implementação:** usa `scipy.signal.find_peaks`, que trata corretamente platôs e é padrão auditável.

---

#### `sac_full(x, fs) -> SACResult`

Aplica SAC-DM e SAC-AM simultaneamente e retorna um `dataclass` `SACResult`:

```python
@dataclass
class SACResult:
    D: float              # SAC-DM (Hz)
    A: float              # SAC-AM
    n_maxima: int         # número de máximos
    duration: float       # duração (s)
    peak_indices: np.ndarray  # índices dos máximos
```

---

### 2. Teste 0-1 (Gottwald–Melbourne)

#### `test_01(phi, n_c=100, seed=42) -> dict`

Teste 0-1 de Gottwald & Melbourne (2009).

**Referência:** Gottwald, G. A.; Melbourne, I. *On the implementation of the 0-1 test for chaos*. SIAM J. Appl. Dyn. Syst., v. 8, n. 1, p. 129–145, 2009. DOI: [10.1137/080718851](https://doi.org/10.1137/080718851).

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `phi` | `np.ndarray` | Série temporal |
| `n_c` | `int` | Número de valores de $c$ amostrados (padrão: 100) |
| `seed` | `int` | Semente do RNG (padrão: 42) |

**Retorno:** dicionário com `K_mean`, `K_median`, `K_std`, `K_all`.

**Interpretação:**
- $K \approx 0$ → regular/periódico
- $K \approx 1$ → caótico

**Três decisões críticas de implementação:**
1. **Normalização obrigatória** (média zero, desvio unitário) — omitida em reimplementações incorretas;
2. **Amostragem de $c$ em $(\pi/5, 4\pi/5)$**, evitando as bordas do intervalo $(0, \pi)$ onde ocorrem ressonâncias;
3. **Descarte de 10% de transiente** e estimador $K_c$ dado pela correlação de Pearson entre $n$ e o deslocamento quadrático médio $M_c(n)$; retorna-se a mediana sobre $n_c$ valores de $c$.

---

### 3. Teste de Kulp–Smith

#### `test_kulp_smith(x, n_symbols=4, embedding=3, delay=1) -> dict`

Teste de determinismo simbólico de Kulp & Smith (2011).

**Referência:** Kulp, C. W.; Smith, S. *Characterization of noisy symbolic time series*. Phys. Rev. E, v. 83, n. 2, 2011.

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `x` | `np.ndarray` | Série temporal |
| `n_symbols` | `int` | Número de níveis equiprováveis (padrão: 4) |
| `embedding` | `int` | Dimensão de *embedding* (padrão: 3) |
| `delay` | `int` | Atraso de *embedding* (padrão: 1) |

**Retorno:** dicionário com `delta` (razão de determinismo) e `n_states`.

**Interpretação:**
- $\delta \approx 1$ → determinismo forte
- $\delta \approx 0$ → ruído estocástico

> **Importante:** o teste 0-1 isolado classifica o ruído branco como caótico ($K \approx 0{,}994$). A combinação dos dois testes desmascara o ruído e atesta o determinismo dos atratores.

---

### 4. Geradores de sinais

| Função | Descrição | Padrão |
|---|---|---|
| `lorenz(T=200.0, fs=200.0, sigma=10.0, rho=28.0, beta=8/3)` | Atrator de Lorenz | T=200 s, fs=200 Hz (20.000 amostras) |
| `rossler(T=200.0, fs=200.0, a=0.2, b=0.2, c=5.7)` | Atrator de Rössler | T=200 s, fs=200 Hz |
| `logistic(n=20000, r=3.99, x0=0.4)` | Mapa logístico | 20.000 iterações (`fs=1`) |
| `harmonic(f=50.0, T=1.0, fs=10000.0)` | Sinal periódico de referência | 50 Hz |
| `white_noise(n=10000, seed=0)` | Ruído branco gaussiano | 10.000 amostras |
| `add_noise(x, snr_db, seed=0)` | Adiciona ruído branco com SNR especificado (dB) | — |

---

### 5. Análise de sensibilidade

#### `sensitivity_snr(signal_fn, fs, snr_range=..., n_trials=15) -> dict`

SAC-DM em função da SNR.

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `signal_fn` | `Callable[[], np.ndarray]` | Função que gera uma nova realização do sinal |
| `fs` | `float` | Frequência de amostragem |
| `snr_range` | `np.ndarray` | Faixa de SNR (padrão: $-5$ a $35$ dB, passo 5) |
| `n_trials` | `int` | Número de realizações por SNR (padrão: 15) |

**Retorno:** dicionário com `snr`, `D_mean`, `D_std`.

---

#### `sensitivity_window(signal, fs, fractions=...) -> dict`

SAC-DM em função do comprimento da janela.

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `signal` | `np.ndarray` | Sinal completo |
| `fs` | `float` | Frequência de amostragem |
| `fractions` | `np.ndarray` | Frações do sinal usadas como janela (padrão: 5% a 100%, 25 pontos) |

**Retorno:** dicionário com `fractions`, `window_seconds`, `D`.

---

## Demonstração gráfica

A rotina `python sacpy.py` executa `_demo_graficos()` e produz quatro painéis:

1. **`sacpy_dashboard.png`** — painel principal 4×3 com:
   - Linha 1: séries temporais de Lorenz, Rössler e comparação entre caóticos;
   - Linha 2: SAC-DM em janelas deslizantes (separação entre regimes);
   - Linha 3: histogramas de $K$ do teste 0-1;
   - Linha 4: SAC-DM por sistema, boxplot de $K$ e máximos marcados sobre a série de Lorenz.

2. **`sacpy_snr.png`** — sensibilidade do SAC-DM ao ruído aditivo para Lorenz, logístico e Rössler.

3. **`sacpy_window.png`** — convergência do SAC-DM com o comprimento da janela.

4. **`sacpy_test01.png`** — trajetórias $(p_c, q_c)$ do teste 0-1 com $c = 1{,}7$ para Lorenz, logístico, harmônico e ruído.

---

## Estrutura do repositório

```
biblio-sac/
├── sacpy.py               # Módulo principal (autocontido)
├── sacpy_dashboard.png    # Gerado por `python sacpy.py`
├── sacpy_snr.png          # Gerado por `python sacpy.py`
├── sacpy_window.png       # Gerado por `python sacpy.py`
├── sacpy_test01.png       # Gerado por `python sacpy.py`
├── apresentacao.tex       # Slides Beamer da apresentação
└── README.md              # Este arquivo
```

---

## Resultados esperados

Validação sintética sobre cinco sistemas canônicos (valores típicos):

| Sistema | SAC-DM (Hz) | Teste 0-1 ($K$) | Kulp–Smith ($\delta$) | Diagnóstico |
|---|---|---|---|---|
| Lorenz | $\sim 35{,}2$ | $+0{,}998$ | $1{,}000$ | Caótico determinístico |
| Rössler | $\sim 28{,}5$ | $+0{,}996$ | $1{,}000$ | Caótico determinístico |
| Logístico ($r=3{,}99$) | $0{,}500$ | $+0{,}999$ | $1{,}000$ | Caótico determinístico |
| Harmônico (50 Hz) | $50{,}000$ | $-0{,}002$ | $0{,}000$ | Regular/periódico |
| Ruído branco | $\sim 105{,}3$ | $+0{,}994$ | $0{,}000$ | Estocástico (caos falso) |

**Leitura:**
- O teste 0-1 isolado classifica o ruído branco como caótico ($K \approx 0{,}994$) — armadilha metodológica conhecida;
- O teste de Kulp–Smith desmascara o ruído ($\delta = 0{,}000$) e atesta o determinismo dos atratores ($\delta = 1{,}000$);
- Isso valida a abordagem de Lucena-Junior et al. (2020) de usar os dois testes em conjunto.

---

## Reprodutibilidade

Após clonar o repositório e instalar as dependências:

```python
import sacpy as sp

x = sp.lorenz(T=20, fs=200)
res = sp.sac_full(x, fs=200)
k = sp.test_01(x)["K_median"]
delta = sp.test_kulp_smith(x)["delta"]

print(f"SAC-DM = {res.D:.3f} Hz")
print(f"SAC-AM = {res.A:.3f}")
print(f"K      = {k:+.3f}")
print(f"delta  = {delta:.3f}")
```

Funções de sensibilidade:

```python
sp.sensitivity_snr(lambda: sp.lorenz(T=10, fs=200), fs=200)
sp.sensitivity_window(sp.lorenz(T=30, fs=200), fs=200)
```

A rotina `python sacpy.py` gera automaticamente o painel de demonstração completo.

---

## Referências

1. **Bazeia, D. et al.** *Density of maxima and correlation length in stochastic simulations of species competition*. Scientific Reports, 2017.
2. **Medeiros, R. L. P. et al.** *A novel approach for failure diagnosis in BLDC motors based on chaos*. Drones, 2018.
3. **Medeiros, R. L. P. et al.** *Signal Analysis based on Chaos using Density of Maxima and Amplitude of Maxima*. IEEE Transactions on Industrial Electronics, 2019.
4. **Gottwald, G. A.; Melbourne, I.** *On the implementation of the 0-1 test for chaos*. SIAM J. Appl. Dyn. Syst., v. 8, n. 1, p. 129–145, 2009. DOI: [10.1137/080718851](https://doi.org/10.1137/080718851).
5. **Kulp, C. W.; Smith, S.** *Characterization of noisy symbolic time series*. Phys. Rev. E, v. 83, n. 2, 2011.
6. **Lucena-Junior, J. A. et al.** *Bearing fault diagnosis using sound and SAC-DM/SAC-AM*. 2020.
7. **Strogatz, S. H.** *Nonlinear Dynamics and Chaos*. 2. ed. CRC Press, 2018.

---

## Licença

Este projeto é disponibilizado sob a **Licença MIT**. Consulte o arquivo `LICENSE` para os termos completos.

---

## Autores

- **João Pedro Feitoza Primo** — UNIPÊ
- **Jean Ícaro Lima de Oliveira** — UNIPÊ
- **Maelso Bruno Pacheco Nunes Pereira** (orientador) — UNIPÊ

**Repositório:** <https://github.com/joaoopedrr/biblio-sac>
